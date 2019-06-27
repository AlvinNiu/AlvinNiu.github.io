---
layout:       post
title:        "vscode-插件集合"
subtitle:     "神奇的插件"
date:         2019-06-27 12:00:00
author:       "Alvin"
header-img:   "img/search-engine-optimization.jpg"
header-mask:  0.3
catalog:      true
tags:
    - 工具
    - vscode插件
    - vscode plugin
---

## 前言
> 使用vscode开发前端无疑是不错的选择，毕竟vscode上有大量的插件提供使用，而且非常方便实用，有些插件还非常炫酷，下面的插件用到一些，就记录一些吧

## 插件集合

### Markdown相关

- Markdown PDF: 用于将Markdown文件转换成pdf，[教程](https://marketplace.visualstudio.com/items?itemName=yzane.markdown-pdf)
- Markdown Preview Enhanced：Markdown预览

### 翻译

- Chinese (Simplified) Language Pack for Visual Studio Code：适用于 VS Code 的中文（简体）语言包

### 路径

- Path Autocomplete：路径自动补全
- Path Intellisense：路径自动识别

### 前端开发相关

- Prettier - Code formatter:格式化代码

```
    // 保存自动化
    "editor.formatOnSave": true,
    "prettier.singleQuote": true,
    "prettier.semi": false,
```

- Vetur ：vue开发的工具 [文档](https://vuejs.github.io/vetur/)

```
    "vetur.validation.template": false,
    "vetur.format.defaultFormatter.html": "js-beautify-html",
    "vetur.format.defaultFormatterOptions": {
      "js-beautify-html": {
        "wrap_attributes": "force" // 可以换成上面任意一种value
      }
```

- ESLint：代码检查  [文档](https://eslint.org/docs/user-guide/getting-started)

```
    "eslint.validate": ["javascript", "javascriptreact", "html", "vue"],
    "eslint.options": {
      "plugins": ["html"]
    },
```
