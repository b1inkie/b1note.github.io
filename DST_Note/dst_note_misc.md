## 杂项功能合集

><i style="color:aqua;">是否写了一些常用的功能函数,例如背包中套娃的容器中的物品,需要获取最上一级的owner即玩家,诸如此类做个合集</i>

先于`/scripts`下创建一个`.lua`
例:`/scripts/util/mymodulefn.lua`
```lua
local commonfns = {}
--私有变量（模块内使用）
local privatevar = "private"
--公有变量
commonfns.publicvar = "public"

function commonfns.test()
    print(privatevar)
end
return commonfns
```

接着在你的预制物或其他文件中加载并使用这个模块:

```lua
--写在首行 加载
local TestFn = require("util/mymodulefn")

TestFn.test() --输出 private
print(TestFn.publicvar) --输出 public
```

## 简易冷却

<details> <summary>代码点我展开</summary>

```lua
    if not inst:HasTag("iscooldown") then
        print("施法!")
        inst:AddTag("iscooldown")
    end
    inst:DoTaskInTime(cooldown_time, function()
        inst:RemoveTag("iscooldown")
    end)
```

</details>

## 自动拾取附近物品

<details> <summary>代码点我展开</summary>

```lua
    -- local player
    local pos = Vector3(player.Transform:GetWorldPosition())
    local ents = TheSim:FindEntities(pos.x,pos.y,pos.z, 3)
    for k, v in pairs(ents) do
        if v.components.inventoryitem ~= nil and
        v.components.inventoryitem.canbepickedup and
        v.components.inventoryitem.cangoincontainer and
        not v.components.inventoryitem:IsHeld() and 
        not v:HasTag("trap") and not v:HasTag("light") and not v:HasTag("blowdart") and not v:HasTag("projectile") and not v:HasTag("custom_cantquickpick") and
        player.components.inventory:CanAcceptCount(v, 1) > 0 then
            SpawnPrefab("sand_puff").Transform:SetPosition(v.Transform:GetWorldPosition())

            local v_pos = v:GetPosition()
            player.components.inventory:GiveItem(v, nil, v_pos)
            --return
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

</details>

## 成组生成并抛出物品

><i style="color:aqua;">成组生成并抛出预制物</i>

<details> <summary>代码点我展开</summary>

```lua
function SpawnPrefabs_byStack(inst,prefabname,prefabnum)
    local function SpawnPrefabs_Stack(inst,prefabname,nums)
        local pos = Vector3(inst.Transform:GetWorldPosition()) + Vector3(0,4.5,0)
        local prefab = SpawnPrefab(prefabname)
        prefab.Transform:SetPosition(pos:Get())
        local down = TheCamera:GetDownVec()
        local angle = math.atan2(down.z, down.x) + (math.random()*60-30)*DEGREES
        local sp = math.random()*4+2
        prefab.Physics:SetVel(sp*math.cos(angle), math.random()*2+8, sp*math.sin(angle))
        if prefab.components and prefab.components.stackable ~= nil and nums ~= 1 then
            prefab.components.stackable.stacksize = nums
        end
    end
    local prefab = SpawnPrefab(prefabname)
    local prefab_maxsize = prefab.components and prefab.components.stackable and prefab.components.stackable.maxsize
    local prefab_stacks = prefab_maxsize and math.floor(prefabnum/prefab_maxsize) 
    local prefab_left = prefab_maxsize and (prefabnum - prefab_stacks*prefab_maxsize)

    if prefab_maxsize then
        if prefab_stacks and prefab_stacks > 0 then
            for k=1,prefab_stacks do
                SpawnPrefabs_Stack(inst,prefabname,prefab_maxsize)
            end
        end
        if prefab_left and prefab_left > 0 then
            SpawnPrefabs_Stack(inst,prefabname,prefab_left)
        end
    else
        for k=1,prefabnum do
            SpawnPrefabs_Stack(inst,prefabname,1)
        end
    end
end
```

</details>

><i style="color:aqua;">简单生成并抛出预制物</i>

<details> <summary>代码点我展开</summary>

```lua
function SpawnSinglePrefab_ThrowOut(tar,item)
    local pt = Vector3(tar.Transform:GetWorldPosition()) + Vector3(0,4.5,0)
    item.Transform:SetPosition(pt:Get())
    local down = TheCamera:GetDownVec()
    local angle = math.atan2(down.z, down.x) + (math.random()*60-30)*DEGREES
    local sp = math.random()*4+2
    item.Physics:SetVel(sp*math.cos(angle), math.random()*2+8, sp*math.sin(angle))
end
```

</details>

## 自定义打包袋物品


## 获取owner

当物品在玩家的物品栏,背包,物品栏或背包中的其他容器或该容器套娃的容器中,获取玩家owner

<details> <summary>代码点我展开</summary>

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

</details>

## 服务器宣告

约等于能直接显示在聊天框的print

<details> <summary>代码点我展开</summary>

```lua
local function declare(icon,...)
    -- @param icon:"vote";"leave_game";"join_game";"death";"resurrect";"mod";"kicked_from_game";"banned_from_game";"dice_roll"
    -- @param ...:string
    local s = ''
    for i = 1, select('#', ...) do s = s .. tostring(select(i, ...)) .. ' ' end
    TheNet:Announce(s,nil,nil,icon)
end
```

<details>

## 打乱数组(Fish-Yates shuffle)

<details> <summary>代码点我展开</summary>

```lua
function shuffleTab(tbl,k)
    -- @param tbl: 需要打乱的表
    -- @param k: k次，因为是倒着遍历的,所以洗好的表也要倒着拿数据,不填则完全打乱
    k = math.min(k or #tbl,#tbl)
    if k > 1 then
        for i = #tbl, #tbl-k+1, -1 do
            local j = math.random(i)
            tbl[i], tbl[j] = tbl[j], tbl[i]
        end
    end
end
```

<details>

## 深拷贝

<details> <summary>代码点我展开</summary>

```lua 
function deepCopy(orig)
    local orig_type = type(orig)
    local copy
    if orig_type == 'table' then
        copy = {}
        for orig_key, orig_value in ipairs(orig) do
            copy[self:deepCopy(orig_key)] = self:deepCopy(orig_value)
        end
        setmetatable(copy, self:deepCopy(getmetatable(orig)))
    else -- number, string, boolean, etc.
        copy = orig
    end
    return copy
end
```

</details>