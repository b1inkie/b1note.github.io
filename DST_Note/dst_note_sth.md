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
    -- @param: angle 角度(角度制)
    -- @param: radius 半径
    angle = angle*direction
    local des_x = x + math.cos(math.rad(angle))*radius
    local des_z = z + math.sin(math.rad(angle))*radius
    
    return des_x, des_z
end
```