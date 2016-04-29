---
title: Python操作Excel
date: 2015-06-17 20:49:05
tags:
- Python
categories:
- Linux
---

`Python`操作`Excel`文件需要用到`xlrd`库，可以使用`pip`安装。

使用方法非常简单

1. 打开文件
```
f = open_workbook(filepath)
```

2. 获取表

一个excel可以有多个sheet

```
# 列出所有的sheet
sheets = f.sheets()
for sheet in sheets:
  print sheet.name

# 获取指定位置的sheet
sheet = f.sheet_by_index(0)
或
sheet = sheets[0]

# 获取指定名字的sheet
sheet = f.sheet_by_name(sheetname)
```

3. 表格操作

```
# 获取行数、列数
nrows = sheet.nrows
ncols = sheet.ncols

# 获取整行、整列的值(list)
row_values = sheet.row_values(nRow)
col_values = sheet.col_values(nCol)

# 获取单元格的值
cell_value = sheet.cell(nRow, nCol).value
```
