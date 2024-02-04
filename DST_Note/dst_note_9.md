## dps显示

生物死亡后服务器宣告所有参与的玩家的dps
拷打摸鱼队友(

```lua
-- 列出需要显示dps的生物表
local need_dps = {
    "dragonfly",

    "moose",
    "antlion",
    "bearger",
    "deerclops",

    "beequeen",
    "spiderqueen",
    "leif",
    "leif_sparse",
    "lordfruitfly",
    "eyeofterror",
    --"twinofterror1",
    --"twinofterror2",
    
    "toadstool",
    "toadstool_dark",
    
    "stalker_atrium",
    "minotaur",

    "malbatross",
    "crabking",

    "klaus",

    "lunarthrall_plant",
    "mutated_bearger",
    "mutated_deerclops",
    "alterguardian_phase3",
    "mutatedwarg",
    "daywalker",
}
local function dps(inst)
    -- 创建一个挂在该生物上的表用于存伤害值及其来源
    inst.attackers = {}

    local function gotattacked(inst,data)
        -- 60s内未造成伤害则清空dps
        if inst.canceldps ~= nil then inst.canceldps:Cancel() inst.canceldps = nil end
        inst.canceldps = inst:DoTaskInTime(60,function(inst) inst.attackers = {} end)

        -- 获取本次伤害值和其来源
        local damage,attacker = math.floor(data.damage),data.attacker

        -- 统计
        local hasElement = false
        for k,v in pairs(inst.attackers) do
            if v[1] == attacker then 
                inst.attackers[k][2] = inst.attackers[k][2] + damage
                hasElement = true
            end
        end
        if not hasElement then table.insert(inst.attackers,{attacker,damage}) end
    end
    local function ondeath( inst )
        local hp = inst.components and inst.components.health and inst.components.health.maxhealth

        if hp then
            local showdps = {}
            if inst.attackers ~= nil then
                for k,v in pairs(inst.attackers) do
                    if v[1]:HasTag("player") then
                        table.insert(showdps,{v[1]:GetDisplayName(),v[2]})
                    elseif v[1].prefab and GLOBAL.STRINGS.NAMES[string.upper(v[1].prefab)] then
                        table.insert(showdps,{GLOBAL.STRINGS.NAMES[string.upper(v[1].prefab)],v[2]})
                    end
                end
            end
            for k,v in pairs(showdps) do
                local rate = math.min(math.floor(v[2]/hp * 1000)/10,100)
                v[2] = math.min(v[2],hp)
                GLOBAL.TheNet:Announce(v[1]..":"..v[2].."/"..rate.."%",AllPlayers[1].entity,nil,"mod")
            end
        end
        
        inst.attackers = {}
    end
    inst:ListenForEvent("attacked",gotattacked)
    inst:ListenForEvent("death",ondeath)
end

for k,v in pairs(need_dps) do 
    AddPrefabPostInit(v,dps)
end

```
