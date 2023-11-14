## 为预制物添加容器成分

```lua
if not TheWorld.ismastersim then
    inst.OnEntityReplicated = function(inst)
	inst.replica.container:WidgetSetup("newpot")  --为客机注册
	end
    return inst
end
inst:AddComponent("container")
inst.components.container:WidgetSetup("newpot")
```

## 容器的UI

上面为预制物注册了叫`"newpot"`的容器,下面可以在modmain中来写UI:
```lua
GLOBAL.setmetatable(env,{__index=function(t,k) return GLOBAL.rawget(GLOBAL,k) end})
local params = {}
params.newpot = {
    widget =
    {
        slotpos = {
            Vector3(0, 0-5+79, 0+30),
            Vector3(0, 0-5, 0+30),
            Vector3(0, 0-5-79, 0+30),
        },
        --下面几句设置容器格子的图片
        --slotbg =
        --{
            --{atlas="images/newpot.xml",image = "newpot.tex"},
            --{atlas="images/newpot.xml",image = "newpot.tex"},
			--{atlas="images/newpot.xml",image = "newpot.tex"},
        --},
        animbank = "ui_newpot_1x3", --这两句是设置容器背景,可不写
        animbuild = "ui_newpot_1x3",
        pos = Vector3(0+60, 0+150, 0+30),
        side_align_tip = 100,
    },
    type = "newpot", --容器类型,同一时间相同容器类型的容器只能打开一个
}
-----------
--添加容器--
-----------
local containers = require "containers"

for k, v in pairs(params) do
    containers.MAXITEMSLOTS = math.max(containers.MAXITEMSLOTS, v.widget.slotpos ~= nil and #v.widget.slotpos or 0)
end

local containers_widgetsetup = containers.widgetsetup

function containers.widgetsetup(container, prefab, data)
    local t = data or params[prefab or container.inst.prefab]
    if t~=nil then
        for k, v in pairs(t) do
            container[k] = v
        end
        container:SetNumSlots(container.widget.slotpos ~= nil and #container.widget.slotpos or 0)
    else
        return containers_widgetsetup(container, prefab, data)
    end
end
```

## 容器的按钮

```lua
params.newpot.widget.buttoninfo = {
            text = "煮饭",
            position = Vector3(0, 0-5-79-59, 0+30),
        }

function params.newpot.widget.buttoninfo.fn(inst, doer)
    if inst.components.container ~= nil then
        if inst.components.container:IsFull() then --容器满了才能点
            if inst.buttonfn then --按钮点击后实现的功能
                inst.buttonfn(inst)
            end
        end
    elseif inst.replica.container ~= nil and not inst.replica.container:IsBusy() then
        SendRPCToServer(RPC.DoWidgetButtonAction, nil, inst, nil)
    end
end

function params.newpot.widget.buttoninfo.validfn(inst)
    return inst.replica.container ~= nil and inst.replica.container:IsFull()
end
```
下面我们回到预制物写一下按钮功能 inst.buttonfn

## 按钮功能

预制物中:
```lua
local function buttonfn(inst)
	--body
end
inst.buttonfn = buttonfn
```
buttonfn可以分两部分,一部分是判断食谱,一部分是开煮

## 判断食谱

```lua
--[[食谱样式
local recipetab_sample = {
	{"食材A","食材B"},
	{},
	{},
}
]]
---------------
local function checkrecipe(inst,recipetab,boiltime,productname,productnum)
	----------------------(inst,食谱,烹煮时间,产品名,产品个数)
    local icc = inst.components.container
    local pot_itms = {} --锅子的食材表
    local recipetab_len = #recipetab --原料个数应和容器格子数一致
    if icc:IsFull() then --容器是满的,若不加该判断,成功执行一次函数后,容器会清空,下面的slots[k]会变成无效值
        for k=1,recipetab_len do
            table.insert(pot_itms,icc.slots[k].prefab) --锅子内的原料
        end

        for k=1,recipetab_len do
            local len = #recipetab[k]
            local times = len
            local needremove = {0} 
            for i=1,len do
                for t=1,recipetab_len do
                    if pot_itms[t] ~= nil then
                        if pot_itms[t] == recipetab[k][times] then --如果锅子中第t项是正确食材
                            needremove[1] = t --保存第t项
                        end
                    end
                end
                times = times - 1
            end
            if needremove[1] > 0 then --如果存在"第t项",即正确食材,则从食材表中删掉这一项
                table.remove(pot_itms,needremove[1])
            end
        end
        if (#pot_itms) == 0 then --如果食材表删空了,则代表食材全部正确,开始煮饭
            boiling(inst,boiltime,productname,productnum) --煮饭函数
        end
    end
end
```

## 开始煮

写煮饭函数`boiling(inst,boiltime,productname,productnum)`
```lua
local function boiling(inst,boiltime,productname,productnum)
    inst.Light:Enable(true) --烹煮时锅子开灯(如果有)
    inst.components.container:Close() --煮饭时容器关闭
    inst.AnimState:PlayAnimation("cooking_boil_small",true) --播放煮饭动画
    inst.components.container.canbeopened = false --煮饭时容器不能打开
    inst.SoundEmitter:KillSound("snd")
    inst.SoundEmitter:PlaySound("dontstarve/common/cookingpot_rattle", "snd") --播放煮饭音效

    inst.boiltimetask = inst:DoTaskInTime(boiltime, function() --烹煮时间到了执行
    	inst.components.container:DestroyContents() --删除锅子里的食材
        inst.Light:Enable(false) --关灯
        inst.components.container.canbeopened = true --煮好锅子能打开了
        inst.AnimState:PlayAnimation("steam",false) --播放煮好时的蒸汽
        inst.AnimState:PushAnimation("idle", false) --回到平时的动画idle
        inst.SoundEmitter:KillSound("snd")
        inst.SoundEmitter:PlaySound("dontstarve/common/cookingpot_finish") --播放煮好的音效
        for k=1,productnum do
            inst.components.container:GiveItem(SpawnPrefab(productname)) --给几个产品
        end
        if inst.components.worldsettingstimer then
            inst:RemoveComponent("worldsettingstimer")
        end
        inst.boiltimetask = nil --释放
    end)

    inst:AddComponent("worldsettingstimer") --加了一个官方的成分"计时器",这样开着Insight这样的MOD,可以看到剩余时间
    inst.components.worldsettingstimer:AddTimer("newpot_boil", boiltime, true)
    inst.components.worldsettingstimer:StartTimer("newpot_boil", boiltime)

end
```

## 测试buttonfn

回到`buttonfn`添加下新食谱
```lua
local function buttonfn(inst)
--食谱中:当某组原料中某一项与其他组内原料相同,则两组原料必须一致
    local recipe_tomatoeggsoup = {
        {"tomato","tomato_cooked"}, --生熟番茄都行
        {"bird_egg","bird_egg_cooked"},
        {"ice"},
    }
    checkrecipe(inst,recipe_tomatoeggsoup,30,"tomatoeggsoup",1)
end
```
