---
layout:       post
title:        "Office加载项安装"
subtitle:     "让你的office与网络直接相连"
date:         2018-09-16 12:00:00
author:       "Alvin"
header-img:   "img/office-web.jpg"
header-mask:  0.3
catalog:      true
tags:
    - Office 加载项
    - Excel Web AddIn
    - Office web
    - Office 外接程序
    - 技术
    - 部署
---
## 前言

>Excel加载项离不开安装，Excel加载项本身安装及其简单，但这是在申请下来Office开发者账户之后，再次之前都得自行安装

## 线上安装

微软申请开发者账户有两种形式，一种是个人（119元），一种是公司（600元），除了付费以外还需要等待很长的一段时间，这个是最受不了的，如果你现在申请下来了，你就可以将你的xml文件上传到Office商店，这样别人直接从商店使用加载项即可。

## 线下安装

>如果你没有开发者账户，又想让别人使用你的加载项该怎么办呢？

### 共享目录

进入xml文件所在的目录->点击共享->点击特定用户->选择共享给那个用户->共享->点击复制将目录复制。
![如图所示]({{ site.url }}/img/postin/share-dirctory.png)
![如图所示]({{ site.url }}/img/postin/copy-share-dir.png)

### 将共享目录添加信任列表

进入Excel->点击文件->点击选项->信任中心->信任中心设置->受信任目录->粘贴刚刚复制的东西，然后将括号外面的删掉->添加目录->添加至菜单->确定->重启Excel

![如图所示]({{ site.url }}/img/postin/trust-dir-inexcel.png)

### 在加载项出点击共享文件夹

打开Excel->插入->我的加载项
![如图所示]({{ site.url }}/img/postin/excel-add-menu.png)

### 结果

目前为止，你的加载项就出现在你的ExcelTab栏里了

![如图所示]({{ site.url }}/img/postin/install-addin-result.png)

