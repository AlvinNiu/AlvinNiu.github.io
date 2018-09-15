---
layout:       post
title:        "Office加载项"
subtitle:     "让你的office与网络直接相连"
date:         2018-09-15 18:00:00
author:       "Alvin"
header-img:   "img/bathroom-escape.jpg"
header-mask:  0.3
catalog:      true
tags:
    - Office
    - 加载项
    - Office web
    - Office 外接程序
    - 技术
    - Excel
---

## 前言

>前一段时间公司做了有关Excel 加载项的开发，也遇到了很多坑，所以在此记录一下，有两个原因，1.留给以后在用到加载项的时候，复习所用，避免 跳进同一个坑里。2.留给其他做加载项的人一个参考

## 概述

>此处只大概说一下，如果想了解详细的，请到[office加载项官网](https://docs.microsoft.com/zh-cn/office/dev/add-ins/overview/office-add-ins)

>Office 开发有很多种选择
>* VBA 参考[浅谈Excel开发](http://www.cnblogs.com/yangecnu/p/Excel-Develpment-Introduction.html)
>* Shared Addin  参考[浅谈Excel开发](http://www.cnblogs.com/yangecnu/p/Excel-Develpment-Introduction.html)
>* VSTO 参考[VSTO官方文档](https://docs.microsoft.com/zh-cn/visualstudio/vsto/programming-vsto-add-ins?view=vs-2017)
>* Office web加载项

**加载项的组成部分**
* 清单文件，mainfast.xml 该文件包含了以下内容
    * 外接程序的显示名称、说明、ID、版本和默认区域设置。
    * web项目的地址
    * 加载项的菜单样式以及布局
    * 外接程序的权限级别和数据访问要求。
* web项目，任何web项目均可，但是一定要兼容ie11，因为office调用的是本地的ie11浏览器。

**功能**
* 通过mainfast.xml文件控制菜单，![如图所示]({{ site.url }}/img/postin/about-addins-addincommands.png)
* web项目是通过点击菜单展示在任务栏，展示效果![如图所示]({{ site.url }}/img/postin/about-addins-taskpane.png)
* web项目中的js文件可以控制Office，操作Office的文件，例如向Excel输入内容

## 开始做一个项目

**跟官网开发的步骤有些不同，但是两种方法均可**
>由于我之前开发的是Excel加载项，所以接下来的例子，均是Excel加载项开发的例子，其他的加载项应该同理。

第一、要新建Excel 外接程序，如图所示

项目建好之后，项目中的xml文件已经在里面了，此处指介绍一些重要的节点，如果感兴趣到此处去了解

第二、创建web项目，目前创建有四种web项目的结构。除了jQuery,其他均是目前流行的组件化开发
* Angluar
* Jquery
* React
* Vue

因为我使用的是Vue，所以此处只介绍Vue搭建站点，其他三种方式去[Office加载项官网](https://docs.microsoft.com/zh-cn/office/dev/add-ins/quickstarts/excel-quickstart-angular)

* 使用 vue-cli脚手架搭建项目[参考](https://cli.vuejs.org/zh/guide/installation.html)
* 部署web项目

**今天先写到着，太累了，没心情写了**