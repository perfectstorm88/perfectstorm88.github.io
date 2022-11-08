
@[TOC]

# 基本功能验证：
- 更新值
- 更新样式为红色字体

参考： [Change value in Excel using Python](https://www.geeksforgeeks.org/change-value-in-excel-using-python/)

## 例子1
```json
# 基于
a = '/Users/lcz/Downloads/d/六中心/人工测试_A.xlsx'
import xlwt
import xlrd
from xlutils.copy import copy
from openpyxl.utils.cell import coordinate_from_string, column_index_from_string 

# load the excel file
rb = xlrd.open_workbook(a)
 
# copy the contents of excel file
wb = copy(rb)
 
# open the first sheet
w_sheet = wb.get_sheet(0)
# https://stackoverflow.com/questions/15649482/how-to-set-color-of-text-using-xlwt
style = xlwt.easyxf('font: color red;') # 红色字体
# row number = 0 , column number = 1
w_sheet.write(0,1,'Modified !',style=style) # 写入
# 也可以按指定的列的“字母编码”写入
w_sheet.write(0,column_index_from_string('D'),'Modified!D',style=xlwt.easyxf('pattern: pattern solid, fore_colour red;')) 
w_sheet.write(0,2,None)  # 清空
# save the file
wb.save(a.replace('.xlsx','_xlrd.xlsx'))
```

## 例子2：

```json
a = '/Users/lcz/Downloads/d/六中心/人工测试_A.xlsx'
from openpyxl import load_workbook
#load excel file
workbook = load_workbook(filename=a)
from openpyxl.styles import colors
from openpyxl.styles import Font, Color,PatternFill
#open workbook
ws = workbook['Padua评估详情_内科']
 
#modify the desired cell
a1 = ws['A1']
d4 = ws['D4']
# 修改字体颜色
ft = Font(color="FF0000")
a1.font = ft
d4.font = ft

# 修改前底色颜色
red_fill =PatternFill("solid", fgColor="FF0000")
ws['B1'].fill=red_fill
ws['C3'].fill=red_fill

# 改空值
ws['B2'].value=None

#save the file
workbook.save(a.replace('.xlsx','_openpyxl.xlsx'))
```

openpyxl的取值方式：
```
sheet1[1][0] == sheet1['A1']
column_index_from_string('A')== 1
```

cell的两种区分，列的索引上有个“坑”：
```bash
ce1 = worksheet.cell(row=i,column=c_time) # 这种取法，行从1开始，列从1开始
row=worksheet[i]
ce2=row[c_time-1] # 这种取法，行从1开始，列从0开始
```