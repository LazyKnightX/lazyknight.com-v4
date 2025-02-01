---
title: Lni Util
date: 2025-2-1 16:00:00 +0800
tags: [nodejs]
---

Lni Util 是我2020年写的一个小工具，用来实现 lni <-> javascript object 的转换。

通过这个工具，用户可以直接读取 lni 中的数据，并将其作为 javascript object 进行使用。

https://github.com/LazyKnightX/lni-util

安装方式：

`npm install lni-util`

实现方式：

字符串替换 & 正则表达式

在考虑要不要使用lpeg这类正规的语法解析方式来实现更准确的解析。
