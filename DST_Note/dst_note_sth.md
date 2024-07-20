## 创建一个工具库

```lua
local tools = {}

function tools:readMe()
    print("This is a tools library")
end

return tools
```

## (下载这个模块)

1. [点我下载](https://github.com/b1inkie/b1note.github.io/releases/download/dst_util/tools_calccoords.lua)
2. 将下载的文件放到例如 `mods\util` 文件夹下
3. 在你们的文件顶部 `local Tools_xxx = require 'util/xxx'` 来加载模块
4. 调用函数计算距离人物面前dist位置的点坐标 `local x,z = Tools_xxx:calcCoordFront(inst,dist)`


## 坐标轴

进游戏逆时针转一次视角,朝下为x,朝右为z,朝着x轴正方向为 *默认方向* ,spriter中是朝右为x

## 获取人物面前的坐标

人物面前距离dist的点的坐标计算

<details> <summary>代码点我展开</summary>

```lua
function tools:calcCoordFront(inst,dist)
    local angle = inst.Transform:GetRotation()
    local x,_,z = inst.Transform:GetWorldPosition()
    local radian_angle = (angle-90) * math.pi / 180
    return x - dist * math.sin(radian_angle), z - dist * math.cos(radian_angle)
end
```

</details>

## 获取默认方向到指定方向的转角

现有两个点 A`(x1,z1)` B`(x2,z2)`,获取默认方向,到向量AB的转角

<details> <summary>代码点我展开</summary>

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

</details>

## 获取两点间的距离

<details> <summary>代码点我展开</summary>

```lua
function tools:calcDist(x1,z1,x2,z2,do_sqrt)
    -- @param: do_sqrt 是否开平方
    local dist = (x1-x2)^2+(z1-z2)^2
    if do_sqrt then dist = math.sqrt(dist) end
    return dist
end
```

</details>

## 获取直线上的某个点

在给定的一条直线上找到距离起点 `(x2, z2)` 指定距离 `n` 的点。这条直线由两点 `(x1, z1)` 和 `(x2, z2)` 定义

<details> <summary>代码点我展开</summary>

```lua
function tools:findPointOnLine(x1, z1, x2, z2, dist, n)
    local dx,dz = x1-x2,z1-z2
    local unitDX,unitDZ = dx/dist,dz/dist
    local newX, newZ = x2+unitDX*n ,z2+unitDZ*n
    return newX, newZ
end
```

</details>

## 获取圆周上的某个点

<details> <summary>代码点我展开</summary>

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

</details>

## 锥形的外切圆

<details> <summary>代码点我展开</summary>

```lua
function tools:coneExcircle(maxRange, centerX, centerZ, pointOnAxis_X, pointOnAxis_Z,angle)
    -- @param maxRange:圆锥的最大长度
    -- @param centerX,centerZ:圆锥的圆心X,Z坐标
    -- @param pointOnAxis_X,pointOnAxis_Z:圆锥中轴线上的任意一个点的X,Z坐标
    -- @param angle:圆锥的张角（角度制）
    -- @return:外切圆的圆心坐标和半径

    -- 将角度转换为弧度
    local angleRad = math.rad(angle)

    -- 计算外切圆的半径
    local radius = maxRange / (1 + math.sin(angleRad / 2))

    -- 计算圆心到锥顶的距离
    local distanceToCenter = radius * math.sin(angleRad / 2)

    -- 计算中轴线的方向向量
    local directionX = pointOnAxis_X - centerX
    local directionZ = pointOnAxis_Z - centerZ
    local directionLength = math.sqrt(directionX * directionX + directionZ * directionZ)

    -- 归一化方向向量
    local normalizedDirectionX = directionX / directionLength
    local normalizedDirectionZ = directionZ / directionLength

    -- 计算外切圆圆心的坐标
    local excircleCenterX = centerX + normalizedDirectionX * distanceToCenter
    local excircleCenterZ = centerZ + normalizedDirectionZ * distanceToCenter

    -- 返回外切圆的圆心坐标和半径
    return {x = excircleCenterX, z = excircleCenterZ, r = radius}
end
```

</details>


## 计算某个点有没有落在圆锥内

<details> <summary>代码点我展开</summary>

```lua 
function tools:isEnemyInCone(checkX, checkZ, centerX, centerZ, pointOnAxis_X, pointOnAxis_Z, coneAngle, maxRange)
    -- @param: checkX, checkZ:要检查的点的坐标
    -- @param: centerX, centerZ:圆锥的圆心坐标
    -- @param: pointOnAxis_X, pointOnAxis_Z:圆锥中轴线上任意一个点的坐标
    -- @param: coneAngle:圆锥的张角（角度制）
    -- @param: maxRange:圆锥的最大长度

    -- 将角度转换为弧度
    local angleRad = math.rad(coneAngle)

    -- 计算圆锥的朝向向量
    local directionX = pointOnAxis_X - centerX
    local directionZ = pointOnAxis_Z - centerZ

    -- 计算检查点到圆锥圆心的向量
    local checkVectorX = checkX - centerX
    local checkVectorZ = checkZ - centerZ

    -- 计算两个向量之间的点积
    local dotProduct = directionX * checkVectorX + directionZ * checkVectorZ

    -- 计算两个向量的模长
    local directionMagnitude = math.sqrt(directionX * directionX + directionZ * directionZ)
    local checkVectorMagnitude = math.sqrt(checkVectorX * checkVectorX + checkVectorZ * checkVectorZ)

    -- 防止除以零错误
    if directionMagnitude == 0 or checkVectorMagnitude == 0 then
        return false
    end

    -- 计算两个向量之间的夹角
    local angle = math.acos(dotProduct / (directionMagnitude * checkVectorMagnitude))

    -- 检查角度是否在圆锥张角之内
    if angle > angleRad / 2 then
        return false
    end

    -- 检查点到圆心的距离是否小于最大射程
    local distance = math.sqrt(checkVectorX * checkVectorX + checkVectorZ * checkVectorZ)
    if distance > maxRange then
        return false
    end

    return true
end
```

</details>



***

## 分割线(以下是实例)



## 射线

按键朝人物方向发射射线,射线覆盖的敌人持续受伤且会被击退,再按一次取消射线

<details> <summary>代码点我展开</summary>

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

</details>


## 移动射线

边走边射,激光跟随指针
![laser_move](../images/dst/laser_move.gif)

<details> <summary>代码点我展开</summary>

预制物方面略过

```lua
local function MoveLaser(doer,invobj,act_pos_x,act_pos_z)
    -- @param doer: 玩家
    -- @param invobj: 手持武器,仅作为特效的载体
    -- @param act_pos_x,act_pos_z: 瞄准位置
    if invobj.movelaser == nil then 
        invobj.movelaser = SpawnPrefab('fx_laserbeem_blue_particle')
        local doer_x,doer_y,doer_z = doer.Transform:GetWorldPosition() -- 人物坐标
        local act_angle = CalcCoords:angleBetweenVectors(doer_x,doer_z,act_pos_x,act_pos_z) -- 人物到施法点的方向与默认方向的转角
        local doer_radian_angle = (act_angle-90) * math.pi / 180 -- 转成弧度
        local fx_len = 28 -- 特效实际长度(自行测试)
        local iter_range = 2 -- 扫描半径

        invobj.movelaser.Transform:SetPosition(doer_x,doer_y+1.5,doer_z)
        invobj.movelaser.Transform:SetRotation(act_angle)
        invobj.movelaser.Transform:SetScale(2,2,2)

        invobj.movelaser.task_period_update_recoil = invobj.movelaser:DoPeriodicTask(0.02,function(inst)
            local doer_x,doer_y,doer_z = doer.Transform:GetWorldPosition()
            invobj.movelaser.Transform:SetPosition(doer_x,doer_y+1.5,doer_z)

            local mouse_x,_,mouse_z = ConsoleWorldPosition():Get() -- 鼠标坐标
            local mouse_angle = CalcCoords:angleBetweenVectors(doer_x,doer_z,mouse_x,mouse_z) -- 人物到鼠标的方向与默认方向的转角
            invobj.movelaser.Transform:SetRotation(mouse_angle) -- 根据鼠标位置更新旋转角度
            local dist_btw_doer_and_mouse = CalcCoords:calcDist(doer_x,doer_z,mouse_x,mouse_z,true) -- 人物到鼠标的距离
            for dist=iter_range,fx_len,iter_range*2 do -- 找到线上的实体
                local iter_x,iter_z = CalcCoords:findPointOnLine(mouse_x,mouse_z,doer_x,doer_z,dist_btw_doer_and_mouse,dist) -- 根据距离找到线上的点
                local ents = TheSim:FindEntities(iter_x,1,iter_z,iter_range,{'locomotor'},{'INLIMBO','player','companion','wall'}) 
                for _,v in pairs(ents) do
                    if v and v.components and v.components.health and not v.components.health:IsDead() and v.components.combat then
                        local v_x,_,v_z = v.Transform:GetWorldPosition() -- 敌人坐标
                        local dist_btw_v_and_doer = CalcCoords:calcDist(doer_x,doer_z,v_x,v_z,true) -- 人物到敌人的距离
                        local des_x,des_z = CalcCoords:findPointOnLine(v_x,v_z,doer_x,doer_z,dist_btw_v_and_doer,dist_btw_v_and_doer+1) -- 击退
                        v.components.combat:GetAttacked(doer,20) -- 伤害
                    end
                end
            end
        end)

    else
        invobj.movelaser:Remove()invobj.movelaser = nil 
    end
end
```

</details>


## 彩虹变色

注意返回的值r,g,b还要/255!

<details> <summary>代码点我展开</summary>

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

</details>

## 锥形范围伤害(例如 散弹)

如果锥形的张角比较大,可以用;如果张角很小,那不如拆成一个个的圆(像糖葫芦一样)来遍历
![laser_move](../images/dst/shotgun.gif)

<details> <summary>代码点我展开</summary>

预制物方面略过

```lua
local function Shotgun(doer,invobj,act_pos_x,act_pos_z)
    -- @param doer: 玩家
    -- @param invobj: 此处为玩家的武器,本参数仅用做激光的载体
    -- @param act_pos_x,act_pos_z: 玩家瞄准的位置(锥形中轴线上的任意一个点)
    local angle = 60 -- 张角
    local ray = 5 -- 射线根数(奇数)
    local interval = 60/5 -- 射线间隔角度
    local fx_len = 6 -- 特效在世界中的实际长度(自行测试)
    local fx_lifetime = 0.8 -- 特效的持续时间/s
    local doer_x,doer_y,doer_z = doer.Transform:GetWorldPosition() -- 获取玩家位置
    local axis_rotation = Tools:angleBetweenVectors(doer_x,doer_z,act_pos_x,act_pos_z) -- 获取默认方向到锥形中轴线的转角
    local cone = Tools:coneExcircle(fx_len, doer_x, doer_z, act_pos_x, act_pos_z,angle) -- 获取锥形外接圆

    if invobj.fx_lasers ~= nil and type(invobj.fx_lasers) == 'table' then for _,v in pairs(invobj.fx_lasers) do if v and v:IsValid() then v:Remove() end end end -- 清除之前的特效
    invobj.fx_lasers = {} -- remove之后再初始化
    for i=1,ray do
        invobj.fx_lasers[i] = SpawnPrefab('fx_laserbeem_hl')
        invobj.fx_lasers[i].Transform:SetPosition(doer_x,doer_y+1,doer_z) -- 设置特效位置
        invobj.fx_lasers[i].Transform:SetRotation(axis_rotation-angle/2+((i-1)*interval)) -- 设置特效方向
        invobj.fx_lasers[i]:DoTaskInTime(fx_lifetime, function(inst) if inst and inst:IsValid() then inst:Remove() end end) -- 设置特效持续时间
    end

    local ents = TheSim:FindEntities(cone.x,1,cone.z,cone.r,{'locomotor'},{'INLIMBO','player','companion','wall'})
    for _,v in pairs(ents) do
        if v and v.components and v.components.combat and v.components.health and not v.components.health:IsDead() then 
            local enemy_x,_,enemy_z = v.Transform:GetWorldPosition() -- 获取敌人位置
            if T:isEnemyInCone(enemy_x, enemy_z, cone.x, cone.z, act_pos_x, act_pos_z, angle, fx_len) then 
                v.components.combat:GetAttacked(doer,20) -- 造成伤害
            end
        end
    end
end
```

</details>


