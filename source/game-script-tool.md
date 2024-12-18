---
title: 游戏脚本工具
---

# 每拥有 `a` 点 `属性1`+`属性2` ， `属性3` +`b`

```lua
local a = 2
local b = 1

local hero
local _n = 0
local _m = 0
hero:loop(1000, function()
    local n1 = hero:get_attr("属性1")
    local n2 = hero:get_attr("属性2")
    local n = math.floor(n1 + n2)
    if _n ~= n then
        local m = n // a
        _n = n
        if _m ~= m then
            local d = m - _m
            _m = m
            hero:inc_attr("属性3", d * b)
        end
    end
end)
```

# 每拥有1**类** `物品?` ，`属性?` +`v`

```lua
local hero
local _n = 0
local v = 0.05
hero:loop(1000, function()
    local n1 = hero:get_owning_count("物品1") or 0
    local n2 = hero:get_owning_count("物品2") or 0
    local n3 = hero:get_owning_count("物品3") or 0
    local n4 = hero:get_owning_count("物品4") or 0
    local n5 = hero:get_owning_count("物品5") or 0
    local n6 = hero:get_owning_count("物品6") or 0
    local n = 0
    if n1 > 0 then n = n + 1 end
    if n2 > 0 then n = n + 1 end
    if n3 > 0 then n = n + 1 end
    if n4 > 0 then n = n + 1 end
    if n5 > 0 then n = n + 1 end
    if n6 > 0 then n = n + 1 end
    if _n ~= n then
        local d = n - _n
        _n = n
        hero:inc_attr("属性1", d * v)
        hero:inc_attr("属性2", d * v)
        hero:inc_attr("属性3", d * v)
    end
end)
```

# 每存在1个 `敌人` 单位， `属性?` +`v` ，上限20

```lua
local hero
local _n = 0
local _m = 20
local v = 0.01 --假设1点属性=0.01逻辑值
hero:loop(1000, function()
    local n = get_units_in_range(hero, 800):filter_unit_type(UNIT_TYPE_ENEMY):get_count()
    if n > _m then
        n = _m
    end
    if _n ~= n then
        local d = n - _n
        _n = n
        hero:inc_attr("属性1", d * v)
        hero:inc_attr("属性2", d * v)
        hero:inc_attr("属性3", d * v)
    end
end)
```
