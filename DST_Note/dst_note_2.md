## component应用

为人物添加升级系统
例:人物可通过某种方式解锁升级系统;解锁后人物可通过某种方式升级饥饿和血量上限

## 为人物添加成分

modmain中
```lua
local function initPalyer(inst)
   inst:AddComponent("upgradesys")
end

if GLOBAL.TheNet:GetIsServer() then --人物三围都放在服务器计算 所以加这句判断
   AddPlayerPostInit(initPalyer)
end
```
下面来写成分`"upgradesys"`

## 成分components编写

`/scripts/components/upgradesys.lua`
```lua
local upgradesys = Class(function(self, inst)
    self.inst = inst

    self.unlock = 0 --是否解锁升级系统 0否 1是

    self.basis_hunger = 0 --饥饿上限计算依据
    self.basis_hp = 0 --血上限计算依据

    --初始化4项数据 分别是饥饿上限 血上限 当前饥饿占比 当前血占比
    self.max_hunger = 1
    self.max_hp = 1
    self.percent_hunger = 1
    self.percent_hp = 1

    if self.unlock == 1 then --初始时未解锁 所以是不执行的
        inst.components.hunger.max = self.max_hunger
        inst.components.health.maxhealth = self.max_hp
    end
end,
nil,
{
})
-----------
--save&load
-----------
function upgradesys:OnSave()
    local data = {
        unlock = self.unlock,

        basis_hunger = self.basis_hunger,
        basis_hp = self.basis_hp,
        
        --存储饥饿上限 血上限 当前饥饿占比 当前血占比
        max_hunger = self.inst.components.hunger.max,
        max_hp = self.inst.components.health.maxhealth,
        percent_hunger = self.inst.components.hunger:GetPercent(),
        percent_hp = self.inst.components.health:GetPercent(),
    }
    return data
end
function upgradesys:OnLoad(data)
    self.unlock = data.unlock or 0

    self.basis_hunger = data.basis_hunger or 0
    self.basis_hp = data.basis_hp or 0

    self.max_hunger = data.max_hunger or 1
    self.max_hp = data.max_hp or 1
    self.percent_hunger = data.percent_hunger or 1
    self.percent_hp = data.percent_hp or 1

    local inst = self.inst
    --读取时 判断人物是否有饥饿和血量组件及是否解锁升级系统
    if inst.components.hunger and inst.components.health and inst.components.upgradesys.unlock == 1 then
        inst.components.hunger.max = data.max_hunger
        inst.components.health.maxhealth = data.max_hp
        inst.components.hunger:SetPercent(data.percent_hunger)
        inst.components.health:SetPercent(data.percent_hp)
    end
end
---------
--升级方式
---------
function upgradesys:lvup_hunger(guy)
    self.basis_hunger = self.basis_hunger + 1 --升级依据,自行设置
    local hunger_percent = guy.components.hunger:GetPercent()
    guy.components.hunger.max = math.ceil(guy.components.hunger.max+1) --升级函数,自行设置
    guy.components.hunger:SetPercent(hunger_percent) --改变最大值后 决定重新计算占比
end
function upgradesys:lvup_hp(guy)
    self.basis_hp = self.basis_hp + 1
    local hp_percent = guy.components.health:GetPercent()
    guy.components.health.maxhealth = math.ceil(guy.components.health.maxhealth+1)
    guy.components.health:SetPercent(hp_percent)
end
--
return upgradesys
```

## 案例一则

为人物添加一个可解锁的升级系统,杀死怪物,获得经验值(等同于怪物的血量),
初始等级 `lv = 0`,
对应等级的经验条总量为 `math.floor(10000*math.log10(lv+2))`
| 等级 | 经验条总量 |
|:----|----:|
| 0 | 3010 |
| 1 | 4771 |
| 2 | 6020 |
| 3 | 6989 |
| 4 | 7781 |
```lua
local upgradesys = Class(function(self, inst)
    self.inst = inst

    self.unlock = 0 --是否解锁升级系统 0否 1是

    self.lv_cur = 0 -- 初始等级为0
    self.exp_cur_max = 0 --经验条总量
    self.exp_cur = 0 --当前经验条

    self.max_hunger = 1
    self.max_hp = 1
    self.percent_hunger = 1
    self.percent_hp = 1

    if self.unlock == 1 then --初始时未解锁 所以是不执行的
        inst.components.hunger.max = self.max_hunger
        inst.components.health.maxhealth = self.max_hp
    end
end,
nil,
{
})
-----------
--save&load
-----------
--读存于上段代码一致,故略去
---------
--升级方式
---------
function upgradesys:getexp(guy,delta) --当获取经验
    self.exp_cur_max = math.floor(10000*math.log10(self.lv_cur+2))
    self.exp_cur = self.exp_cur + delta
    local canlevelup = {0} --临时弄个表,用来判断玩家是否升级了
    while self.exp_cur > 0 do
        --如果当前经验值大于等于经验条总量,即满足升级条件,则逐级升级,否则跳出循环
        if self.exp_cur >= exp_cur_max then 
            self.exp_cur = self.exp_cur - self.exp_cur_max
            self.lv_cur = self.lv_cur + 1
            canlevelup[1] = 1 --如果玩家升级了,值变为1
            self.exp_cur_max = math.floor(10000*math.log10(self.lv_cur+2))
        else
            break
        end
    end
    if canlevelup[1] == 1 then --若玩家升级了,才会加三围
        --此处设置每升一级,加一点饥饿和血上限
        guy.components.hunger.max = math.ceil(guy.components.hunger.max + self.lv_cur)
        guy.components.health.maxhealth = math.ceil(guy.components.health.maxhealth + self.lv_cur)
    end
    print("等级为："..self.lv_cur)
    print("当前经验值为："..self.exp_cur.."/"..self.exp_cur_max)
end
--
return upgradesys
```
下面我们同过监听玩家杀死怪物的事件,来实现获取经验值

## 事件Event

1. 你可以通过PushEvent("事件",data)来让事件发生,data是一张表,记录发生事件时的数据,然后传给监听对象
2. 用ListenForEvent("事件",fn,source)来监听这个事件,fn固定有两个参数,被监听对象的引用和传递过来的data表,source是被监听对象，此参数可以为空,则默认source是监听对象自身,另外可以通过重复使用ListenForEvent设置多个监听器
3. 可以使用RemoveEventCallback("事件",fn)来移除监听器
继续在upgradesys成分中添加:
```lua
local function onkilled(inst,data)
    local victim = data.victim
    if victim.components.health ~= nil then
        local exp_get = math.ceil(victim.components.health.maxhealth)
        inst.components.upgradesys:getexp(inst,exp_get)
    end   
end
function upgradesys:unlock(guy)
    self.unlock = 1
    guy:ListenForEvent("killed",onkilled)
end
```
每次读档时要重新设置监听器,所以在`function upgradesys:OnLoad(data)`中要添加:
```lua
    local inst = self.inst
    if inst.components.upgradesys.unlock == 1 then
        inst:ListenForEvent("killed",onkilled)
    end
```
