---
layout:       post
title:        "VSCODE开发Python"
subtitle:     "安装、插件、调试"
date:         2018-10-07 09:00:00
author:       "Alvin"
header-img:   "img/vscode-python.jpg"
header-mask:  0.3
catalog:      true
tags:
    - vscode
    - python
    - IDE
    - 开发工具
---

## 前言

>为零基础的小白介绍如何使用vscode开发python项目。

>在进行配置vscode必须要先安装Python，[安装教程](http://www.runoob.com/python3/python3-install.html)

## 安装

* [下载地址](https://code.visualstudio.com/download)
![如图所示]({{ site.url }}/img/postin/down-vscode.png)
* 下载之后安装，一路默认，安装路径可以自定义

## 安装Python插件
安装插件的步骤：点击拓展->输入插件名称->点击安装->安装成功之后点击重新加载
>如图所示
![如图所示]({{ site.url }}/img/postin/vscode-python-plugin.png)

## 设置setting文件

* 打开setting.json文件, 文件->首选项->设置，出现如图所示：

![如图所示]({{ site.url }}/img/postin/vscode-open-settingjson.png)

* 将以下内容复制过去

```
 // Python specific
  "python.pythonPath": "D:/Python34/python.exe",
  "python.linting.pylintEnabled": false,
  "python.linting.flake8Enabled": true,
  "python.linting.flake8Args": ["--max-line-length=248"],
  "python.formatting.provider": "autopep8",
```
结果如图所示：
![如图所示]({{ site.url }}/img/postin/vscode-settingjson-content.png)

* 如果右下角出现让你安装某些插件，点击是，因为flake8没有安装但是配置中有，所以vscode会提示安装。
* 解释一下上述配置
    * python.pythonPath:python 的安装路径
    * python.linting.pylintEnabled：是否启动pylint插件，代码检测插件，此处设为false，是因为flake8设置为了ture。
    * python.linting.flake8Enabled：设置是否启用flake8插件，代码检测插件，此处设为true。相对于Pylint来说，Flake8检查规则灵活，支持集成额外插件，扩展性强
    * python.formatting.provider：提供自动格式化代码的插件为autopep8

## 开始编写python文件

* 打开一个文件夹，此处最好是一个空文件夹。快捷键：`Ctrl+K+O`。
* 创建一个`main.py`文件，点击图中的位置
![如图所示]({{ site.url }}/img/postin/vscode-python-createfile.png)
* 输入下面的代码
```
print("Hello,World!")
```
此时的python文件已经可以运行了

## 调试项目

有两种方式，
* 第一种：在main.py文件中，`右键`->在终端中运行python文件，如图所示：
![如图所示]({{ site.url }}/img/postin/vscode-debug-python1.png)
* 第二种
    * 第一步，在debug中，点击`添加配置`，如图所示：
    ![如图所示]({{ site.url }}/img/postin/vscode-debug-addsetting.png)
    * 第二步，在点击添加配置之后，配置文件会自动天剑python的调试配置。此时,切换到main.py的窗口，按`F5`即可运行

`注意：第一种方式不可以断点调试python项目`

## 小结

>这里只写了vscode开发python的基础中的基础，你可以使用这种方式，开发单个python文件然后调试，等把基础学完了，在学习如何调试整个python项目

## 参考链接

* [vscode官方文档](https://code.visualstudio.com/docs/python/python-tutorial)    