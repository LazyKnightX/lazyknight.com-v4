---
title: UI 边界定位计算
date: 2025-10-6 17:37:53 +0800
tags: [UI编程]
---

预设条件：
* 画布大小：128x128
* 原点：左上角
* 坐标轴：X→，Y↓

基于此条件，在画布上放置一个16x16的矩形，其左上角坐标为(10,10)，margin为4，padding为2。

计算该矩形的实际位置：
* 左上角坐标 = (x + margin + padding, y + margin + padding)
* 右下角坐标 = (x + margin + padding + width + padding + margin, y + margin + padding + height)

定义：
* margin - 矩形边框外的间隔距离
* padding - 矩形内容与边框的距离
