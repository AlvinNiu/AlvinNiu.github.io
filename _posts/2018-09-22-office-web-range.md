---
layout:       post
title:        "Office加载项对Excel进行读写操作"
subtitle:     "让你的office与网络直接相连"
date:         2018-09-22 10:00:00
author:       "Alvin"
header-img:   "img/office-web.jpg"
header-mask:  0.3
catalog:      true
tags:
    - Office 加载项
    - Excel Web AddIn
    - Office 外接程序
    - Range
    - 技术
    - Excel读写
---
## 前言

>在开发ExcelWeb插件的时候，一大亮点就是可以在web项目中操作Excel，读取Excel的内容，也可以将服务端的数据写入的 Excel中，大大方便的用户使用Excel，提高工作效率

## Ranges

Ranges表示Excel的区域，例如一个单元格的区域是=A1,多个单元格的区域，B1:B4表示连续的4个单元格，对Excel 内容的读写，即是对Ranges的读写。

### Ranges的属性

* address，表示Ranges的地址，例如：A1，B1:B4
* values,二维数组，表示区域的实际值
* texts,二维数组，表示区域的展示值
* formulas,二维数组，表示区域的公式，excel可以使用公式，使用之后，看到的值是公式计算之后的值，如果想要知道使用了那些公式，就需要使用formulas属性
* format，区域的字体格式，可以该表文字字体，颜色，也可以改变区域的背景色。
* numberFormat,二维数组，区域的格式，有文本、日期、数值等，和Excel右击设置单元格的格式里所有的格式一致

    * General     General
    * Number      0
    * Currency    $#,##0.00;[Red]$#,##0.00
    * Accounting  _($* #,##0.00_);_($* (#,##0.00);_($* "-"??_);_(@_)
    * Date        m/d/yy
    * Time        [$-F400]h:mm:ss am/pm
    * Percentage  0.00%
    * Fraction    # ?/?
    * Scientific  0.00E+00
    * Text        @
    * Special     ;;
    * Custom      #,##0_);[Red](#,##0)
    * 来源：[Stack Overflow上的回答](https://stackoverflow.com/questions/20648149/what-are-numberformat-options-in-excel-vba)

### Ranges的方法

* load(),表示接下来你要使用哪个属性，需要在此处表明，例如，你要使用address，则 range.load('address')。
* clear(),清空该区域
* delete(),删除该区域，并指定是否需要下面的值上移，或者右面的值左移，参数枚举值如下：
    * Excel.DeleteShiftDirection.up:删除后，下面的值，向上移动填充空值
    * Excel.DeleteShiftDirection.left:删除后，右面的值，向左移动填充空值
* select()，选中该区域
* copyFrom(),将指定区域的值复制给当前区域

### 具体用法

准备工作：
* 首先在vue中要安装excel-addin

```
npm install excel-addin --save
```
* 在index.html中假如office.js
```
  <script src="https://appsforoffice.microsoft.com/lib/1/hosted/office.js"></script>

```
* 如果想要在Excel初始化时做些什么，还需要在main.js中假如下面的代码
```
const Office = window.Office
Office.initialize = () => {
    //初始化内容
}
```
* 操作Excel的行为均要写在特定方法里，即：
```
//其中context表示Excel的上下文，操作Excel的东西均通过该参数
 Excel.run(function(context) {
     //此处写操作Excel的方法
     //处理完之后要提交，Excel才会识别，另外所有的//读写数据的操作均在，
     //context.sync().then(=>{
         //此处读写数据
     })
     return context.sync()
 }
```
#### 写入

```
    write: function() {
      Excel.run(function(context) {
        //获取指定名字的工作薄sheet
        // var sheet = context.workbook.worksheets.getItem('sheetName')
        //获取当前活动工作薄
        var sheet = context.workbook.worksheets.getActiveWorksheet()
        var data = [
          ['Product', 'Qty', 'Unit Price', 'Total Price'],
          ['Almonds', '2', '7.5', '15'],
          ['Coffee', '1', '34.5', '34.5'],
          ['Chocolate', '5', '9.56', '47.8'],
          ['', '', '', '97.3']
        ]
        //此二维数组的长度要和数据的保持一致，否则无效
        var formats = [
          ['@', '@', '@', '@'], //设置格式为文本
          ['0.00', '0.00', '0.00', '0.00'],
          ['0.00', '0.00', '0.00', '0.00'],
          ['0.00', '0.00', '0.00', '0.00'],
          ['0.00', '0.00', '0.00', '0.00']
        ]

        var range = sheet.getRange('A1:D5')
        //选中该区域
        range.select()
        // 设置背景色和字体
        range.format.fill.color = '#4472C4'
        range.format.font.color = 'white'
        //设置区域的格式
        range.numberFormat = formats

        //表示加载values属性，如果不加载在下面是不可以使用的
        range.load('values')

        return context.sync().then(function() {
          //写入方法必须在该方法内执行才有效
          range.values = data
        })
      }).catch(_this.errorHandler)
    },
```

#### 读取数据

```
read: function() {
      let _this = this
      Excel.run(function(context) {
        //获取指定名字的工作薄sheet
        // var sheet = context.workbook.worksheets.getItem('sheetName')
        // 获取当前选中的单元格
        var range = context.workbook.getSelectedRange()
        //获取当前选中的单元格
        //表示加载以下属性，如果不加载在下面是不可以使用的
        range.load('values')
        range.load('address')
        range.load('formulas')
        range.load('text')

        return context.sync().then(function() {
          //写入方法必须在该方法内执行才有效
          _this.content = {
            values: range.values,
            formulas: range.formulas,
            address: range.address,
            texts: range.text
          }
          console.log(_this.content)
        })
      }).catch(_this.errorHandler)
    }
```

#### 删除某一行数据

```
deleteOne: function() {
      let _this = this
      Excel.run(function(context) {
        //获取指定名字的工作薄sheet
        // var sheet = context.workbook.worksheets.getItem('sheetName')
        // 获取当前选中的单元格
        var sheet = context.workbook.worksheets.getActiveWorksheet()
        var range = sheet.getRange('A2:D2')
        range.delete(Excel.DeleteShiftDirection.up)
        //提交操作
        return context.sync()
      }).catch(_this.errorHandler)
    },
```

#### 清空所有数据

```
clear: function() {
      let _this = this
      Excel.run(function(context) {
        //获取指定名字的工作薄sheet
        // var sheet = context.workbook.worksheets.getItem('sheetName')
        // 获取当前选中的单元格
        var sheet = context.workbook.worksheets.getActiveWorksheet()
        var range = sheet.getRange('A1:D4')
        range.clear()
        //提交操作
        return context.sync()
      }).catch(_this.errorHandler)
    },
```

## 参考链接

[Excel JavaScrip API](https://docs.microsoft.com/en-us/office/dev/add-ins/excel/excel-add-ins-ranges#delete-a-range-of-cells)

## 项目地址
[项目链接](https://github.com/AlvinNiu/ExcelWebAddInDemo)

****