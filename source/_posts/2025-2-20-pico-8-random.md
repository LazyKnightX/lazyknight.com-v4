---
title: PICO-8 随机数
date: 2025-2-20 13:20:37 +0800
tags: [PICO-8]
---

https://pico-8.fandom.com/wiki/Rnd

PICO-8 使用 rnd() 函数来创建随机数。

    scale    = rnd(20) - 10     -- a random number between -10 and 10
    bomb.x   = 32 + rnd(64)     -- a random number between 32 and 96
    die_roll = flr(rnd(6)) + 1  -- a random integer between 1 and 6
    
    earnings = flr(rnd(50 * 100)) / 100  -- between 0.00 and 50.00
    
    print(rnd(20))       -- for example, 3.837
    print(flr(rnd(20)))  -- for example, 17
    
    -- note this is the same as rnd(0xffff)
    print(rnd(-1))       -- for example, -1734.56 or 13744.63
    
    print(rnd(split("hello",""))) -- h, e, o have 20% probability, l has 40% probability
    print(rnd({5,6,7,8,21,10,31,256})) -- returns a random value from the table between table[1] and table[#table]
