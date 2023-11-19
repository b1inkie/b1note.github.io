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
