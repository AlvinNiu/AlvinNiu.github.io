---
layout:       post
title:        "Office加载项"
subtitle:     "让你的office与网络直接相连"
date:         2018-09-15 18:00:00
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

>第一、新建Excel 外接程序，使用的开发工具是vs2017。
![如图所示]({{ site.url }}/img/postin/new-excelwebaddin-project.png)
>新建项目的过程中，你会发现它有两个选项，默认选择第一个，第二个是更丰富的外接程序，可以实现更多Excel的自定义行为。 
 ![如图所示]({{ site.url }}/img/postin/select-create-excelwebaddin-type.png)
>项目建好之后，项目中的xml文件已经在里面了，项目新建之后会出现默认的节点，并且已经给出了注释，注释非常详细。此处指介绍一些重要的节点，如果感兴趣到此处去了解

* ID： 此处创建GUID可以去线上生成，vs2017也自带创建guid。工具->创建GUID

```
 <!-- 重要事项！ID 对于外接程序必须是唯一的，如果重复使用该清单，请确保将此 ID 改为新的 GUID。 -->
  <Id>2a18a912-de33-4f62-92f7-ce7c2899ea77</Id>
```

* AppDomains:由于浏览器是不允许跨域访问的，你必须要把你web的域名放在此处，才可以使用，如果你想使用多个web项目，那就都在此处列出域名，比如我测试的地址是localhost:8081

```
<!-- 导航时允许使用的域。例如，如果使用 ShowTaskpane，然后得到一个 href 链接，则只有在此列表上存在该域时，才允许导航。 -->
  <AppDomains>
    <AppDomain>localhost:8081</AppDomain>
    <AppDomain>gooogle.com</AppDomain>
    <AppDomain>mywebsite.com</AppDomain>
  </AppDomains>
```

* 默认访问的首页,此处的~remoteAppUrl即是你的url，在测试时会自动替换成你的web服务地址，在上线发布时，需要手动替换

```
<DefaultSettings>
    <SourceLocation DefaultValue="localhost:8081" />
  </DefaultSettings>
```

* Host：此节点下的子节点即是你展示在Excel顶部的菜单栏，使用节点来控制显示
* Resources：此节点存放所有的资源，比如菜单名称、要访问的url、图片的地址
* web项目开始后执行的

```
<GetStarted>
            <!-- “入门”标注的标题。resid 属性指向 ShortString 资源 -->
            <Title resid="Contoso.GetStarted.Title"/>
            <!-- 入门标注的描述。ResID 指向 LongString 资源 -->
            <Description resid="Contoso.GetStarted.Description"/>
            <!-- 指向详细说明外接程序使用方法的 URL 资源。 -->
            <LearnMoreUrl resid="Contoso.GetStarted.LearnMoreUrl"/>
          </GetStarted>
```

* FunctionFile:如果想在菜单栏里操作Excel文件，比如要写东西，必须要在此处增加那个包含js的html文件

```
<!-- 函数文件是包含 JavaScript 的 HTML 页面，将在此页面中调用用于 ExecuteAction 的函数。             将 FunctionFile 视为代码隐藏 ExecuteFunction。 -->
          <FunctionFile resid="Contoso.DesktopFunctionFile.Url" />
```

* 菜单区域：分为组、一级菜单、二级菜单。只能到二级菜单。

```
  <!-- PrimaryCommandSurface 为 Office 主功能区。 -->
          <ExtensionPoint xsi:type="PrimaryCommandSurface">
            <!-- 使用 OfficeTab 来扩展现有选项卡。使用 CustomTab 来创建新选项卡。 -->
            <OfficeTab id="TabHome">
              <!-- 确保为组提供唯一 ID。建议 ID 为使用公司名的命名空间。 -->
              <Group id="Contoso.Group1">
                <!-- 为组指定标签。resid 必须指向 ShortString 资源。 -->
                <Label resid="Contoso.Group1Label" />
                <!-- 图标。必需大小: 16、32、80，可选大小: 20、24、40、48、64。强烈建议为大 UX 提供所有大小。 -->
                <!-- 使用 PNG 图标。资源部分中的所有 URL 必须使用 HTTPS。 -->
                <Icon>
                  <bt:Image size="16" resid="Contoso.tpicon_16x16" />
                  <bt:Image size="32" resid="Contoso.tpicon_32x32" />
                  <bt:Image size="80" resid="Contoso.tpicon_80x80" />
                </Icon>

                <!-- 控件。可以为“按钮”类型或“菜单”类型。 -->
                <Control xsi:type="Button" id="Contoso.TaskpaneButton">
                  <Label resid="Contoso.TaskpaneButton.Label" />
                  <Supertip>
                    <!-- 工具提示标题。resid 必须指向 ShortString 资源。 -->
                    <Title resid="Contoso.TaskpaneButton.Label" />
                    <!-- 工具提示标题。resid 必须指向 LongString 资源。 -->
                    <Description resid="Contoso.TaskpaneButton.Tooltip" />
                  </Supertip>
                  <Icon>
                    <bt:Image size="16" resid="Contoso.tpicon_16x16" />
                    <bt:Image size="32" resid="Contoso.tpicon_32x32" />
                    <bt:Image size="80" resid="Contoso.tpicon_80x80" />
                  </Icon>

                  <!-- 这是触发命令时的操作(例如单击功能区)。支持的操作为 ExecuteFunction 或 ShowTaskpane。 -->
                  <Action xsi:type="ShowTaskpane">
                    <TaskpaneId>ButtonId1</TaskpaneId>
                    <!-- 提供将显示在任务窗格上的位置的 URL 资源 ID。 -->
                    <SourceLocation resid="Contoso.Taskpane.Url" />
                  </Action>
                </Control>
              </Group>
            </OfficeTab>
          </ExtensionPoint>
```

配置好自己的东西之后，F5运行，Excel便会出现初始化好的项目，其中excel单元格里的内容是js写入的，具体如何写入，下文分解。右边侧栏则是需要开发的web项目。![如图所示]({{ site.url }}/img/postin/init-project-run.png)

>第二、创建web项目，其实在新建项目时，web项目已经创建好了。可以在这个项目的基础上开发jQuery项目。如果不想用这个，也需要在此框架的基础上开发其他项目。web项目和加载项项目，是两个完全独立的项目，放在一起只是为了测试方便，web项目独立部署，只是如果你想要让web操作exce 的话，需要引入一些必要的文件与配置。

目前创建有四种web项目的结构。除了jQuery,其他均是目前流行的组件化开发
* Angluar
* Jquery
* React
* Vue

因为接下来要使用vue开发前端，所以删除原框架无用的东西，将整个web项目删掉。![如图所示]({{ site.url }}/img/postin/delete-useless.png)
因为我使用的是Vue，所以此处只介绍Vue搭建站点，其他三种方式去[Office加载项官网](https://docs.microsoft.com/zh-cn/office/dev/add-ins/quickstarts/excel-quickstart-angular)


* 使用 vue-cli脚手架搭建项目，具体步骤就不写了，文档写的很详细[参考](https://cli.vuejs.org/zh/guide/installation.html)

* 搭建好之后，运行vue项目，然后把xml文件里的~remoteurl,替换成你现在的项目地址就可以了。运行结果：
![如图所示]({{ site.url }}/img/postin/init-run-result.png)

## 项目地址
[项目链接](https://github.com/AlvinNiu/ExcelWebAddInDemo)

****

