---
title: 魔兽3地形替换说明
date: 2025-2-21 16:18:55 +0800
tags: [war3]
---

本文假设用户使用 w3x2lni 的情况进行开发。



一般需要替换下方5个文件：
map\war3map.doo
map\war3map.shd
map\war3map.w3e
map\war3map.wpm
map\war3mapUnits.doo

如果需要更新物编需替换：
map\doodad.ini



文件说明：
war3map.doo 装饰物
war3map.shd 阴影
war3map.w3e 大地形
war3map.wpm 地面阻断 (pathing)
war3mapUnits.doo 单位

二进制文件：
.w3u 物编单位数据
.w3d 物编装饰物数据
.w3b 物编可破坏物数据

其它文件：
.w3i 地图预设信息文件
.imp 地图导入文件



ref:
https://www.npmjs.com/package/wc3maptranslator
