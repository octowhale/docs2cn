# 简单用例Simple usage

## 编辑工作簿

```python
>>> from openpyxl import Workbook
>>> from openpyxl.compat import range     # import后的range 将覆盖标准库中的range函数
>>> from openpyxl.cell import get_column_letter
>>>
>>> wb = Workbook()     # 创建一个workbook
>>>
>>> dest_filename = 'empty_book.xlsx'       # 保存使用的位置和文件名
>>>
>>> ws1 = wb.active     # 获取工作表 (创建新工作簿时始终会有一个工作表)
>>> ws1.title = "range names"   # 为工作命名
>>>
>>> for row in range(1, 40):    # 向工作表写入40行
...     ws1.append(range(600))  # 向工作表写入600列
>>>
>>> ws2 = wb.create_sheet(title="Pi")   # 创建第二个工作表，并命名为Pi
>>>
>>> ws2['F5'] = 3.14        # 给单元格'F5'赋值
>>>
>>> ws3 = wb.create_sheet(title="Data") # 创建第三个工作表，并命名为Data
>>> for row in range(10, 20):
...     for col in range(27, 54):
...         _ = ws3.cell(column=col, row=row, value="%s" % get_column_letter(col)) # 将单元格的列名作为单元格的值
>>> print(ws3['AA10'].value)
AA
>>>
>>> ws4=wb.create_sheet('NEW_DATA')
>>> for row in range(1,10):
        for col in range(1,10):
            # 如果没有使用赋值语句 "_ = .... ",结果如下
            ws4.cell(column=col,row=row, value="%s" % get_column_letter(col))
            
<Cell NEW_DATA.A1>
<Cell NEW_DATA.B1>
<Cell NEW_DATA.C1>
<Cell NEW_DATA.D1>
<Cell NEW_DATA.E1>
...
>>> 
>>> wb.save(filename = dest_filename)       # 保存文件
```


### Write a workbook from *.xltx as *.xlsx
### 读取xltx模板并保存到xlsx文件

```python
>>> from openpyxl import load_workbook
>>>
>>>
>>> wb = load_workbook('sample_book.xltx') 
>>> ws = wb.active 
>>> ws['D2'] = 42 
>>>
>>> wb.save('sample_book.xlsx') 
>>>
>>> # or you can overwrite the current document template
>>> # wb.save('sample_book.xltx')
```

## Write a workbook from *.xltm as *.xlsm

```python
>>> from openpyxl import load_workbook
>>>
>>>
>>> wb = load_workbook('sample_book.xltm', keep_vba=True) 
>>> ws = wb.active 
>>> ws['D2'] = 42 
>>>
>>> wb.save('sample_book.xlsm') 
>>>
>>> # or you can overwrite the current document template
>>> # wb.save('sample_book.xltm')
```

### Read an existing workbook

```python
>>> from openpyxl import load_workbook
>>> wb = load_workbook(filename = 'empty_book.xlsx')
>>> sheet_ranges = wb['range names']
>>> print(sheet_ranges['D18'].value)
3
```

**Note**

There are several flags that can be used in load_workbook.

guess_types will enable or disable (default) type inference when reading cells.
data_only controls whether cells with formulae have either the formula (default) or the value stored the last time Excel read the sheet.
keep_vba controls whether any Visual Basic elements are preserved or not (default). If they are preserved they are still not editable.
Warning

openpyxl does currently not read all possible items in an Excel file so images and charts will be lost from existing files if they are opened and saved with the same name.

##Using number formats

```python
>>> import datetime
>>> from openpyxl import Workbook
>>> wb = Workbook(guess_types=True)
>>> ws = wb.active
>>> # set date using a Python datetime
>>> ws['A1'] = datetime.datetime(2010, 7, 21)
>>>
>>> ws['A1'].number_format
'yyyy-mm-dd h:mm:ss'
>>>
>>> # set percentage using a string followed by the percent sign
>>> ws['B1'] = '3.14%'
>>>
>>> ws['B1'].value
0.031400000000000004
>>>
>>> ws['B1'].number_format
'0%'
Using formulae
>>> from openpyxl import Workbook
>>> wb = Workbook()
>>> ws = wb.active
>>> # add a simple formula
>>> ws["A1"] = "=SUM(1, 1)"
>>> wb.save("formula.xlsx")
```

**Warning**

NB you must use the English name for a function and function arguments must be separated by commas and not other punctuation such as semi-colons.
openpyxl never evaluates formula but it is possible to check the name of a formula:

>>> from openpyxl.utils import FORMULAE
>>> "HEX2DEC" in FORMULAE
True
If you’re trying to use a formula that isn’t known this could be because you’re using a formula that was not included in the initial specification. Such formulae must be prefixed with xlfn. to work.

Merge / Unmerge cells
>>> from openpyxl.workbook import Workbook
>>>
>>> wb = Workbook()
>>> ws = wb.active
>>>
>>> ws.merge_cells('A1:B1')
>>> ws.unmerge_cells('A1:B1')
>>>
>>> # or
>>> ws.merge_cells(start_row=2,start_column=1,end_row=2,end_column=4)
>>> ws.unmerge_cells(start_row=2,start_column=1,end_row=2,end_column=4)
Inserting an image
>>> from openpyxl import Workbook
>>> from openpyxl.drawing.image import Image
>>>
>>> wb = Workbook()
>>> ws = wb.active
>>> ws['A1'] = 'You should see three logos below'
>>> # create an image
>>> img = Image('logo.png')
>>> # add to worksheet and anchor next to cells
>>> ws.add_image(img, 'A1')
>>> wb.save('logo.xlsx')
Fold columns (outline)
>>> import openpyxl
>>> wb = openpyxl.Workbook(True)
>>> ws = wb.create_sheet()
>>> ws.column_dimensions.group('A','D', hidden=True)
>>> wb.save('group.xlsx')