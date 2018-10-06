---
layout:       post
title:        "Windows下安装ElasticSearch及工具"
subtitle:     "让你的搜索变得无比简单"
date:         2018-10-06 13:00:00
author:       "Alvin"
header-img:   "img/elasticsearch-install.jpg"
header-mask:  0.3
catalog:      true
tags:
    - ElasticSearch
    - Kibana
    - 搜索引擎
    - 技术
    - X-pack
---

## 前言

### 什么是ElasticSearch
>[官网如是介绍](https://www.elastic.co/guide/cn/elasticsearch/guide/current/preface.html)：Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎。 它能从项目一开始就赋予你的数据以搜索、分析和探索的能力，这是通常没有预料到的。 Elasticsearch 不仅仅只是全文搜索，我们还将介绍结构化搜索、数据分析、复杂的语言处理、地理位置和对象间关联关系等。

### 目的
>由于ElasticSearch更新的很快，所以安装它及其插件时，遇到一些坑，在国内并未找到答案，所以在此记录一下，为了以后给再遇到的这些问题的人（也包括自己）。

## 安装ElasticSearch
* 版本：6.4.2
* [下载地址](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.2.zip) 
* 解压到本地目录
* 运行bin目录下的elasticsearch.bat文件（此文件是批处理文件，在Windows下双击也可以，但是双击之后，如果出现错误，我们是看不见的，所以此处不建议双击）

```
#进入cmd控制台，进入elasticsearch.bat文件所在目录，然后运行如下命令
elasticsearch.bat
```
* 浏览器访问http://localhost:9200/，如果出现如下则服务启动成功，否则按照错误信息，去查找

```
{
  "name" : "node-1",
  "cluster_name" : "my-application",
  "cluster_uuid" : "tE18H2eXSimOzKKgAyZMtw",
  "version" : {
    "number" : "6.4.2",
    "build_flavor" : "default",
    "build_type" : "zip",
    "build_hash" : "04711c2",
    "build_date" : "2018-09-26T13:34:09.098244Z",
    "build_snapshot" : false,
    "lucene_version" : "7.4.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

### 安装ElasticSearch问题集锦


1. 运行elasticsearch.bat 出现如下问题

```
此时不应有 \Java\jdk1.8.0_181\bin\java.exe" -cp "!ES_CLASSPATH!" "org.elasticsearch.tools.launchers.JvmOptionsParser" "!ES_JVM_OPTIONS!" || echo jvm_options_parser_failed"`)
```
>解决方案:将elasticsearch.bat文件中的,[解决方案来源](https://github.com/elastic/elasticsearch/issues/30606)

```
set "ES_JVM_OPTIONS=%ES_PATH_CONF%\jvm.options"
@setlocal
for /F "usebackq delims=" %%a in (`"%JAVA% -cp "!ES_CLASSPATH!" "org.elasticsearch.tools.launchers.JvmOptionsParser" "!ES_JVM_OPTIONS!" || echo jvm_options_parser_failed"`) do set JVM_OPTIONS=%%a
@endlocal & set "MAYBE_JVM_OPTIONS_PARSER_FAILED=%JVM_OPTIONS%" & set ES_JAVA_OPTS=%JVM_OPTIONS:${ES_TMPDIR}=!ES_TMPDIR!% %ES_JAVA_OPTS%
```
替换成

```
set ES_JVM_OPTIONS=%ES_PATH_CONF%\jvm.options
@setlocal
for /F "usebackq delims=" %%a in (`CALL %JAVA% -cp "!ES_CLASSPATH!" "org.elasticsearch.tools.launchers.JvmOptionsParser" "!ES_JVM_OPTIONS!" ^|^| echo jvm_options_parser_failed`) do set JVM_OPTIONS=%%a
@endlocal & set "MAYBE_JVM_OPTIONS_PARSER_FAILED=%JVM_OPTIONS%" & set ES_JAVA_OPTS=%JVM_OPTIONS:${ES_TMPDIR}=!ES_TMPDIR!% %ES_JAVA_OPTS%
```
2. 出现如下错误


```
[WARN ][o.e.b.ElasticsearchUncaughtExceptionHandler] [] uncaught exception in thread [main]
org.elasticsearch.bootstrap.StartupException: ElasticsearchException[X-Pack is not supported and Machine Learning is not available for [windows-x86]; you can use the other X-Pack features (unsupported) by setting xpack.ml.enabled: false in elasticsearch.yml]
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:140) ~[elasticsearch-6.4.2.jar:6.4.2]
        at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:127) ~[elasticsearch-6.4.2.jar:6.4.2]
        at org.elasticsearch.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:86) ~[elasticsearch-6.4.2.jar:6.4.2]
        at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:124) ~[elasticsearch-cli-6.4.2.jar:6.4.2]
        at org.elasticsearch.cli.Command.main(Command.java:90) ~[elasticsearch-cli-6.4.2.jar:6.4.2]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:93) ~[elasticsearch-6.4.2.jar:6.4.2]
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:86) ~[elasticsearch-6.4.2.jar:6.4.2]
Caused by: org.elasticsearch.ElasticsearchException: X-Pack is not supported and Machine Learning is not available for [windows-x86]; you can use the other X-Pack features (unsupported) by setting xpack.ml.enabled: false in elasticsearch.yml
        at org.elasticsearch.xpack.ml.MachineLearningFeatureSet.isRunningOnMlPlatform(MachineLearningFeatureSet.java:103) ~[?:?]
        at org.elasticsearch.xpack.ml.MachineLearningFeatureSet.isRunningOnMlPlatform(MachineLearningFeatureSet.java:94) ~[?:?]
        at org.elasticsearch.xpack.ml.MachineLearning.createComponents(MachineLearning.java:374) ~[?:?]
        at org.elasticsearch.node.Node.lambda$new$8(Node.java:420) ~[elasticsearch-6.4.2.jar:6.4.2]
        at java.util.stream.ReferencePipeline$7$1.accept(ReferencePipeline.java:267) ~[?:1.8.0_181]
        at java.util.ArrayList$ArrayListSpliterator.forEachRemaining(ArrayList.java:1382) ~[?:1.8.0_181]
        at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:481) ~[?:1.8.0_181]
        at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:471) ~[?:1.8.0_181]
        at java.util.stream.ReduceOps$ReduceOp.evaluateSequential(ReduceOps.java:708) ~[?:1.8.0_181]
        at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:234) ~[?:1.8.0_181]
        at java.util.stream.ReferencePipeline.collect(ReferencePipeline.java:499) ~[?:1.8.0_181]
        at org.elasticsearch.node.Node.<init>(Node.java:423) ~[elasticsearch-6.4.2.jar:6.4.2]
        at org.elasticsearch.node.Node.<init>(Node.java:256) ~[elasticsearch-6.4.2.jar:6.4.2]
        at org.elasticsearch.bootstrap.Bootstrap$5.<init>(Bootstrap.java:213) ~[elasticsearch-6.4.2.jar:6.4.2]
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:213) ~[elasticsearch-6.4.2.jar:6.4.2]
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:326) ~[elasticsearch-6.4.2.jar:6.4.2]
        at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:136) ~[elasticsearch-6.4.2.jar:6.4.2]
        ... 6 more
```
>解决方案：将jdk与jre安装成64位,[解决来源](https://discuss.elastic.co/t/x-pack-is-not-supported-and-machine-learning-is-not-available-for-windows-x86/85084)


## 安装Kibana

### 什么是Kibana
>Kibana 是通向 Elastic 产品集的窗口。 它可以在 Elasticsearch 中对数据进行视觉探索和实时分析。此处简单介绍，稍后会专门写一篇博客介绍Kibana及其用法
* 版本：6.4.2
* [下载地址](https://artifacts.elastic.co/downloads/kibana/kibana-6.4.2-windows-x86_64.zip)
* 下载下来之后，解压到相关目录下
* 打开 config/kibana.yml，并修改其中elasticsearch.url的值,此处的URL要修改成ElasticSearch启动的URL，此处是默认值，如果ElasticSearch的URL没有更改，此处也不用改。

```
elasticsearch.url: "http://localhost:9200"
```
* 使用cmd进入Kibana的bin目录，然后运行kibana.bat文件。

```
kibana.bat
```
* 打开浏览器，访问http://localhost:5601即可。如果该端口被占用了，在启动的cmd控制台有显示访问的地址，复制访问即可。

![如图所示]({{ site.url }}/img/postin/kibana-index.png)
我在安装Kibana的时候没有遇到问题。

## 插件们

下面是ElasticSearch 5.X 版本才有的插件，6.X的插件都集成到Kibana与x-pack中了
![如图所示]({{ site.url }}/img/postin/elasticsearch5x-plugin.png)

### X-Pack

在安装插件的时候发现，该插件已经集成到ElasticSearch中了，所以不需要单独安装。

## 参考链接

* [Elastic官网](https://www.elastic.co)
* [Elastic社区](https://discuss.elastic.co/t/x-pack-is-not-supported-and-machine-learning-is-not-available-for-windows-x86/85084)
* [Elastic GitHub问答](https://github.com/elastic/elasticsearch/issues/30606)

## 小结
安装总结起来还是比较简单的，接下来就是开始进行搜索了。