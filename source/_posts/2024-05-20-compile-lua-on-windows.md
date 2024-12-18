---
title: 在Windows下编译Lua源代码
date: 2024-5-20 02:35:33 +0800
update: 2024-10-19 12:09:03 +0800
tags: [lua]
---

## 源码编译

下载Lua源码，复制下面仓库中的Compile.bat，放到源代码目录下运行。

https://github.com/Pharap/CompilingLua

形成目录结构：

```
/lua-x.x.x
    /src
    /doc
    /Compile.bat
```

注意下方路径需要添加到PATH中。

`C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin`

可能需要安装MSVC v140。

![](img/miscs/7.png)

## 其它：Lua for Windows

如果不需要非得编译源码，只希望使用Lua并且不特别在意版本，可以使用 Lua for Windows (Lua 5.1) 。

https://github.com/rjpcomputing/luaforwindows/releases
