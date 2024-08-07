## 复杂的武器伤害

武器除了面板伤害,我们还可以添加连击伤害,暴击伤害,怪物杀手即对特定怪造成更多伤害等等

变量设置:
拿暴击几率来说,除了当前暴击几率和设定好的初始暴击几率,还需要一个变量作为暴击几率计算的依据
所以一般来说有3个变量

好了我们来给武器加点料:
1. 武器面板可成长
2. 连击(越打越疼)
3. 暴击(概率触发后伤害更高)
4. 斩杀(对低血量怪物造成更多伤害)
5. AOE伤害(每攻击3次后5秒内的下一次攻击附带AOE伤害)
6. 巨人杀手(对血量高于一定值并当前血量占比超过设定比例的怪物造成更多伤害)
7. 怪物杀手(对特定怪造成更多伤害)
8. 断筋(移动掉血)

`scripts/components/elucidator_sys.lua`
<!-- 
```lua
local elucidator_sys = Class(function(self, inst)
    self.inst = inst

    self.combo = 0 --连击系统临时用一下

    self.aoe = 0 --AOE

    self.basis_weapon_panel_bonus = 0 --武器面板伤害计算依据
    self.weapon_panel_bonus = 0 --武器面板伤害需要加上该值

    self.init_critical_chance = 5 --//设置初始暴击率5%
    self.init_critical_dmg_mult = 1.2 --//设置初始暴击倍率
    self.basis_critical_chance = 0 --暴击率计算依据
    self.basis_critical_dmg_mult = 0 --暴击倍率计算依据
    self.critical_chance = 5 --当前暴击率5%
    self.critical_dmg_mult = 1.2 --当前暴击倍率

    self.beheaded_line = 0.2 --斩杀血量占比
    self.beheaded_dmg_mult = 1.2 --伤害倍率

    self.giantkiller_line = 0.9 --巨人杀手血量比例最低值
    self.giantkiller_dmg_mult = 1.4 --伤害倍率

    self.init_hamstring_chance = 0.2 --断筋充能概率
    self.hamstring_charge_stage = 0 --当前断筋充能层数
    self.hamstring_charge_stage_max = 5 --允许下一击断筋的层数
    self.hamstring_dmg = 8 --断筋伤害
    self.hamstring_period_time = 0.01 --怪物被断筋后的检测频率
    self.hamstring_last_time = 10 --断筋持续时间

    --设置对某种怪物造成更多伤害(该伤害应最后计算)
    self.killer = {0}
end,
nil,
{
})
function elucidator_sys:OnSave()
    local data = {
        combo = self.combo,
        aoe = self.aoe,
        basis_weapon_panel_bonus = self.basis_weapon_panel_bonus,
        weapon_panel_bonus = self.weapon_panel_bonus,
        init_critical_chance = self.init_critical_chance,
        init_critical_dmg_mult = self.init_critical_dmg_mult,
        basis_critical_chance = self.basis_critical_chance,
        basis_critical_dmg_mult = self.basis_critical_dmg_mult,
        critical_chance = self.critical_chance,
        critical_dmg_mult = self.critical_dmg_mult,
        beheaded_line = self.beheaded_line,
        beheaded_dmg_mult = self.beheaded_dmg_mult,
        giantkiller_line = self.giantkiller_line,
        giantkiller_dmg_mult = self.giantkiller_dmg_mult,

        init_hamstring_chance = self.init_hamstring_chance,
        hamstring_charge_stage = self.hamstring_charge_stage,
        hamstring_charge_stage_max = self.hamstring_charge_stage_max,
        hamstring_dmg = self.hamstring_dmg,
        hamstring_period_time = self.hamstring_period_time,
        hamstring_last_time = self.hamstring_last_time,
        killer = self.killer,
    }
    return data
end
function elucidator_sys:OnLoad(data)
    self.combo = 0
    self.aoe = 0
    self.basis_weapon_panel_bonus = data.basis_weapon_panel_bonus or 0
    self.weapon_panel_bonus = data.weapon_panel_bonus or 0
    self.init_critical_chance = data.init_critical_chance or 5
    self.init_critical_dmg_mult = data.init_critical_dmg_mult or 1.2
    self.basis_critical_chance = data.basis_critical_chance or 0
    self.basis_critical_dmg_mult = data.basis_critical_dmg_mult or 0
    self.critical_chance = data.critical_chance or 5
    self.critical_dmg_mult = data.critical_dmg_mult or 1.2
    self.beheaded_line = data.beheaded_line or 0.2
    self.beheaded_dmg_mult = data.beheaded_dmg_mult or 1.2
    self.giantkiller_line = data.giantkiller_line or 0.9
    self.giantkiller_dmg_mult = data.giantkiller_dmg_mult or 1.4

    self.init_hamstring_chance = data.init_hamstring_chance or 0.2
    self.hamstring_charge_stage = 0
    self.hamstring_charge_stage_max = data.hamstring_charge_stage_max or 5
    self.hamstring_dmg = data.hamstring_dmg or 8
    self.hamstring_period_time = data.hamstring_period_time or 0.01
    self.hamstring_last_time = data.hamstring_last_time or 5
    self.killer = data.killer or {0}
end
return elucidator_sys
```
读存也放进去了,方便复制粘贴

## 各类伤害的计算

先写怪物杀手,这个比较特殊:
```lua
--这个表用来存怪的预制物名字,一组一组往里面添加
local killer_tab = {
    --self.killer[1]
    {
        "spider",
        "spiderqueen",
        "spider_dropper",
        "spider_hider",
        "spider_moon",
        "spider_warrior",
        "spider_spitter",
        "spider_healer",
    },
    --self.killer[2]
    {
        "hound",
        "firehound",
        "icehound",
    },
}

local function killer(self,target,mobstab)
    local dmg = 0
    if #mobstab > 0 then
        for t=1,#mobstab do
            for k,v in pairs(mobstab[t]) do
                if target.prefab == v then
                    dmg = self.killer[t]
                end
            end
        end
    end
    --如果对象判断通过,则返回伤害值,注意有可能返回的是nil
    return dmg 
end
----------------
--怪物杀手伤害计算
----------------
function elucidator_sys:calc_killer(num,dmg)
    self.killer[num] = dmg
end
```

其他几项:
```lua
---------------------------
--几样数据的计算依据和最终计算
---------------------------
--武器面板(计算依据是武器获得的黄金价值)
function elucidator_sys:delta_panel(delta)
    self.basis_weapon_panel_bonus = self.basis_weapon_panel_bonus + delta
    self.weapon_panel_bonus = 0.1*math.floor(10*math.exp(1.2*math.log10(self.basis_weapon_panel_bonus)))
end
--暴击率
function elucidator_sys:delta_cc(delta)
    self.basis_critical_chance = self.basis_critical_chance + delta
    self.critical_chance = self.init_critical_chance + 0.2*self.basis_critical_chance
end
--暴击伤害
function elucidator_sys:delta_cdm(delta)
    self.basis_critical_dmg_mult = self.basis_critical_dmg_mult + delta
    self.critical_dmg_mult = self.init_critical_dmg_mult + 0.01*math.floor(10*math.exp(1.6*math.log10(0.6*self.basis_critical_dmg_mult)))
end
--以上3项数据的函数设计的很好了,注意暴击率最好设置一个上限,例如30%
```

## 最终伤害计算

```lua
--
local function tell(inst,line,content,last,colour)
    if not colour then
        colour = {1,1,1,1}
    end
    for k=1,line do
        content = content.." \n "
    end
    inst.components.talker:Say(content,last,true,nil,nil,colour)
end
--
---------
--伤害计算
---------
--[[
TUNING.ELUCIDATOR_DMG = 27        --武器初始面板
TUNING.ALLOW_COMBO_DMG = true     --连击
TUNING.ALLOW_CRITICAL_DMG = true  --暴击
TUNING.ALLOW_BEHEADED = true      --斩杀(对低血量怪物造成更多伤害)
TUNING.ALLOW_AOE = true           --AOE伤害(每攻击3次后5秒内的下一次攻击附带AOE伤害)
TUNING.ALLOW_GIANT_KILLER = true  --巨人杀手(对血量高于一定值并当前血量占比超过设定比例的怪物造成更多伤害)
TUNING.ALLOW_HAMSTRING = true     --断筋(移动后受伤)
]]
function elucidator_sys:atk(inst,attacker,target,bottom)
    --建一个表存伤害倍率
    local dmg_mult = {}
    --------------------------------------------------
    --1--从TUNING表获取是否开启连击系统,在该表设置连击倍率
    dmg_mult[1] = 1
    if TUNING.ALLOW_COMBO_DMG == true then 
        --连击的计算优先级高于暴击
        self.combo = math.min(self.combo+1,5) --每次攻击后连击数+1,最大为5
        --将取消计时器任务写在计时器前面,这样每次攻击就会取消上一次的计时器
        if inst.combo_atk_task ~= nil then 
            inst.combo_atk_task:Cancel()
            inst.combo_atk_task = nil
        end
        inst.combo_atk_task = inst:DoTaskInTime(2, function()
            self.combo = 0
            dmg_mult[1] = 1+0.05*self.combo
        end)
        dmg_mult[1] = 1+0.05*self.combo --连击数对伤害倍率的修正值
    end
    ----------------------------------------------
    --2--如果开启了暴击系统并造成了暴击,在该表设置暴击倍率
    dmg_mult[2] = 1
    if TUNING.ALLOW_CRITICAL_DMG==true and math.random(0,1)<=self.critical_chance then 
        dmg_mult[2] = self.critical_dmg_mult 
        --暴击提示和音效
            attacker.SoundEmitter:PlaySound("dontstarve/common/whip_large")
            tell(inst,3,"暴击!",1,{.9,.1,.1,1})
        --暴击特效
        local fx = SpawnPrefab("impact") 
        fx.Transform:SetScale(math.random(1,5), math.random(1,5), math.random(1,5))
        fx.Transform:SetPosition(target.Transform:GetWorldPosition())
    else
        dmg_mult[2] = 1 
    end
    --------------------------------------
    --3--斩杀(对低血量怪物造成更多伤害)
    dmg_mult[3] = 1
    if TUNING.ALLOW_BEHEADED == true then
        if target.components.health and target.components.health:GetPercent() <= self.beheaded_line then
            dmg_mult[3] = self.beheaded_dmg_mult            
        end
    end
    ---------------------------------
    --4--巨人杀手(对血量高于一定值并当前血量占比超过90%的怪物造成更多伤害)
    dmg_mult[4] = 1
    if TUNING.ALLOW_GIANT_KILLER == true then
        if target.components.health and target.components.health:GetPercent() >= self.giantkiller_line then
            dmg_mult[4] = self.giantkiller_dmg_mult
            local fx_gaint = SpawnPrefab("fx_boat_pop")
            fx_gaint.Transform:SetScale(0.1*math.random(5,13), 0.1*math.random(5,13), 0.1*math.random(5,13))
            fx_gaint.Transform:SetPosition(target.Transform:GetWorldPosition())
        end
    end
    -------------------------------------------
    --E--设置对某种怪物造成更多伤害(该伤害应最后计算)
    local killer_dmg = killer(self,target,killer_tab)
    if killer_dmg == nil or killer_dmg == 0 then
        killer_dmg = 0
    end
    -------------------------------
    --断筋(需要充能,每次攻击有20%几率充能,充能五次,下一次攻击可以断筋)
    if TUNING.ALLOW_HAMSTRING == true then
        if math.random(0,100) <= self.init_hamstring_chance*100 then
            self.hamstring_charge_stage = self.hamstring_charge_stage + 1
            if self.hamstring_charge_stage == self.hamstring_charge_stage_max+1 then
                tell(inst,2,"断筋",2,{.5,0,.5,1})
                local tar_coord = {}
                if target then
                    tar_coord[1], tar_coord[2], tar_coord[3] = target.Transform:GetWorldPosition()
                end
                local function checkMovement()
                    if attacker and target and target.components and target.components.combat and target.components.health then
                        local currentX, currentY, currentZ = target.Transform:GetWorldPosition()
                        if tar_coord[1]~=nil and tar_coord[2]~=nil and tar_coord[3]~=nil then
                            if tar_coord[1]~=currentX or tar_coord[2]~=currentY or tar_coord[3]~=currentZ then
                                target.components.combat:GetAttacked(attacker, self.hamstring_dmg)
                                tar_coord[1], tar_coord[2], tar_coord[3] = currentX, currentY, currentZ
                                --target.components.health:DoDelta(-10)
                                local fx_hamstring = SpawnPrefab("shadowstrike_slash_fx")
                                fx_hamstring.Transform:SetScale(0.1*math.random(3,12),0.1*math.random(3,12),0.1*math.random(3,12))
                                local fx_hamstring_pos = Vector3(target.Transform:GetWorldPosition())
                                fx_hamstring.Transform:SetPosition(fx_hamstring_pos.x+0.3*math.random(1,5)-0.3*math.random(1,5),fx_hamstring_pos.y+0.3*math.random(1,5)-0.3*math.random(1,5),fx_hamstring_pos.z+0.3*math.random(1,5)-0.3*math.random(1,5))
                            end
                        end
                    end
                end
                
                inst.hamstring_task = inst:DoPeriodicTask(self.hamstring_period_time, checkMovement)
                inst:DoTaskInTime(self.hamstring_last_time, function()
                    if inst.hamstring_task ~= nil then
                        inst.hamstring_task:Cancel()
                        inst.hamstring_task = nil
                    end
                end)
                self.hamstring_charge_stage = 0
            end
        end
    end
    --------------------------------------------
    --获取武器面板
    local weapon_panel = TUNING.ELUCIDATOR_DMG + self.weapon_panel_bonus
    --计算最终伤害
    local final_dmg = weapon_panel
    if #dmg_mult > 0 then
        for k=1,#dmg_mult do
            final_dmg = final_dmg*dmg_mult[k]
        end
    end
    final_dmg = final_dmg + killer_dmg
    --计算追加伤害
    local bonus_dmg = final_dmg - weapon_panel
    --造成伤害
    if bonus_dmg > 0 then
        target.components.combat:GetAttacked(attacker, bonus_dmg)
    end
    -------------------------------
    --AOE(每攻击3次后5秒内的下一次攻击附带AOE伤害)
    local function DoAOEAtk(attacker,target,range,dmg)
        local AREAATTACK_MUST_TAGS = { "_combat" }
        local x, y, z = target.Transform:GetWorldPosition()
        local ents = TheSim:FindEntities(x, y, z, range, AREAATTACK_MUST_TAGS)
        for i, ent in ipairs(ents) do
            if ent ~= target and ent ~= attacker and 
                ent.components and ent.components.combat and ent.components.health and
                not ent.components.combat:IsAlly(attacker) then
                ent.components.combat:GetAttacked(attacker, dmg)
            end
        end
    end
    if TUNING.ALLOW_AOE == true then 
        self.aoe = self.aoe+1
        if self.aoe == 4 then --当第4次攻击时,取消倒计时,并在攻击后,将aoe计数器清零
            if inst.aoe_task ~= nil then
                inst.aoe_task:Cancel()
                inst.aoe_task = nil
            end
            --设置AOE伤害
            local aoe_dmg = (TUNING.ELUCIDATOR_DMG + self.weapon_panel_bonus)*1.3
            DoAOEAtk(attacker,target,4,aoe_dmg)
            self.aoe = 0
        elseif self.aoe == 3 then --当攻击3次后,添加一个倒计时,时间到了将aoe计数器清零
            inst.aoe_task = inst:DoTaskInTime(5, function()
                self.aoe = 0
                inst.aoe_task = nil
            end)
        end
    end
end
``` -->

## 击晕(附赠)

`scripts/components/elucidator_sys.lua`
```lua
function elucidator_sys:stun(target,attacker,stun_chance,stun_time)
    ------------------------(target,attacker,眩晕几率,眩晕时间)
    if target.brain and target.components.combat and target.components.locomotor and not target:HasTag("elucidator_stun") and math.random(1,100) <= stun_chance*100 then
        target:AddTag("elucidator_stun")
        SpawnPrefab("lightning_rod_fx").Transform:SetPosition(target.Transform:GetWorldPosition())
        target.components.locomotor:Stop()
        target.brain:Stop()
        target:DoTaskInTime(stun_time,function(target)
            target.brain:Start()
            target:RemoveTag("elucidator_stun")
            end)
    end
end
```
调用:
`scripts/prefabs/mysword.lua`
```lua
local function onattack(inst, attacker, target)
    inst.components.elucidator_sys:stun(target,attacker,0.08,2)
end
inst.components.weapon.onattack = onattack
```

## 对指定怪更易暴击(附赠)

`scripts/prefabs/mysword.lua`
```lua
local function onattack(inst, attacker, target)
    local nightmaremobs = {
        "crawlinghorror",
        "crawlingnightmare",
        "terrorbeak",
        "nightmarebeak",
        "stalker_atrium",
        "shadow_knight",
        "shadow_bishop",
        "shadow_rook",
        "oceanhorror",
        "shadowthrall_horns",
        "shadowthrall_hands",
        "shadowthrall_wings",
        "fused_shadeling",
    }
    --如果已解锁对影怪更容易打出暴击
    if inst:HasTag("could_dohighcc_to_nightmaremobs") then 
        for k,v in pairs(nightmaremobs) do
            if target.prefab == v and not target:HasTag("elucidator_dohighcc") then
                --给怪物加上容易打出暴击的tag,直接写在onattack里好处是不用考虑读存的问题了
                target:AddTag("elucidator_dohighcc")
            end
        end
        if target:HasTag("elucidator_dohighcc") then
            inst.components.elucidator_sys:atk(inst,attacker,target,35)
        else
            inst.components.elucidator_sys:atk(inst,attacker,target,100)
        end
    else
        inst.components.elucidator_sys:atk(inst,attacker,target,100)
    end
end
inst.components.weapon.onattack = onattack
```
最好都写在components里面,因为是附赠的,所以就这样了
