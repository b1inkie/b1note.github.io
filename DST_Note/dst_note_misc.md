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
        cooldown_on = self.cooldown_on,
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