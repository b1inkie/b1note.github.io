## 创建一个工具库

```lua
local tools = {}

function tools:readMe()
    print("This is a tools library")
end

return tools
```

## 坐标轴

进游戏逆时针转一次视角,朝下为x,朝右为z,朝着x轴正方向为 *默认方向* ,spriter中是朝右为x

## 获取人物面前的坐标

人物面前距离dist的点的坐标计算

```lua
function tools:calcCoordFront(inst,dist)
    local angle = inst.Transform:GetRotation()
    local x,_,z = inst.Transform:GetWorldPosition()
    local radian_angle = (angle-90) * math.pi / 180
    return x - dist * math.sin(radian_angle), z - dist * math.cos(radian_angle)
end
```

## 获取默认方向到指定方向的转角

现有两个点 A`(x1,z1)` B`(x2,z2)`,获取默认方向,到向量AB的转角

```lua
function tools:angleBetweenVectors(x1, z1, x2, z2)
    -- @param :从点(x1,z1)到点(x2,z2)的向量
    local x0,z0 = x1 + 1,z1
    local vec1X,vec1Z = x0 - x1,z0 - z1 -- 默认方向向量
    local vec2X,vec2Z = x2 - x1,z2 - z1 -- 向量AB

    local dotProduct = vec1X * vec2X + vec1Z * vec2Z -- 点乘
    local magA,magB = math.sqrt(vec1X^2+vec1Z^2),math.sqrt(vec2X^2+vec2Z^2) -- 向量模长
    
    local cosTheta = dotProduct / (magA * magB) -- 余弦值
    local theta = math.deg(math.acos(cosTheta)) -- 角度
    if z2-z1>0 then theta = 360-theta end -- 旋转是固定顺时针的,所以要转的角度超过180度时,要...
    return theta
end
```

## 获取两点间的距离

```lua
function tools:calcDist(x1,z1,x2,z2,do_sqrt)
    -- @param: do_sqrt 是否开平方
    local dist = (x1-x2)^2+(z1-z2)^2
    if do_sqrt then dist = math.sqrt(dist) end
    return dist
end
```

## 获取直线上的某个点

在给定的一条直线上找到距离起点 `(x2, z2)` 指定距离 `n` 的点。这条直线由两点 `(x1, z1)` 和 `(x2, z2)` 定义

```lua
function tools:findPointOnLine(x1, z1, x2, z2, dist, n)
    local dx,dz = x1-x2,z1-z2
    local unitDX,unitDZ = dx/dist,dz/dist
    local newX, newZ = x2+unitDX*n ,z2+unitDZ*n
    return newX, newZ
end
```

## 获取圆周上的某个点

```lua
function tools:findPointOnCircle(x,z,radius,direction,angle)
    -- @param: direction 1:顺时针 -1:逆时针
    -- @param: angle 角度()
    -- @param: radius 半径
    angle = angle*direction
    local des_x = x + math.cos(math.rad(angle))*radius
    local des_z = z + math.sin(math.rad(angle))*radius
    
    return des_x, des_z
end
```

## 射线

按键朝人物方向发射射线,射线覆盖的敌人持续受伤且会被击退,再按一次取消射线

预制物方面:
```lua 
-- MakeInventoryPhysics(inst) -- 这句可以注释掉

inst.AnimState:PlayAnimation('idle',true) -- 循环播放
inst.AnimState:SetOrientation(ANIM_ORIENTATION.OnGround) -- 设置为俯视角,这样就可以360度转了
inst.AnimState:SetSortOrder(5) -- 设置覆盖优先度 1为地面
```

生成射线:
```lua
local Tools = require 'tools' -- 把模块加载进来     

TheInput:AddKeyDownHandler(GLOBAL.KEY_H, function() 
    if TheFrontEnd:GetActiveScreen() and TheFrontEnd:GetActiveScreen().name == 'HUD' then 
        local inst = ThePlayer

        if inst.fx_laser == nil then 
        inst.components.playercontroller:Enable(false) -- 射线发射中,取消玩家控制移动
            inst.fx_laser = SpawnPrefab("fx_laserbeem") -- 给inst挂一个属性用来存储特效

            local angle = inst.Transform:GetRotation() -- 获取玩家当前角度
            local x,y,z = inst.Transform:GetWorldPosition() -- 获取玩家当前位置
            local radian_angle = (angle-90) * math.pi / 180 -- 将角度转换为弧度

            
            inst.fx_laser.Transform:SetPosition(x,y+1.5,z) -- 给纵轴一个偏移量，使特效在玩家头部
            inst.fx_laser.Transform:SetRotation(angle) -- 特效角度和人物角度保持一致
            inst.fx_laser.Transform:SetScale(2,2,2) -- 给特效一个缩放看起来更大

            local fx_len = 28 -- 特效在世界中的实际长度(自行测试)
            local iter_range = 2 -- 每次迭代半径,以刚好能覆盖特效宽度为佳

            local pos_inline = {} -- 存储需要进行迭代的圆心坐标
            for dist=iter_range,fx_len,iter_range*2 do -- 生成圆心坐标
                local x2,z2 = x - dist * math.sin(radian_angle), z - dist * math.cos(radian_angle)
                table.insert(pos_inline,{x2,z2})
            end
            
            -- 
            inst.task_period_beem = inst:DoPeriodicTask(0.5,function(inst)
                for _,pos in pairs(pos_inline) do
                    local ents = TheSim:FindEntities(pos[1],0,pos[2],iter_range)
                    for _,v in pairs(ents) do
                        if v and not v:HasTag('player') and v.prefab and v.components and v.components.health and not v.components.health:IsDead() and v.components.combat then 
                            -- 已筛选出范围内的目标
                            local v_x,_,v_z = v.Transform:GetWorldPosition() -- 获取目标位置
                            local dist_btw_v_and_p = Tools:calc_dist(x,z,v_x,v_z,true) -- 计算目标与玩家的距离
                            local des_x,des_z = Tools:findPointOnLine(v_x,v_z,x,z,dist_btw_v_and_p,dist_btw_v_and_p+2) -- 计算目标被击退后的坐标
                            v.Physics:Teleport(des_x,0,des_z)

                            v.components.combat:GetAttacked(inst,20) -- 造成伤害
                        end
                    end
                end
            end)
        else
            inst.components.playercontroller:Enable(true)
            inst.fx_laser:Remove()
            inst.fx_laser = nil
            if inst.task_period_beem then inst.task_period_beem:Cancel()inst.task_period_beem=nil end
        end

    end
end)

```

## 彩虹变色

注意返回的值r,g,b还要/255!

```lua
local function hsv_to_rgb(h, s, v)
    local r, g, b = 0, 0, 0
    local i = math.floor(h * 6)
    local f = h * 6 - i
    local p = v * (1 - s)
    local q = v * (1 - f * s)
    local t = v * (1 - (1 - f) * s)

    local rgb_map = {
        [0] = {v, t, p},
        [1] = {q, v, p},
        [2] = {p, v, t},
        [3] = {p, q, v},
        [4] = {t, p, v},
        [5] = {v, p, q},
    }

    local r, g, b = unpack(rgb_map[i % 6])

    return r, g, b
end

function tools:smoothRainbowColor(t)
    -- @param t: 1,2,3,4,5,...步长可以适当增加
    local h = t % 360 / 360  -- 将时间t映射到0到1之间的值，360度是一个完整的色轮周期
    local s, v = 1, 1        -- 固定饱和度和明度为最大值，以获得鲜艳的颜色

    -- 调用HSV到RGB转换函数
    local r, g, b = hsv_to_rgb(h, s, v)

    -- 将RGB值从[0,1]范围转换到[0,255]范围
    r = math.floor(r * 255 + 0.5)
    g = math.floor(g * 255 + 0.5)
    b = math.floor(b * 255 + 0.5)

    return r, g, b
end
```