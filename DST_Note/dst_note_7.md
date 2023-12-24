## 按键技能

```lua
TheInput = GLOBAL.TheInput
local mymodhandlers = {}
AddModRPCHandler(modname, "SKILL_A", function(player)
	if not player:HasTag("playerghost") then

	end
end)
AddModRPCHandler(modname, "SKILL_B", function(player)
	if not player:HasTag("playerghost") then

	end
end)
AddPlayerPostInit(function(inst)
	inst:DoTaskInTime(0, function()
		if inst == GLOBAL.ThePlayer then
			--if inst.prefab == "charactor_name" then --只允许某个人物使用技能
				mymodhandlers[0] = TheInput:AddKeyDownHandler(TUNING.CONFIG_SKILL_A, function()
					local screen = GLOBAL.TheFrontEnd:GetActiveScreen()
            		local IsHUDActive = screen and screen.name == "HUD"
            		if inst:IsValid() and IsHUDActive then
            			SendModRPCToServer(MOD_RPC[modname]["SKILL_A"])
            		end
				end)
				mymodhandlers[1] = TheInput:AddKeyDownHandler(TUNING.CONFIG_SKILL_B, function()
					local screen = GLOBAL.TheFrontEnd:GetActiveScreen()
            		local IsHUDActive = screen and screen.name == "HUD"
            		if inst:IsValid() and IsHUDActive then
            			SendModRPCToServer(MOD_RPC[modname]["SKILL_B"])
            		end
				end)
			--else
				--for k, v in pairs(mymodhandlers) do
					--mymodhandlers[k] = nil
				--end
			--end
		end
	end)
end)
```
