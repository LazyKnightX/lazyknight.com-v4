---
title: AHK2 鼠标点击
date: 2024-04-09 14:51:46 +0800
tags: [ahk2]
---

按下 `Ctrl + 1` 以使用这个脚本。  
会在鼠标当前所在位置重复点击72次，点击间隔为2秒。

```ahk
#Requires AutoHotkey v2.0

^1::                ; Ctrl + 1
{
    Loop 72
    {
        MouseClick "Left", , , 1, 100
        Sleep 2000
    }
}

Esc::ExitApp  ; Exit script with Escape key
^!p::Pause    ; Pause script with Ctrl+Alt+P
^!s::Suspend  ; Suspend script with Ctrl+Alt+S
^!r::Reload   ; Reload script with Ctrl+Alt+R
```

文档:
* [MouseClick](https://www.autohotkey.com/docs/v2/lib/MouseClick.htm)
* [Keystrokes](https://www.autohotkey.com/docs/v2/howto/SendKeys.htm)
* [Loop](https://www.autohotkey.com/docs/v2/lib/Loop.htm)

参考:
* [How do I stop an active AutoHotkey script?](https://stackoverflow.com/questions/45700383/how-do-i-stop-an-active-autohotkey-script)