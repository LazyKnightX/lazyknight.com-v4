---
title: xlwings
date: 2025-2-2 19:52:03 +0800
tags: [automation]
---

自动化处理Excel，可用于自动化测试、数据处理、报表生成等。

也可用于实现基于Excel的游戏数值验证Demo。

pip install xlwings
xlwings addin install

```python
import xlwings as xw

app = xw.apps.active

wb = xw.books.active  # in active app
# wb = app.books.active  # in specific app

sheet1 = xw.sheets.active  # in active book
# sheet = wb.sheets.active  # in specific book

# sheet1.range("A1")
# sheet1.range("A1:C3")
# sheet1.range((1,1))
# sheet1.range((1,1), (3,3))
# sheet1.range("NamedRange")

# Or using index/slice notation
# sheet1["A1"]
# sheet1["A1:C3"]
# sheet1[0, 0]
# sheet1[0:4, 0:4]
# sheet1["NamedRange"]

sheet1['A1'].value = 'Hello xlwings!'

for i in range(1, 10):
    sheet1[i, 0].value = i
    sheet1[i, 1].value = i * i
    sheet1[i, 2].value = i * i * i

```