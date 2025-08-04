---
title: Windows 11 重设控制台位置
date: 2025-8-4 19:23:27 +0800
tags: [windows]
---

在 `计算机\HKEY_USERS` 下找到与自己账户相关的 `Console` ，点击它。

![](img/miscs/8.png)

找到 `WindowPosition` ，删除。

![](img/miscs/9.png)

重新打开Console，位置就会重置。

适用于解决多屏幕转换单屏幕情况下，Console位置被设置到了第二屏幕后无法拽回的问题。
