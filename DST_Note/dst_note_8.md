## 添加动作

```lua
STRINGS.MYMOD_ACTION = {
	REPAIRARMORWOOD = "修复护甲",
}

local function RepairFn(items,target)
	if target.components and target.components.armor then
		target.components.armor:SetPercent(target.components.armor:GetPercent()+.1)
        if items.components.stackable then
            items.components.stackable:Get():Remove()
        else
            items:Remove()
        end
    end
	return true
end

local actions = {
	{
		id = "REPAIRARMORWOOD", --动作ID
		str = STRINGS.MYMOD_ACTION.REPAIRARMORWOOD, --动作显示文字
		fn = function(act)
			if act.doer ~= nil and act.invobject ~= nil and act.target ~= nil and 
            act.invobject.prefab == "boards" and act.target.prefab == "armorwood" then
				return RepairFn(act.invobject,act.target) --动作执行函数:修复护甲耐久
			end
		end,
		state = "give", --绑定sg
		actiondata = {
			priority = 99,
			mount_valid = true,
		},
	},
    --[[
	{
		id = "XXX",
		str = STRINGS.MYMOD_ACTION.XXX,
		fn = function(act)

		end,
		state = "give",
		actiondata = {
			priority = 90,
			mount_valid = true,
		},
	},
    ]]
}
--绑定组件
local component_actions = {
	{
		type = "USEITEM", --动作类型
		component = "inventoryitem",
		tests = { --尝试显示
			{
				action = "REPAIRARMORWOOD",
				testfn = function(inst, doer, target, actions, right)
					return doer:HasTag("player") and inst.prefab=="boards" and target.prefab=="armorwood"
				end,
			},
            --[[
            {
				action = "XXX",
				testfn = function(inst, doer, target, actions, right)
					
				end,
			},
            ]]
		},
	},
}

for _,act in pairs(actions) do
    local addaction = AddAction(act.id,act.str,act.fn)
    if act.actiondata then
        for k,v in pairs(act.actiondata) do
            addaction[k] = v
        end
    end
    AddStategraphActionHandler("wilson",GLOBAL.ActionHandler(addaction, act.state))
    AddStategraphActionHandler("wilson_client",GLOBAL.ActionHandler(addaction,act.state))
end

for _,v in pairs(component_actions) do
    local testfn = function(...)
        local actions = GLOBAL.select (-2,...)
        for _,data in pairs(v.tests) do
            if data and data.testfn and data.testfn(...) then
                data.action = string.upper(data.action)
                table.insert(actions,GLOBAL.ACTIONS[data.action])
            end
        end
    end
    AddComponentAction(v.type, v.component, testfn)
end
```