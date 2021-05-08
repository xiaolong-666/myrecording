# python操作excel
使用 xlrd&xlwt&xlutils 三个库
- xlrd：用于读取 Excel 文件；
- xlwt：用于写入 Excel 文件；
- xlutils：用于操作 Excel 文件的实用工具，比如复制、分割、筛选等；

openpyxl: 可以直接修改excel

## 安装库
直接使用pip3安装即可
```bash
sudo pip3 install xlrd xlwt xlutils                                     
Collecting xlrd
  Downloading https://files.pythonhosted.org/packages/a6/0c/c2a72d51fe56e08a08acc85d13013558a2d793028ae7385448a6ccdfae64/xlrd-2.0.1-py2.py3-none-any.whl (96kB)
    100% |████████████████████████████████| 102kB 20kB/s 
Collecting xlwt
  Downloading https://files.pythonhosted.org/packages/44/48/def306413b25c3d01753603b1a222a011b8621aed27cd7f89cbc27e6b0f4/xlwt-1.3.0-py2.py3-none-any.whl (99kB)
    100% |████████████████████████████████| 102kB 14kB/s 
Collecting xlutils
  Downloading https://files.pythonhosted.org/packages/c7/55/e22ac73dbb316cabb5db28bef6c87044a95914f713a6e81b593f8a0d2f79/xlutils-2.0.0-py2.py3-none-any.whl (55kB)
    100% |████████████████████████████████| 61kB 8.4kB/s 
Installing collected packages: xlrd, xlwt, xlutils
Successfully installed xlrd-2.0.1 xlutils-2.0.0 xlwt-1.3.0

```
## 读取Excel文件
```python
import xlrd
data = xlrd.open_workbook(filename) # 文件名以及路径，如果有中文加上 r
```
### 常用的函数
#### 获取工作表
```python
table = data.sheets()[0]					# 通过索引顺序获取
table = data.sheet_by_index(sheet_indx) 	# 通过索引顺序获取
table = data.sheet_by_name(sheet_name)		# 通过名称获取

names = data.sheet_names()					# 返回book中所有工作表的名字
```
#### 行的操作
```python
nrows = table.nrows							# 获取该sheet中的行数
table.row(rowx)								# 返回由该行中所有单元格对象组成的列表
table.row_values(rowx, start_colx=0, end_colx=None)	# 返回由该行中所有单元格的数据组成的列表
table.row_len(rowx)							# 返回该行的有效单元格长度
```
#### 列的操作
```python
ncols = table.ncols							# 获取列表的有效列数
table.col(colx)								# 返回由该列中所有的单元格对象组成的列表
table.col_values(colx, star_rowx=0, end_rowx=None)	# 返回由该列中所有单元格的数据组成的列表
```
#### 单元格的操作
```python
table.cell(rowx, colx)						# 返回单元格对象
table.cell_type(rowx, colx)					# 返回对应位置单元格中的数据类型
table.cell_value(rowx, colx)				# 返回对应位置单元格的数据
```



## 创建表格
```python
book = xlwt.Workbook(encoding='utf-8', style_compression=0)		# 创建新的Excel
sheet = book.add_sheet('豆瓣电影Top250', cell_overwrite_ok=True)	# 创建新的sheet表
sheet.write(0, 0, '名称')				# 往表格写入内容
sheet.write(0, 1, '图片')
sheet.write(0, 2, '排名')
sheet.write(0, 3, '评分')
sheet.write(0, 4, '作者')
sheet.write(0, 5, '简介')
book.save("test.xlsx")					# 保存
```

### 设置字体样式
```python
workbook = xlwt.Workbook(encoding='utf-8')
workshellt = workbook.add_sheet('my test sheet')

# 初始化样式
style = xlwt.XFStyle()

# 为样式创建字体
font = xlwt.Font()
font.name = 'Time New Roman'	# 字体
font.bold = True				# 加粗
font.underline = True			# 下划线
font.italic = True				# 斜体

# 设置样式
style.font = font

# 往表格写入内容
worksheet.write(0,0,"测试内容1")
worksheet.write(2,1,"测试内容2",style)	# 应用样式

# 保存
workbook.save("测试.xlsx")

```

### 设置列宽、行高
```python
worksheet.col(0).width = 256*20			# 设置第一列的列宽

# 设置行高
style = xlwt.easyxf('font:height 360;')	# 18pt, 类型小初的字号
row = worksheet.row(0)
row.set_style(style)

```
### 合并列和行
```python
# 合并 第1行到第2行 的 第0列到第3列
worksheet.write_merge(1,2,0,3,'merge test')
```
### 为单元格设置背景色
```python
worksheet.write(1,1,"内容1")

# 创建样式
pattern = xlwt.Pattern()
pattern.pattern = xlwt.Pattern.SOLID_PATTERN
pattern.pattern_fore_colour = 5
style = xlwt.XFStyle()
style.pattern = pattern

# 使用样式
wooksheet.write(2,2,"内容2",style)
```
### 单元格对齐
```python
# 设置样式
style = xlwt.XFStyle()
al = xlwt.Alignment()
# VERT_TOP = 0x00       上端对齐
# VERT_CENTER = 0x01    居中对齐（垂直方向上）
# VERT_BOTTOM = 0x02    低端对齐
# HORZ_LEFT = 0x01      左端对齐
# HORZ_CENTER = 0x02    居中对齐（水平方向上）
# HORZ_RIGHT = 0x03     右端对齐
al.horz = 0x02		# 设置水平居中
al.vert = 0x01		# 设置垂直居中
style.alignment = al

# 应用
worksheet.write(2,2,"内容2",style)
```