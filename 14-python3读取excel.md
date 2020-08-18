<!--markdown-->```python
import xlrd
wb = xlrd.open_workbook('test.xlsx')
#查看包含的工作表
sheetnames = wb.sheet_names()
#获取一个表的四种方法
sheet = wb.sheet_by_name('Sheet1')
sheet = wb.sheets()[0]
sheet = wb.sheet_by_index(0)
sheet = wb['Sheet1']
#行数和列数
nrows = sheet.nrows
ncols = sheet.ncols
#枚举所有行/列
sheet.iter_rows(min_row=2)
sheet.iter_columns()

#单元格0行1列的值
sheet.cell(0,1).value
#第一列/行的值?
sheet.col_values(0)
sheet.row_values(0)
#cell的位置
列(column)以1开始，行(row)以0开始
cell = ['B5']
cell.row // 5
cell.column //B
cell.col_idx //2
#写入
sheet.cell(column=1, row=0, value='a')
```	