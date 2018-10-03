---
layout:       post
title:        "翻译-ExcelDNA开发文档"
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

>翻译开源项目ExcelDNA开发文档

## 异步处理

ExcelDNA支持两种异步函数：
* RTD，该函数适用与Excel2003及以上版本，（当你使用ExcelAsyncUtil.*时，RTD起作用）
* 本地Excel异步函数，使用Excel2010及以上版本（当你的函数使用ExcelAsyncHandle作为参数并且返回值为void时）

两种方式的不同之处
* RTD函数允许你在函数执行的时候，与Excel通信
* 本地异步函数，当函数已经被执行时，Excel只允许执行工作薄的其他部分。所以在单元格计算时，你不并能影响Excel。

RTD的使用方法之一是：ExcelAsyncUtil.Run。为了方便执行，ExcelDNA在内部定义了RTD服务（[详情](https://docs.microsoft.com/en-us/previous-versions/office/developer/office-xp/aa140060(v=office.10))）RTD服务允许ExcelDNA通知Excel，例如：在异步任务完成之后，该公式是否需要重新计算。

异步函数如下：
* 你定义一个异步函数在公式里
* Excel需要重新计算，调用异步函数执行ExcelAsyncUtil.Run 
* 使用ExcelAsyncUtil.Run 创建RTD主题（引用方法名和在参数中信息中在让ExcelAsyncUtil.Run作为第一个参数）
* ExcelAsyncUtil.Run 使异步工作开始，并且与RTD主题保持联系
* 当持续计算时，ExcelAsyncUtil.Run会返回#N/A给UDF。
* 你的UDF会返回#N/A到Excel工作薄
* 当你工作完成时，ExcelDNA会发送标识让RTD更新
* 当RTD出现错误时，Excel会在单元格上标记
* Excel重新计算时，会让单元格重新计算，会让UDF重新被调用
* UDF调用ExcelAsyncUtil.Run和之前的主题信息一致
* ExcelAsyncUtil.Run查找在方法中存储的主题和完成的值，它会直接返回值，如果找不到会返回#N/A
* UDF函数会接收到结果值，并且将值返回给工作薄
* 由于ExcelAsyncUtil.Run不能再次调用Excel RTD方法，所以RTD主题会清空Excel内部的值

## ExcelFunction 属性


* `Name`
* `Description`
* `Category`
* `HelpTopic`
* `IsVolatile` (`!` suffix)
* `IsHidden`
* `IsExceptionSafe`
* `IsMacroType` (`#` suffix)
* `IsThreadSafe` (`$` suffix)
* `IsClusterSafe` (`&` suffix)
* `ExplicitRegistration`
* `SuppressOverwriteError`

### IsMacroType
是否使用宏类型
当IsMacroType=true时，ExcelDNA注册该函数，会调用**xlfRegister**
[Excel API Reference for `xlfRegister`](https://msdn.microsoft.com/en-us/library/office/bb687900.aspx)。详细说，如果IsMacroType=true，ExcelDNA会在pxTypeText后加"#"
相关文档中如此描述
>在**pxTypeText**最后一个参数加#，给函数相同的调用许可证，作为一个宏中的函数
>* 函数会获取在重新计算的循环中，尚未计算的单元格的值
>* 这个函数可以调用任一xlm信息中的函数，例如：**xlfGetCell**

>如果并没有出现#符号
>* 求并没有计算的单元格的结果，会出现**xlretUncalced** 错误。一旦单元格被计算，当前函数会被再次调用
>* 调用xlm中除**xlfCaller**以外的函数，会引发**xlretInvXlfn**错误


使用IsMacroType=true的一些弊端
* 如果不加"$"后缀，不能使用多线程，即使被标记了IsThreadSafe=true
* 如果他们包含至少一种参数，标记**[ExcelArgument(AllowReference=true)]**的类型参数，则Excel自动将函数视为不稳定性（即使函数被标记IsVolatile=false）

进一步看，我理解的是,在Excel计算期间,这些函数处理不同的线程，因此，你可能期待在工作薄计算时，会有一些变化，我并没有引用也没有重写它。

我建议值在意料之外的案例中只设置IsMacroType=true，当你确定需要并且已经足够了解时，你可以升级

### IsThreadSafe

是否线程安全，设置为true时，意味着你的函数安全的多线程重新计算，如果在注册字符串最后加上"$"符号，可以在内部调用**xlfRegister**

### IsClusterSafe

是否集群安全，设置为true时，意味着你的函数在集群时安全
[Cluster safe functions](https://docs.microsoft.com/en-us/office/client-developer/excel/cluster-safe-functions)

### IsExceptionSafe

是否异常安全，设置为true时，意味着无论何时出现未知的异常时，Excel应该崩溃，该参数最好忽略

## ExcelArgumentAttribute


* `Name`
* `Description`
* `AllowReference`

## ExcelCommandAttribute

* `Name`
* `Description` (_Unused_)
* `HelpTopic` (_Unused_)
* `ShortCut`
* `MenuName`
* `MenuText`
* `IsExceptionSafe`
* `ExplicitRegistration`
* `SuppressOverwriteError`

## 函数注册

### 默认注册

默认情况下，ExcelDNA所有的函数注册方法必须是 `public static`,为的是能在.dna文件中访问到，下面有两个属性，你可以放在.dna文件中便于控制你的注册。

#### ExplicitExports
* 如果你仅仅想注册一个方法并且明确的标志他是一个Excel方法或者Excel命令，你可以在.dna文件中假如 `ExplicitExports='true'`
例如：

```
<ExternalLibrary Path="MyFunctions.dll" Pack="true" ExplicitExports="true" />
```
* 这些属性在.dna文件中的项目与类库中的标记中是有效的

### 显式注册选项

如果你的AddIn明确的注册（如果你要注册一个扩展类库），你可以在.dna文件中增加`ExplicitRegisration='true'`，在ExcelDNA中，并不会自动注册任一个函数，并且你的AddIn可以使用`ExcelIntegration.RegisterDelegates(...)`被调用，
`ExplicitRegistration`选项运行允许明确的方法或者类库退出默认的注册处理，例如，方法或者类库有`explicitly register`，则调用`ExcelIntegration.RegisterXXX `方法中的其中一个
* 在`ExcelFunctionAttribute ` 和`ExcelCommandAttribute`均有效
* 在.dna文件中的`Project`和`ExternalLibrary`中有效
* 当`ExcelIntegration.RegisterMethods`或者`ExcelIntegration.RegisterDelegates`被调用时，这些标志会在任一属性被调用之前删除

### 动态注册

ExcelDNA自定义外部类库，可以通过自定义方法与注册线程，动态注册函数，此处使用了委托，`ExcelIntegration.RegisterDelegates`。[详情](https://github.com/Excel-DNA/Registration)

## 数据类型

以下是函数的参数以及返回值的类型
* Double
* String
* DateTime    -- 返回double类型到Excel (或许直接返回字符串会更好)
* Double[]    -- 如果只有一列,则取该列，否则将会使用该行
* Double[,]
* Object
* Object[]    -- 如果只有一列,则取该列，否则将会使用该行
* Object[,]
* Boolean (bool) -- 返回Excel布尔值 (返回一个字符串会更好)
* Int32 (int)
* Int16 (short)
* UInt16 (ushort)
* Decimal

传入函数的参数类型，只允许传入以下的参数:
* Double
* String
* Boolean
* ExcelDna.Integration.ExcelError
* ExcelDna.Integration.ExcelMissing
* ExcelDna.Integration.ExcelEmpty
* Object[,] containing an array with a mixture of the above types
* ExcelReference -- (只有 AllowReference=true 时)

参数类型为 Object[] 、 Object[,] 的函数将接受上述类型，返回类型如下:
* Double
* String
* DateTime
* Boolean
* Double[]
* Double[,]
* Object[]
* Object[,]
* ExcelDna.Integration.ExcelError
* ExcelDna.Integration.ExcelMissing.Value // Converted by Excel to be 0.0
* ExcelDna.Integration.ExcelEmpty.Value   // Converted by Excel to be 0.0
* Int32 (int)
* Int16 (short)
* UInt16 (ushort)
* Decimal
* otherwise return #VALUE! error


## 自定义编译输出

当**ExcelDna.AddIn**NuGet包被安装在项目中，一些附加的编译配置已经被定义好，此时，只需要Copy所需的.xll文件到输出目录，即可以创建一个用于插件的单独的包。

安装包会增加文件到项目中（Properties\ExcelDna.Build.props）。这个文件被用于自定义需要哪些附件。**ExcelDna.Build.props**允许配置一下内容

```
  <!--
    Configuration properties for building .dna files
  -->
  <PropertyGroup>
    <!--
      Enable/Disable automatic generation of platform-specific versions of .dna files
    -->
    <ExcelDnaCreate32BitAddIn Condition="'$(ExcelDnaCreate32BitAddIn)' == ''">true</ExcelDnaCreate32BitAddIn>
    <ExcelDnaCreate64BitAddIn Condition="'$(ExcelDnaCreate64BitAddIn)' == ''">true</ExcelDnaCreate64BitAddIn>

    <!--
      Define the suffix used for each platform-specific file e.g. MyAddIn64.dna
    -->
    <ExcelDna32BitAddInSuffix Condition="'$(ExcelDna32BitAddInSuffix)' == ''"></ExcelDna32BitAddInSuffix>
    <ExcelDna64BitAddInSuffix Condition="'$(ExcelDna64BitAddInSuffix)' == ''">64</ExcelDna64BitAddInSuffix>
  </PropertyGroup>

  <!--
    Configuration properties for packing .dna files
  -->
  <PropertyGroup>
    <!--
      Enable/Disable packing of .dna files
    -->
    <RunExcelDnaPack Condition="'$(RunExcelDnaPack)' == ''">true</RunExcelDnaPack>

    <!--
      Suffix used for packed .xll files e.g. MyAddIn-packed.xll
    -->
    <ExcelDnaPackXllSuffix Condition="'$(ExcelDnaPackXllSuffix)' == ''">-packed</ExcelDnaPackXllSuffix>
  </PropertyGroup>
```


## 相关链接

[文档地址](https://github.com/Excel-DNA/ExcelDna/wiki)