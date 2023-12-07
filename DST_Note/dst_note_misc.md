## 简易冷却

```lua
    if not inst:HasTag("iscooldown") then
        print("施法!")
        inst:AddTag("iscooldown")
    end
    inst:DoTaskInTime(cooldown_time, function()
        inst:RemoveTag("iscooldown")
    end)
```
也可以放进components里面
```lua
local bfnpack = Class(function(self, inst)
    self.inst = inst
    self.cooldown_on = false
end,
nil,
{
})
function bfnpack:OnSave()
    local data = {
        --cooldown_on = self.cooldown_on,
    }
    return data
end
function bfnpack:OnLoad(data)
    self.cooldown_on = false
end
---------------------------
--简易冷却(注:冷却时间不保存)
function bfnpack:cooldown(cooldown_time)
    if not self.cooldown_on then
        self.cooldown_on = true
    end
    inst:DoTaskInTime(cooldown_time, function()
        self.cooldown_on = false
    end)
end
function bfnpack:iscooldown()
    return self.cooldown_on
end
--[[e.g.
    if not inst.components.bfnpack:iscooldown() then
        print("法术!")
        inst.components.bfnpack:cooldown(8)
    end
]]
---------------------------
return bfnpack
```

## 采集附近农作物

以自己为中心放射状采集附近农作物,并且采集时赠送一个对应的种子
```lua
    local farm_plant_table = {
        {"farm_plant_garlic","garlic_seeds"},
        {"farm_plant_carrot","carrot_seeds"},
        {"farm_plant_potato","potato_seeds"},
        {"farm_plant_pumpkin","pumpkin_seeds"},
        {"farm_plant_onion","onion_seeds"},
        {"farm_plant_tomato","tomato_seeds"},
        {"farm_plant_corn","corn_seeds"},
        {"farm_plant_eggplant","eggplant_seeds"},
        {"farm_plant_pepper","pepper_seeds"},
        {"farm_plant_asparagus","asparagus_seeds"},
        {"farm_plant_pomegranat","pomegranat"},
        {"farm_plant_watermelon","watermelon_seeds"},
        {"farm_plant_dragonfruit","dragonfruit_seeds"},
        {"farm_plant_durian","durian_seeds"},
    }
        for rangs = 1, 10 do
            inst:DoTaskInTime(rangs, function() 
                local ents = TheSim:FindEntities(pos.x,pos.y,pos.z,rangs*2)
                for k,v in pairs(ents) do
                    --农作物有farm_plant的标签
                    if v.components.pickable and v:HasTag("farm_plant") then  
                        SpawnPrefab("sand_puff").Transform:SetPosition(v.Transform:GetWorldPosition()) --加采集特效,可以不加,防卡顿
                        local seed_pt = Vector3(v.Transform:GetWorldPosition()) + Vector3(math.random(0,3),math.random(0,3),math.random(0,3)) - Vector3(math.random(0,3),math.random(0,3),math.random(0,3))

                        for i, entry in ipairs(farm_plant_table) do
                            local farm_plant_name = entry[1]
                            local seed_name = entry[2]
                            if v.prefab == farm_plant_name then
                                SpawnPrefab(seed_name).Transform:SetPosition(seed_pt:Get())
                            end
                        end
                        v.components.pickable:Pick()
                        if caster.SoundEmitter then
                            caster.SoundEmitter:PlaySound("dontstarve/wilson/pickup_plants")
                        end
                    end
                end
            end)
        end
```

## 自动拾取附近物品

```lua
--caster为玩家自己
if not caster:HasTag("playerghost") and caster.components and caster.components.inventory then
        local pos = Vector3(caster.Transform:GetWorldPosition())
        local ents = TheSim:FindEntities(pos.x,pos.y,pos.z, 3) --3是拾取范围
                for k, v in pairs(ents) do
                    if v.components.inventoryitem ~= nil and
                    v.components.inventoryitem.canbepickedup and
                    v.components.inventoryitem.cangoincontainer and
                    not v.components.inventoryitem:IsHeld() and 
                    not v:HasTag("trap") and not v:HasTag("light") and not v:HasTag("blowdart") and not v:HasTag("projectile") and not v:HasTag("custom_cantquickpick") and
                    caster.components.inventory:CanAcceptCount(v, 1) > 0 then
                    SpawnPrefab("sand_puff").Transform:SetPosition(v.Transform:GetWorldPosition())

                    local v_pos = v:GetPosition()
                    caster.components.inventory:GiveItem(v, nil, v_pos)
                    end
                end
end
```
`modmain.lua`
```lua
--添加一些不可拾取的物品,例如马鞍,骑马时拾取马鞍会报错
local cantquickpicktab = {
   "saddle_basic",
   "saddle_war",
   "saddle_race",
}
local function cantquickpick( inst )
	if not inst:HasTag("custom_cantquickpick") then
   	inst:AddTag("custom_cantquickpick")
	end
end
for k,v in pairs(cantquickpicktab) do
   AddPrefabPostInit(v, cantquickpick) 
end
```

## 自定义打包袋物品

```lua
--自定义打包袋物品,只有4个格子所以最多4个物品
local pack_items = {}
pack_items = {
    SpawnPrefab("tomato"),
    SpawnPrefab("tomato"),
    SpawnPrefab("tomato"),
    SpawnPrefab("tomato"),
}
local package = SpawnPrefab("gift")
package.components.unwrappable:WrapItems(pack_items)
```

## 成组交易

每次交易掉手上拿的所有物品;不改组件;注意每次交易判断通过后,手上的东西会先被交易掉一个

下面一例为交易换取金子

<div class="output_wrapper" id="output_wrapper_id" style="font-size: 16px; color: rgb(62, 62, 62); line-height: 1.6; word-spacing: 0px; letter-spacing: 0px; font-family: 'Helvetica Neue', Helvetica, 'Hiragino Sans GB', 'Microsoft YaHei', Arial, sans-serif;"><pre style="font-size: inherit; color: inherit; line-height: inherit; margin: 0px; padding: 0px;"><code class="lua language-lua hljs" style="overflow-wrap: break-word; margin: 0px 2px; line-height: 18px; font-size: 14px; font-weight: normal; word-spacing: 0px; letter-spacing: 0px; font-family: Consolas, Inconsolata, Courier, monospace; border-radius: 0px; color: rgb(169, 183, 198); background: rgb(40, 43, 46); overflow-x: auto; padding: 0.5em; white-space: pre !important; word-wrap: normal !important; word-break: normal !important; overflow: auto !important; display: -webkit-box !important;"><span class="hljs-comment" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(128, 128, 128); word-wrap: inherit !important; word-break: inherit !important;">--按组生成预制物</span><br><span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;<span class="hljs-function" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;"><span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">function</span>&nbsp;<span class="hljs-title" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(165, 218, 45); word-wrap: inherit !important; word-break: inherit !important;">SpawnPrefabs_byStack</span><span class="hljs-params" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(255, 152, 35); word-wrap: inherit !important; word-break: inherit !important;">(inst,prefabname,prefabnum)</span></span><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(128, 128, 128); word-wrap: inherit !important; word-break: inherit !important;">--一个通用的生成并抛出预制物的函数</span><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;<span class="hljs-function" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;"><span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">function</span>&nbsp;<span class="hljs-title" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(165, 218, 45); word-wrap: inherit !important; word-break: inherit !important;">SpawnPrefabs_Stack</span><span class="hljs-params" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(255, 152, 35); word-wrap: inherit !important; word-break: inherit !important;">(inst,prefabname,nums)</span></span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;pos&nbsp;=&nbsp;Vector3(inst.Transform:GetWorldPosition())&nbsp;+&nbsp;Vector3(<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">0</span>,<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">4.5</span>,<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">0</span>)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;prefab&nbsp;=&nbsp;SpawnPrefab(prefabname)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;prefab.Transform:SetPosition(pos:Get())<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;down&nbsp;=&nbsp;TheCamera:GetDownVec()<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;angle&nbsp;=&nbsp;<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">math</span>.<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">atan2</span>(down.z,&nbsp;down.x)&nbsp;+&nbsp;(<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">math</span>.<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">random</span>()*<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">60</span><span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">-30</span>)*DEGREES<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;sp&nbsp;=&nbsp;<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">math</span>.<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">random</span>()*<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">4</span>+<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">2</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;prefab.Physics:SetVel(sp*<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">math</span>.<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">cos</span>(angle),&nbsp;<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">math</span>.<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">random</span>()*<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">2</span>+<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">8</span>,&nbsp;sp*<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">math</span>.<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">sin</span>(angle))<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">if</span>&nbsp;prefab.components&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">and</span>&nbsp;prefab.components.stackable&nbsp;~=&nbsp;<span class="hljs-literal" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">nil</span>&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">and</span>&nbsp;nums&nbsp;~=&nbsp;<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">1</span>&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">then</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;prefab.components.stackable.stacksize&nbsp;=&nbsp;nums<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;prefab&nbsp;=&nbsp;SpawnPrefab(prefabname)<br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;prefab_maxsize&nbsp;=&nbsp;<span class="hljs-literal" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">nil</span><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;prefab_stacks&nbsp;=&nbsp;<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">0</span><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;prefab_left&nbsp;=&nbsp;<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">0</span><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">if</span>&nbsp;prefab.components&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">and</span>&nbsp;prefab.components.stackable&nbsp;~=&nbsp;<span class="hljs-literal" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">nil</span>&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">then</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;prefab_maxsize&nbsp;=&nbsp;prefab.components.stackable.maxsize<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;prefab_stacks&nbsp;=&nbsp;<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">math</span>.<span class="hljs-built_in" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">floor</span>(prefabnum/prefab_maxsize)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;prefab_left&nbsp;=&nbsp;prefabnum&nbsp;-&nbsp;prefab_stacks*prefab_maxsize<br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">if</span>&nbsp;prefab_maxsize&nbsp;~=&nbsp;<span class="hljs-literal" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">nil</span>&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">then</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">if</span>&nbsp;prefab_stacks&nbsp;&gt;&nbsp;<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">0</span>&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">then</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">for</span>&nbsp;k=<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">1</span>,prefab_stacks&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">do</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpawnPrefabs_Stack(inst,prefabname,prefab_maxsize)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">if</span>&nbsp;prefab_left&nbsp;&gt;&nbsp;<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">0</span>&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">then</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpawnPrefabs_Stack(inst,prefabname,prefab_left)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">else</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">for</span>&nbsp;k=<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">1</span>,prefabnum&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">do</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpawnPrefabs_Stack(inst,prefabname,<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">1</span>)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br><span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br><span class="hljs-comment" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(128, 128, 128); word-wrap: inherit !important; word-break: inherit !important;">--交易金子</span><br><span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;<span class="hljs-function" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;"><span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">function</span>&nbsp;<span class="hljs-title" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(165, 218, 45); word-wrap: inherit !important; word-break: inherit !important;">DoTradeForGold</span><span class="hljs-params" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(255, 152, 35); word-wrap: inherit !important; word-break: inherit !important;">(inst,giver,item)</span></span><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">if</span>&nbsp;item.components&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">and</span>&nbsp;item.components.tradable&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">and</span>&nbsp;item.components.tradable.goldvalue&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">and</span>&nbsp;item.components.tradable.goldvalue&gt;<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">0</span>&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">then</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;item_goldvalue&nbsp;=&nbsp;item.components.tradable.goldvalue<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">if</span>&nbsp;giver.components&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">and</span>&nbsp;giver.components.inventory&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">and</span>&nbsp;<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;giver.components.inventory.activeitem&nbsp;~=&nbsp;<span class="hljs-literal" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">nil</span>&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">then</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;activenums&nbsp;=&nbsp;<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">0</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">local</span>&nbsp;total_goldvalue&nbsp;=&nbsp;activenums*item_goldvalue<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">if</span>&nbsp;giver.components.inventory.activeitem.components.stackable&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">then</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;activenums&nbsp;=&nbsp;giver.components.inventory.activeitem.components.stackable:StackSize()<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;total_goldvalue&nbsp;=&nbsp;activenums*item_goldvalue<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;giver.components.inventory.activeitem:Remove()<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpawnPrefabs_byStack(inst,<span class="hljs-string" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(238, 220, 112); word-wrap: inherit !important; word-break: inherit !important;">"goldnugget"</span>,total_goldvalue+<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">1</span>*item_goldvalue)<br><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">else</span><br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpawnPrefabs_byStack(inst,<span class="hljs-string" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(238, 220, 112); word-wrap: inherit !important; word-break: inherit !important;">"goldnugget"</span>,<span class="hljs-number" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(174, 135, 250); word-wrap: inherit !important; word-break: inherit !important;">1</span>*item_goldvalue)<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br>&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br><span class="hljs-keyword" style="font-size: inherit; line-height: inherit; margin: 0px; padding: 0px; color: rgb(248, 35, 117); word-wrap: inherit !important; word-break: inherit !important;">end</span><br></code></pre></div>

## 获取owner

当物品在玩家的物品栏,背包,物品栏或背包中的其他容器或该容器套娃的容器中,获取玩家owner

```lua
local function GetOwnerPlayer(invitem)
    local _player = nil
    if invitem.components and invitem.components.inventoryitem then
        local seekowner = invitem.components.inventoryitem.owner
        while seekowner ~= nil do
            if seekowner:HasTag("player") then
                _player = seekowner
                break
            elseif seekowner.components and seekowner.components.container and
            seekowner.components.inventoryitem and seekowner.components.inventoryitem.owner then
                seekowner = seekowner.components.inventoryitem.owner
            else
                break
            end
        end
    end
    return _player
end
```
