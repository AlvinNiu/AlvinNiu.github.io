---
layout:       post
title:        "翻译-ExcelDNA开发文档-首页"
subtitle:     "更容易理解ExcelDNA的开发"
date:         2018-10-03 09:00:00
author:       "Alvin"
header-img:   "img/excel-dna.jpg"
header-mask:  0.3
catalog:      true
tags:
    - Office 加载项
    - Office 外接程序
    - 翻译
    - 技术
    - Excel-DNA
---

## 前言

ExcelDNA是一名国际友人开发的开源框架，文档全是英文文档，当时看的时候非常吃力，现在将英文文档翻译过来，为的是让自己加深印象以及自己以后看的时候能不用这么吃力。


## 介绍

Excel-DNA是一个独立于Excel的.net项目。使用Excel-DNA你可以用C#、VB、F#创建一个本地的Excel插件，该插件可以执行用户自定义函数（UDF）自定义菜单栏等。整个插件可以打包进一个.xll文件，不需要安装或者注册

## 开始

如果的的Visual Studio 版本支持 NuGet Package Manager（包管理工具）你可以很容易使用Excel-DNA add-in

* 创建一个新的类库项目在VB、C#、F#中
* 使用Manage NuGet Packages窗口或者Package Manager控制台，安装Excel-DNA的包
    
    ```
    PM> Install-Package ExcelDna.AddIn
    ```
* 新建一个类，并添加如下代码

    ```
    //安装所需的包后，在相关类文件上也要引用次文件
    using ExcelDna.Integration;

    public static class MyFunctions
    {
        //此处便是定义该方法为excel自定义函数，函数的名称为SayHello
        [ExcelFunction(Description = "My first .NET function")]
        public static string SayHello(string name)
        {
            return "Hello " + name;
        }
    }

    ```
* 设置项目调试，启动Excel
    ![如图所示]({{ site.url }}/img/postin/exceldna-run-excel.png)

* 编译，加载你的Excel公式,并在Excel单元格中输入以下公式，便能看见输出的东西

    ```    
    =SayHello("World!")
    ```

使用ExcelDNA NuGet包安装必要的文件和配置，编译你的项目，便生成出ExcelDNA插件
或者，从(GitHub)[https://github.com/Excel-DNA/ExcelDna/releases]获取源码，然后通过开始页面，一步一步创建C# 插件。

## 更多信息

ExcelDNA 依赖于.NET 开发，并且用户需要安装免费提供的.NET Framework。项目代码会整合到Excel插件（.xll文件）中并安装到Excel上。代码可以写在文本脚本文件(.dna)中，也可以写在可编译的.NET 类库中（.dll）。Excel-DNA支持.NET Framework 2.0/3.0/3.5/4。插件致力于运行时版本，并且Excel支持同事加载多个版本的Excel插件

Excel版本从97-2016均可以使用ExcelDNA插件，一些高级功能在不同的版本支持情况不同，例如，多线程重新计算（2007版及之后的支持）、注册免费的RTD（异步自定义函数）（2002版及以后的支持）、自定义菜单栏接口（2007及2010版支持，其他均不支持）自定义任务窗格（2007版及之后的版本支持）、卸载UDF计算功能（2010版及之后的支持）、64位版本（2010版及之后的支持）


## 最新版本

最新版本是 ExcelDNA0.34，最新发布时间为2017.06，最新发布包括修复bug、优化运行效果、整合NuGet包


## 相关链接

[原文文档](https://excel-dna.net/)

此处作者回答了很多问题[作者的Stack Overflow](https://stackoverflow.com/users/44264/govert)

[Google Group](https://groups.google.com/forum/#!forum/exceldna)