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
--local caster
for rangs = 1, 10 do
    inst:DoTaskInTime(rangs, function() 
        local ents = TheSim:FindEntities(pos.x,pos.y,pos.z,rangs*2)
        for k,v in pairs(ents) do
            --农作物有farm_plant的标签
            if v.components.pickable and v:HasTag("farm_plant") then  
                SpawnPrefab("sand_puff").Transform:SetPosition(v.Transform:GetWorldPosition()) --加采集特效,可以不加,防卡顿
                local seed_pt = Vector3(v.Transform:GetWorldPosition()) + Vector3(math.random(0,3),math.random(0,3),math.random(0,3)) - Vector3(math.random(0,3),math.random(0,3),math.random(0,3))

                local prefix = "farm_plant_"
                local seed_name = string.sub(v.prefab,1+string.len(prefix)).."_seeds"
                SpawnPrefab(seed_name).Transform:SetPosition(seed_pt:Get())
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

## 成组生成并抛出物品

><i style="color:aqua;">成组生成并抛出预制物</i>

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

><i style="color:aqua;">简单生成并抛出预制物</i>

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

## 自定义打包袋物品

><i style="color:aqua;">简易定义打包袋</i>

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

><i style="color:aqua;">生成包裹并抛出或给予玩家</i>

```lua
function WrapItems(target,SpawnAndThrowOut,GiveToTargetPlayer,...)
    --(target,生成并抛出,给予target玩家,...)
    local packs = {...} --{预制物A,数量,预制物B,数量...}
    local _packs = {}
    for k=1,#packs,2 do
        if type(packs[k]) == "string" then
            if SpawnPrefab(packs[k]) and SpawnPrefab(packs[k]).components then
                --预制物名需存在
                _packs[k] = packs[k]
                if type(packs[k+1]) == "number" then
                    _packs[k+1] = packs[k+1]
                else
                    _packs[k+1] = 1
                end
            end
        end
    end
    local new_packs = {}
    for k,v in pairs(_packs) do
        if v~=nil then
            table.insert(new_packs,v)
        end
    end
    if #new_packs == 0 then
        return
    end
    --生成包裹
    local gift_tab = {}
    local len = #new_packs
    if len > 8 then
        len = 8
    end
    local _i = 0
    for k=1,len,2 do
        _i = _i + 1
        gift_tab[_i] = SpawnPrefab(new_packs[k])
        if gift_tab[_i].components.stackable then
            local _maxsize = gift_tab[_i].components.stackable.maxsize
            if _maxsize < new_packs[k+1] then
                new_packs[k+1] = _maxsize
            end
            gift_tab[_i].components.stackable.stacksize = new_packs[k+1]
        end
    end
    local gift = SpawnPrefab("gift")
    gift.components.unwrappable:WrapItems(gift_tab)
    --抛出或给予玩家
    if SpawnAndThrowOut then
        --页内的函数
        SpawnSinglePrefab_ThrowOut(target,gift) 
        return
    elseif GiveToTargetPlayer then
        if target and target.components and target.components.inventory then
            target.components.inventory:GiveItem(gift)
        end
        return
    end
    return gift
end
```

## 成组交易

每次交易掉手上拿的所有物品;不改组件;注意每次交易判断通过后,手上的东西会先被交易掉一个

下面一例为交易换取金子

```lua
--交易金子
local function DoTradeForGold(inst,giver,item)
    if item.components and item.components.tradable and item.components.tradable.goldvalue and item.components.tradable.goldvalue>0 then
        local item_goldvalue = item.components.tradable.goldvalue

        if giver.components and giver.components.inventory and 
        giver.components.inventory.activeitem ~= nil then
            local activenums = 0
            local total_goldvalue = activenums*item_goldvalue
            if giver.components.inventory.activeitem.components.stackable then
                activenums = giver.components.inventory.activeitem.components.stackable:StackSize()
                total_goldvalue = activenums*item_goldvalue
            end
            giver.components.inventory.activeitem:Remove()
            --页内的函数
            SpawnPrefabs_byStack(inst,"goldnugget",total_goldvalue+1*item_goldvalue)
        else
            --页内的函数
            SpawnPrefabs_byStack(inst,"goldnugget",1*item_goldvalue)
        end
    end
end
```

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
