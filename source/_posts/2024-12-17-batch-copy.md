---
title: Batch批处理 Copy
date: 2024-12-17 23:50:37 +0800
tags: [batch]
---

复制并覆盖已有文件，不提示。

`copy /b/v/y dirname_a\dirname_b\filename.ext dirname_c\filename.ext`

**参考：**

* [How to overwrite existing files in batch?](https://stackoverflow.com/questions/4051294/how-to-overwrite-existing-files-in-batch)
* [SS64: Copy](https://ss64.com/nt/copy.html)

**其它：**

也可以使用 `xcopy` 命令来覆盖文件夹。

`xcopy /s /y "C:\backupMai" "\\myserver\backup$\logFile\c022456"`

* [Overwrite folder with Xcopy](https://superuser.com/questions/1643024/overwrite-folder-with-xcopy)
* [SS64: XCOPY](https://ss64.com/nt/xcopy.html)
