---
title: 游戏中的事件机制设计：在事件中改变运行参数
date: 2023-09-01 00:08:02 +0800
tags: [game-prog]
---

# 英雄发现宝箱

一般来说，像“英雄发现了一个宝物”、“英雄打开了一个宝箱”，这样的事件是常见的。发送这样的事件是单向的，我们只需要告知所有事件监听者，这个事件发生了就可以。

但如果我们的游戏中存在“当英雄发现一个宝物时，有20%概率额外发现一个宝物”这样的效果时，整个“英雄发现宝物”流程就会被重复调用，同时要为这样的效果设计逻辑流程，就必须考虑到可能会出现的某个数值的变更。

在这种情况下，我们就需要提供一个可以编辑的对象，允许脚本编写者在事件中改变某个数值。

以本段我们描述的场景为例：

```lua

--发送事件
local hero --某个英雄
local treasure_chest_id = 1 --假设发现的宝箱ID为1
local treasure_params = {} --创建一个对象，用于在事件中修改
treasure_params["宝箱数量"] = 1 --在默认情况下，我们只进行一次奖励
event_sender.send("英雄发现宝箱", hero, treasure_chest_id, treasure_params)

--在发送了事件后，获取动态的宝箱数量以进行奖励
if treasure_params["宝箱数量"] > 0 then
    for i = 1, treasure_params["宝箱数量"] do
        do_action("获得宝箱奖励", treasure_chest_id)
    end
end

--接收事件
event_listener.listen("英雄发现宝箱", function(hero, treasure_chest_id, treasure_params)
    --假设我们的英雄拥有技能效果可以在发现宝箱时有10%概率额外获得双倍奖励
    if hero:has_skill("双倍财富") and check_success(0.10) then
        treasure_params["宝箱数量"] = treasure_params["宝箱数量"] + 1
    end
end)

```
