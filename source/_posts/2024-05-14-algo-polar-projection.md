---
title: 算法 - 极坐标位移
date: 2025-05-14 18:19:40 +0800
tags: [algo]
---

    0: →
    90: ↑
    180: ←
    270: ↓

```lua

---@param ox number 原点X
---@param oy number 原点Y
---@param deg number 偏移朝向
---@param dis number 偏移距离
local function calculateOffset(ox, oy, deg, dis)
    local radians = deg * math.pi / 180
    local fx = ox + math.cos(radians) * dis
    local fy = oy + math.sin(radians) * dis
    return fx, fy
end

---@param ox number 原点X
---@param oy number 原点Y
---@param deg number 偏移朝向
---@param dis number 偏移距离
local function calculateOffset(ox, oy, deg, dis)
    -- 将角度转换为弧度
    local radians = deg * math.pi / 180

    -- 计算偏移
    local fx = ox + math.cos(radians) * dis
    local fy = oy + math.sin(radians) * dis

    -- DEBUG
    -- ox = math.round(ox * 10000) / 10000
    -- oy = math.round(oy * 10000) / 10000
    -- fx = math.round(fx * 10000) / 10000
    -- fy = math.round(fy * 10000) / 10000
    -- print('deg', deg)
    -- print('ox, oy', ox, oy)
    -- print('fx, fy', fx, fy)

    return fx, fy
end

```
