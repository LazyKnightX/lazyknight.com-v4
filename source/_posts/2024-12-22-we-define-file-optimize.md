---
title: 一种替换WE TriggerTypes TriggerParams 的方式
date: 2024-12-22 14:11:15 +0800
tags: [we]
---

一种替换WE的 TriggerTypes TriggerParams 的方式。

```
[TriggerTypes]
mkwefilterunitname=0,0,0,筛选单位目标类型,string

[TriggerParams]
MKWEFilterUnitNameNull=0,mkwefilterunitname,null,无
MKWEFilterUnitNameEnemy=0,mkwefilterunitname,`敌人`,敌人
```

```json
{
    "typename": "mkwefilterunitname",
    "label": "筛选单位目标类型",
    "type": "string",
    "enums": [
        { "key": "MKWEFilterUnitNameNull", "value": "null", "label": "无" },
        { "key": "MKWEFilterUnitNameEnemy", "value": "`敌人`", "label": "敌人" },
    ]
}
```

```
[[.args]]
--targetFilterName
type = string
default = "\"敌人\""

[[.args]]
--targetFilterName
type = mkwefilterunitname
default = "MKWEFilterUnitNameEnemy"
```
