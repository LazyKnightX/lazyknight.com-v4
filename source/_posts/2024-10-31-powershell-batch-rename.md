---
title: Powershell 批量修改文件名
date: 2024-10-31 14:47:57 +0800
tags: [Powershell]
---

**功能：**

```powershell
# 把所有的 JPG 文件重新命名为 image_编号.jpg
Get-ChildItem *.jpg | ForEach-Object -Begin {
  $count = 1
} -Process {
  Rename-Item $_ -NewName "image_$count.jpg"
  $count++
}
```

**参考：**

* [在 Windows 中更改大量檔案名稱，批次修改檔名，快速又省力！](https://blog.gtwang.org/windows/how-to-batch-rename-files-in-windows/)
