---
title: Defold引擎以及Lua、TypeScript2Lua
date: 2025-2-20 12:42:27 +0800
tags: [ts2lua]
---

## TypeScript2Lua

[TypeScript2Lua](https://github.com/TypeScriptToLua/TypeScriptToLua) 是一个将 TypeScript 转换为 Lua 的转换器。

游戏开发中，如果你能在编写代码的过程中尽早发现错误，就能尽可能多的避免将问题遗留到游戏运行中或是游戏上线后。

使用TypeScript作为代码的基础仓库对于长期和稳定的游戏开发而言是有很大意义的，因为TypeScript的强类型支持、编辑期间的代码错误检查等功能，能够综合的提升代码的可靠性，降低维护难度。

## Defold引擎

[ts-defold](https://ts-defold.dev/) [defold](https://defold.com/)

Defold 是一个开源的跨平台游戏引擎，它使用 Lua 作为其脚本语言。

在TypeScript2Lua的支持下，现在可以使用TypeScript为其编写游戏脚本。

## 参考资料

* [ts2lua](https://github.com/Halliwood/ts2lua) - 看上去是某个ts2lua的另一种实现，但这个仓库已经停止维护多年。
* [lua-objects](https://github.com/dmccuskey/lua-objects) - Lua中的一种OOP实现方案，10年前已停止维护。
