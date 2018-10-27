---
layout:       post
title:        "Python基础学习笔记"
subtitle:     "python数据分析"
date:         2018-10-27 18:00:00
author:       "Alvin"
header-img:   "img/office-web.jpg"
header-mask:  0.3
catalog:      true
tags:
    - Python
    - 学习笔记
    - 数据类型
    - 技术
---

## 前言
>由于工作涉及到数据处理，听说使用python可以提升数据分析的效率，所以来学习一下。此篇文章只是开始学习的python基础，以及自己理解，防止自己遗忘，随手记下来


# Python基础
## python数据类型
* 整数，任意大小，不设大小限制
* 浮点数，任意大小，不设大小限制，如果超出一定范围，则使用`inf`表示。即无限大
* 字符串
* 布尔值，使用 and or not来运算
* 控制，None表示
* 变量，不需要定义，初始化为什么类型的，该变量即是什么类型
* 常量，常量必须使用大写字母表示


## 字符串与编码
* 目前python使用的Unicode编码，可以支持多种语言
* 中文需要utf-8的编码
* encode():将字符串转换成bytes
* decode():将bytes转换成字符串
* len():判断字符串长度,根据数据类型，字符串判断的是字符长度，bytes判断的是字节数
* 为了中文乱码，最好使用utf-8编码
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
```
* 格式化字符串，即是指定占位符

占位符 | 内容类型
---|---
%d | 整数
%f | 浮点数
%s | 字符串,这个也是万能的，如果你不知道要添加的内容是什么都可以使用这个
%x | 十六进制整数

* format() :格式化字符串,该方法通过{n}来确定替换哪一个位置的占位符
```
'Hello, {0}, 成绩提升了 {1:.1f}%'.format('小明', 17.125)
```

## 集合
* list:可变长度
* tuple：不可变长度
* 对list集合与tuple集合可以进行切片，所谓的切片就是可以取集合中的多少元素
```
arr = [1, 2, 3, 4, 5, 6, 7]
print(arr[0:3])  # [1, 2, 3]
print(arr[:3])  # [1, 2, 3]
print(arr[-3:-1])  # [5, 6]
print(arr)  # [1, 2, 3, 4, 5, 6, 7]
string = '987654321'
print(string[1:3])  # 87
```
* 列表生成式，可以生成想要的列表
```
print(list(range(1, 11)))  # [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# 下面这个是简洁的循环，意思就是1到10 循环，分别对本身做幂运算
print([x * x for x in range(1, 11)])  # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
# 下面的运行结果等于上面
arr = []
for x in range(1, 11):
    arr.append(x*x)
print(arr)

```
* 生成器：我所理解的生成器是指存储生成列表的算法，而不是存储列表。适用于数据量很大，如果一开始就定义在列表里，会消耗很大的内存。生成器便是，在你调用的时候才生成该元素
    * 第一种方法：使用()
    * 第二种，在函数里使用yield关键字，改关键字后面的值代表要算出来的值
    * 调用方法有两种，next与迭代
```
# 第一种生成器方法
g = (x*x for x in range(1, 11))
# 使用next调用下一个值
print(next(g))
arr = []
# 使用迭代器调用下一个值
for i in g:
    arr.append(i)
print(arr)

# 第二种生成器的方法，函数


def power():
    for x in range(1, 11):
        yield x*x


# 调用
arr = []
for i in power():
    arr.append(i)

print(arr)

```
* 可迭代对象：顾名思义就是可以进行迭代的对象，generator、list、tuple、set、dict、str等类型的对象为可迭代对象，即`Iterable`。
* 迭代器：表示的是一个有序的数据流，这个数据流只能使用next才能访问下一个，不能直接选择访问哪一个，这个数据流可以无限长。


## 循环(只有两种)
* for in
```
# 输出从1到9
for i in range(1, 10):
    print(i)
```
* while
```
# 输出从1到9
i = 0
while(i < 9):
    i += 1
    print(i)

```
`注意：`没有i++这种运算方式

## dict与set
* dict :无序的key-vale集合，
* set:无序的不可重复的集合，里面只有key没有value，所以单个值而且需要搜索的，可以是用set


## 函数
### 定义函数 def
```
# 取反函数
def negation(result):
    return not result
```
* 使用from ** import ** 导入
```
# untils 为同目录下的文件名，negation为函数名
from untils import negation
```
* 可以使用pass来占位未想好的代码
* 如果参数传入错误，会抛出TypeError
* 函数执行完毕也没有`return`语句时，自动`return None`
* 函数可以返回多个值，类型就是tuple
```
# 返回多个值
def return_more():
    return 1, 2, 3, 4

print(return_more())

#结果：(1, 2, 3, 4)
```
### map/reduce/filter/sorted
* map(fun,iterable)              
    * fun:函数，该函数有一个参数，
    * iterable：可迭代对象，
    * 返回值：iterator，对象迭代器的算法就是遍历集合中的值然后经过方法计算得到的值

例子如下：

```
# 定义方法
def power(x):
    return x*x


# 执行map方法
# 由于是iterator对象，所以打出来是 <map object at 0x0000000003452668>
print(map(power, range(1, 11)))
arr = []
g = map(power, range(1, 11))
for x in g:
    arr.append(x)
print(arr)  # [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]

```

* reduce(fun,iterable)
    * fun:该函数有两个参数   
    * iterable：可迭代对象
    * 返回值：任意类型，将集合中的当前值与下一个值，传入方法计算得到的值,得到的结果继续与下一个值通过方法计算，直到最后一个元素，得到结果，直接返回。

例子如下：
```
# 由于reduce 移到了functools模块，所以此处需要引用
from functools import reduce
# 定义方法


def sum(x, y):
    return x+y


print(reduce(sum, range(1, 11))) # 55

```

* filter(fun,iterable):筛选函数
    * fun：函数：有一个参数，返回值为布尔类型，想要留下该元素则返回True，否则返回False
    * iterable：可迭代对象，也为待筛选集合对象
    * 返回值：生成器，iterator对象

例子如下：
```
# 只保留大于5的值
def top5(x):
    return x > 5

# 使用list转换成list类型的
print(list(filter(top5, range(1, 11))))

```

* sorted(iterable,key)：排序算法，可以实现自定义排序，也可以使用默认的,
    * iterable:待排序的可迭代对象
    * key：可选参数，自定义排序处理函数,通过该函数处理每个元素后的值，进行拍讯
    * 返回值：可迭代对象

例子如下：
```
# 只保留大于5的值
arr = [1, 5, 6, 8, -4, -3]
print(sorted(arr))  # [-4, -3, 1, 5, 6, 8]
# 通过绝对值排序，得到的结果还是原来的值
print(sorted(arr, key=abs))  # [1, -3, -4, 5, 6, 8]
```

## 参考链接
[廖雪峰官网](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)