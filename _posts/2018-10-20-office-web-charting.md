---
layout:       post
title:        "Office加载项对Excel制图"
subtitle:     "让你的office与网络直接相连"
date:         2018-10-20 10:00:00
author:       "Alvin"
header-img:   "img/office-web.jpg"
header-mask:  0.3
catalog:      true
tags:
    - Office 加载项
    - Excel Web AddIn
    - Office 外接程序
    - Charting
    - 技术
    - 制图
---

## 前言

>使用js在Excel上画出图表，是不是很酷炫，试想一下，在左边工作栏选择一些条件，通过条件制图，并且保存在Excel中，将Excel分享给所有人。这是不是为你的工作提高了不少效率，而且还吸引眼球呢

>本篇还是先说怎么实现，在解释代码的每个类的属性与方法，并且介绍在使用时需要注意的地方（我踩过的坑，别人就不要再掉进去了），说说原理

## 普及图表常识
`如图所示：`
![如图所示]({{ site.url }}/img/postin/excel-web-charting.png)

* 选中画图的区域，点击插入图表，出现默认图
* 其中第一列除了第一行以外，其他的均表示x轴的数据
* 其中第一行除了第一列以外，其他数据均表示系列的数据
* 最多只有两个Y轴

## 开始画图

流程如下：
* 向Excel中插入数据
* 根据插入数据的区域，选中生成图表
* 默认的图表生成后，在根据特殊需求，遍历图表中的系列集合（如果没有特殊需求，则下面的步骤省略）
* 修改每个系列的格式，比如属于主轴还是次轴

### 插入数据
代码如下：
```
Excel.run(function(context) {
        //获取当前活动工作薄
        var sheet = context.workbook.worksheets.getActiveWorksheet()
        var data = [
          ['', '水果', '蔬菜', '房子'],
          ['2015', '4', '3.2', '10000'],
          ['2016', '4.5', '4.1', '15000'],
          ['2017', '4.8', '4.2', '20000'],
          ['2018', '5.2', '4.9', '30000']
        ]
        var range = sheet.getRange('A1:D5')
        //表示加载values属性，如果不加载在下面是不可以使用的
        range.load('values')
        return context.sync().then(function() {
          //写入方法必须在该方法内执行才有效
          range.values = data
        })
      }).catch(_this.errorHandler)
```

### 插入图表代码

```
Excel.run(function(context) {
        var sheet = context.workbook.worksheets.getActiveWorksheet()
        var dataRange = sheet.getRange('A1:D5')
        var chart = sheet.charts.add('Line', dataRange, 'auto')
        chart.title.text = 'Sales Data'
        chart.legend.position = 'right'
        chart.legend.format.fill.setSolidColor('white')
        chart.dataLabels.format.font.size = 15
        chart.dataLabels.format.font.color = 'black'

        return context.sync()
      }).catch(errorHandler)
```
结果如图所示：
![如图所示]({{ site.url }}/img/postin/insert-chart-one.png)
你会发现，只有一个房子的价格趋势，并没有其他两种的，仔细看图，会发现其他两种系列均在图的最底部，（原因是这两个系列的数值太小）

解决办法就是为房子单独设置一个坐标轴，并且设置坐标轴的单位。代码如下：
```
Excel.run(function(context) {
        var sheet = context.workbook.worksheets.getActiveWorksheet()
        var dataRange = sheet.getRange('A1:D5')
        var chart = sheet.charts.add('Line', dataRange, 'auto')
        chart.title.text = 'Sales Data'
        chart.legend.position = 'right'
        chart.legend.format.fill.setSolidColor('white')
        chart.dataLabels.format.font.size = 15
        chart.dataLabels.format.font.color = 'black'
        let serie = chart.series.getItemAt(2) //获取第三个系列
        serie.load('axisGroup') //由于我要设置这个属性，所以此处我要加载这个属性
        chart.load('axes')
        return context.sync().then(function() {
          serie.axisGroup = 'Secondary'
          chart.axes.getItem('Value', 'Secondary').displayUnit = 'Hundreds' //设置单位为百，此处getItem的第一个参数不知道为什么一定是Value，试了其他的都不行
        })
      }).catch(errorHandler)
```
结果如图所示：
![如图所示]({{ site.url }}/img/postin/insert-chart-two.png)

设置其中一个系列为柱形图,代码如下：
```
Excel.run(function(context) {
        var sheet = context.workbook.worksheets.getActiveWorksheet()
        var dataRange = sheet.getRange('A1:D5')
        var chart = sheet.charts.add('Line', dataRange, 'auto')
        chart.title.text = 'Sales Data'
        chart.legend.position = 'right'
        chart.legend.format.fill.setSolidColor('white')
        chart.dataLabels.format.font.size = 15
        chart.dataLabels.format.font.color = 'black'
        let serie = chart.series.getItemAt(2) //获取第三个系列
        serie.load('axisGroup') //由于我要设置这个属性，所以此处我要加载这个属性
        chart.load('axes')
        return context.sync().then(function() {
          serie.axisGroup = 'Secondary'
          serie.chartType = 'ColumnClustered'
          chart.axes.getItem('Value', 'Secondary').displayUnit = 'Hundreds' //设置单位为百
        })
      }).catch(errorHandler)
```
结果如图所示：
![如图所示]({{ site.url }}/img/postin/insert-chart-three.png)

到此时基本的图表功能都已经完成了，如果还想实现更复杂的功能，那就自己去看文档，[文档地址](https://docs.microsoft.com/zh-cn/javascript/api/excel/excel.chart?view=office-js#charttype)

## 常用的类、属性、方法

### 与图相关的-Chart
* axes：图表的坐标轴
    * categoryAxis：表示图表的类别轴（说实话，每名吧什么是类别轴）
    * seriesAxis：表示三维图表的系列轴（暂时没用到）
    * valueAxis：表示坐标轴的数值轴，我理解就是Y轴
    * getItem(type, group)：获取由类型与组标识的特定轴
        * type:类型为ChartAxisType,枚举值分别为：category、invalid、series、value
            * category：如果坐标轴显示的是类别，则为category
            * invalid：
            * series：表示坐标轴显示的是系列
            * Value：表示坐标轴显示的是数值
* chartType：表示图表的整体样式，是柱形图还是线形图,枚举值如下
```
| "Invalid" | "ColumnClustered" | "ColumnStacked" | "ColumnStacked100" | "3DColumnClustered" | "3DColumnStacked" | "3DColumnStacked100" | "BarClustered" | "BarStacked" | "BarStacked100" | "3DBarClustered" | "3DBarStacked" | "3DBarStacked100" | "LineStacked" | "LineStacked100" | "LineMarkers" | "LineMarkersStacked" | "LineMarkersStacked100" | "PieOfPie" | "PieExploded" | "3DPieExploded" | "BarOfPie" | "XYScatterSmooth" | "XYScatterSmoothNoMarkers" | "XYScatterLines" | "XYScatterLinesNoMarkers" | "AreaStacked" | "AreaStacked100" | "3DAreaStacked" | "3DAreaStacked100" | "DoughnutExploded" | "RadarMarkers" | "RadarFilled" | "Surface" | "SurfaceWireframe" | "SurfaceTopView" | "SurfaceTopViewWireframe" | "Bubble" | "Bubble3DEffect" | "StockHLC" | "StockOHLC" | "StockVHLC" | "StockVOHLC" | "CylinderColClustered" | "CylinderColStacked" | "CylinderColStacked100" | "CylinderBarClustered" | "CylinderBarStacked" | "CylinderBarStacked100" | "CylinderCol" | "ConeColClustered" | "ConeColStacked" | "ConeColStacked100" | "ConeBarClustered" | "ConeBarStacked" | "ConeBarStacked100" | "ConeCol" | "PyramidColClustered" | "PyramidColStacked" | "PyramidColStacked100" | "PyramidBarClustered" | "PyramidBarStacked" | "PyramidBarStacked100" | "PyramidCol" | "3DColumn" | "Line" | "3DLine" | "3DPie" | "Pie" | "XYScatter" | "3DArea" | "Area" | "Doughnut" | "Radar" | "Histogram" | "Boxwhisker" | "Pareto" | "RegionMap" | "Treemap" | "Waterfall" | "Sunburst" | "Funnel";
```
* dataLabels：表示图表上的数据标签
* displayBlanksAs：返回或设置在图表上绘制空白单元格方式，枚举值如下：
```
| "NotPlotted" | "Zero" | "Interplotted"
```
与Excel中此处的功能是一样的，如图所示：
![如图所示]({{ site.url }}/img/postin/excel-web-chart-null.png)
* height :表示图表的高度，以磅值表示。
* id:表示图表的唯一ID，只读
* legend：设置图例的位置，如果不设置，默认不显示图例。
    * position：代表图表上图例的位置，枚举值如下：
    ```
    | "Invalid" | "Top" | "Bottom" | "Left" | "Right" | "Corner" | "Custom"
    ```
    * showShadow:表示图例是否有阴影
* name：表示图表的名称
* series：表示图表的系列集合，
    * items:存储了该图表的所有系列
    * add(name, index)，向图表中添加一个系列
    * getItemAt(index)：通过索引获取图表中的某一个系列

### 与系列相关-ChartSeries

* axisGroup：返回或设置指定的数据系列的组，枚举值如下：
```
| "Primary" | "Secondary"
```
* chartType
* dataLabels:代表系列中的所有数据标签的集合
* delete() :将该系列删除

### 与坐标轴相关-ChartAxes
表示图表中的多个坐标轴。
* categoryAxis：表示图表的类别轴（说实话，每名吧什么是类别轴）
* seriesAxis：表示三维图表的系列轴（暂时没用到）
* valueAxis：表示坐标轴的数值轴，我理解就是Y轴
* getItem(type, group)：获取由类型与组标识的特定轴

### 与坐标轴相关-ChartAxis
表示图表中的单个坐标轴
* alignment：代表指定的坐标轴刻度线标签的对齐方式，枚举值如下：
```
  "Center" | "Left" | "Right"
```
* axisGroup
* baseTimeUnit:返回或设置指定分类轴的基本单位,枚举值如下：
```
 "Days" | "Months" | "Years"
```
* categoryType:返回或设置分类轴类型,枚举值如下：
```
 "Automatic" | "TextAxis" | "DateAxis"
```
* displayUnit :代表轴显示单位,枚举值如下：
```
"None" | "Hundreds" | "Thousands" | "TenThousands" | "HundredThousands" | "Millions" | "TenMillions" | "HundredMillions" | "Billions" | "Trillions" | "Custom"
```

`注意事项：`

>所有要在context.sync()后调用的属性，必须在load方法里标明，不然这里面是访问不到该属性的。

## 原理
使用js画图的步骤和在Excel上画图的步骤是一样的，都是先画出一个图来，然后再针对每个系列每个坐标轴去处理样式。
在js画图时，只要不 调用这句 `return context.sync()`，图就不会出现在Excel上，所有的系列修饰以及次坐标轴的修饰，都在图表画完之后，调用的then方法中处理的。比如：`return context.sync().then(function(){//此处写修饰系列与坐标轴的代码})`。掌握这个规律，在开发excel 制图时，一切问题都迎刃而解了。

## 参考链接

* [Work with Charts using the Excel JavaScript API](https://docs.microsoft.com/en-us/office/dev/add-ins/excel/excel-add-ins-charts)

* [JavaScript API for Office ](https://docs.microsoft.com/zh-cn/javascript/api/excel/excel.chart?view=office-js)

## 项目地址
[项目链接](https://github.com/AlvinNiu/ExcelWebAddInDemo)

****