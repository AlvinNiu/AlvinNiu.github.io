---
layout:       post
title:        "开发中的小工具-统计代码行数"
subtitle:     "这些小工具往往能给你惊喜"
date:         2019-01-23 15:00:00
author:       "Alvin"
header-img:   "img/search-engine-optimization.jpg"
header-mask:  0.3
catalog:      true
tags:
    - 工具
    - vscode插件
    - vscode plugin
    - 代码行数统计
---

## 前言
> 暂时不想写


## VSCode 插件中的 LineCounter
从[此处](http://www.voidcn.com/article/p-mmgtdsbq-gp.html)知道的LineCounter
> 此处不得不说的一个是，网上普遍说的一个插件叫：line-counter，我用了之后发现不好用，或许是我配置有问题吧（我是按照插件介绍一步一步配置的）

- 安装插件，搜索LineCounter，点击安装，重新加载即可
- 配置设置区（文件-》首选项-》设置）,将下面代码复制过去即可。

```
"LineCount.showStatusBarItem": true,

  "LineCount.includes": ["**/*"],
  //筛除所有你不想要统计的目录
  "LineCount.excludes": ["**/.vscode/**", "**/node_modules/**", "**/dist/**"],

  "LineCount.output": {
    "txt": true,
    "json": true,
    "csv": true,
    "md": true,
    "outdir": "out"
  },
  "LineCount.sort": "filename",

  "LineCount.order": "asc",

  "LineCount.comment": [
    {
      "ext": ["c", "cpp", "java"],
      "separator": {
        "linecomment": "//",
        "linetol": false,
        "blockstart": "/*",
        "blockend": "*/",
        "blocktol": false,
        "string": {
          "doublequotes": true,
          "singlequotes": true
        }
      }
    },
    {
      "ext": ["html"],
      "separator": {
        "blockstart": "<!--",
        "blockend": "-->"
      }
    }
  ]
```

- 配置完成之后，快捷键：Ctrl+Shift+P（另一种：查看-》命令面板）调出命令面板,按照需求输入以下命令：
    - LineCount:Count current file
    - LineCount:Count Workspace files

> 第一个命令表示，查看当前文件的行数，第二个命令表示查看当前工作空间的所有文件的行数

- 结果如下所示：
![如图所示]({{ site.url }}/img/postin/dev-tools-statics-linecount-result.png)
![如图所示]({{ site.url }}/img/postin/dev-tools-statics-linecount-result2.png)

## 直接通过目录查看文件行数的工具SourceCounter

[点击下载](https://github.com/AlvinNiu/SourceCounter)

> 优点就是方便，不需要配置什么，直接下载即可使用。

> 缺点：
- 只能在Windows下使用
- 只支持一些比较早的语言（.net、java、xml、json、Python、html、js等），像vue项目它就不支持了

> 虽然它有上述缺点，但是不妨碍我喜欢使用这个，就一个方便就足够了

## 使用正则的方式

> 网上有些人利用全局搜索与正则搜索来找到想要的行数，这种做法并没有满足我的需求，但是不失为一种方式，此处列出来，与君共享
转载自[【Python】在VSCode中统计有效代码行数](http://www.zgljl2012.com/python-zai-vscodezhong-tong-ji-you-xiao-dai-ma-xing-shu/)

```
^b*[^:b#/]+.*$
```


