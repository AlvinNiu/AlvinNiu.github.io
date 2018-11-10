---
layout:       post
title:        "Python面向对象学习笔记"
subtitle:     "python数据分析"
date:         2018-11-10 18:00:00
author:       "Alvin"
header-img:   "img/python-basic.jpg"
header-mask:  0.3
catalog:      true
tags:
    - Python
    - 学习笔记
    - 面向对象
    - 技术
---

## 前言
> python也是高级语言，现在的高级语言大多数都是面向对象编程的。当然python的面向对象编程与其他的语言有很多不同的特性，也有很多类似的地方。面向对象中最重要的两个名词，`类`与`实例`。类是用来定义对象的模板，表示这个对象是个什么样子的。而实例则是一个实实在在的对象。通过类的定义来实例化一个对象。而实例化的对象则存在于内存中。

## 类

* 定义类
使用关键字，class来定义类，我用过的所有高级语言都是这么定义类的。

```
class classname(object):
    pass
```
* 构造函数

使用 `__init__` 函数作为构造函数,第一个参数为self，表示对象本身，后面的参数个数没有限制，为对象的属性进行初始化赋值。（`注意:`这里的init前后都是两个下划线）

```
class classname(object):
    def __init__(self,name,age):
        self._name=name
        self._age=age
```
* 实例属性和类属性
    * 实例属性为实例化的对象所有，所以每个实例化属性的值是不相等的。每个实例在内存中都会开辟一块内存
    * 类属性：为类所有，每个对象都可以访问，并且访问的值都是一样的，其中有一个对象改变了类属性，其他对象访问到的值也会改变（此处改变需要注意）

```
# name与age为实例属性
class classname(object):
    def __init__(self,name,age):
        self._name=name
        self._age=age
# type 为类属性
class classname(object):
    type='person'
```
### 实例属性
此处重点说一下实例属性，因为类中最常用到的就是实例属性与实例方法。
由于python中不需要定义变量，当然也不需要定义属性，赋值即定义：
* 在构造函数中初始化

```
class Person(object):
    type = 'person'

    def __init__(self, name, age):
        self._name = name
        self._age = age


# 实例化
person = Person('alvin', 18)
# 调用
print('姓名：{0}，年龄：{1}'.format(person._name, person._age))
```
* 在实际开发过程中要对属性进行过滤，不一定所有的值都要进入，比如人类有两个性别，肯定不允许错误数据进入，所以要使用下面的方法

```
class Person(object):
    type = 'person'

    def __init__(self, name, age):
        self._name = name
        self._age = age

    def get_sex(self):
        return self._sex

    def set_sex(self, sex):
        sexs = ('男', '女')
        if(sex in sexs):
            self._sex = sex
        else:
            print('输入错误')


# 实例化
person = Person('alvin', 18)
# 调用
print('姓名：{0}，年龄：{1}'.format(person._name, person._age))  # 姓名：alvin，年龄：18
person.set_sex('其他')  # 输入错误
person.set_sex('男')  # 男
print(person.get_sex())
```

* 但是上面这种方式又太繁琐，所以利用装饰器@property

```
class Person(object):
    type = 'person'

    def __init__(self, name, age):
        self._name = name
        self._age = age

    @property
    def sex(self):
        return self._sex

    @sex.setter
    def sex(self, sex):
        sexs = ('男', '女')
        if(sex in sexs):
            self._sex = sex
        else:
            print('输入错误')

# 实例化
person = Person('alvin', 18)
# 调用
print('姓名：{0}，年龄：{1}'.format(person._name, person._age))  # 姓名：alvin，年龄：18
person.sex = '其他'  # 输入错误
person.sex = ('男')  # 男
print(person.sex)
```

## 面向对象的三大特性

### 封装

所谓封装就是类要干的事情，类把一个事物的特性封装到一起，事物拥有的特性就是类封装的属性，事物要做的事情就是类封装的方法。

### 继承

继承表示，一个类可以使用其他类的属性与方法，这样大大提升了开发效率与可维护性，举个简单的例子，动物是一个类，这个类具备的属性是动物都具备，猴、鱼、鸟这些都是动物，具备动物的属性，所以这些类只要继承动物这个类即可。动物类为父类，这些类为子类。

```
class Animal(object):
    def __init__(self, age):
        self.age = age

    def move(self):
        print('动物都可以动')

# 继承


class Tiger(Animal):
    pass


# 调用
tiger = Tiger(1)
print('老虎{0}岁了'.format(tiger.age))
tiger.move()
```
除了上面继承过来之外，子类可以扩展类，自定义一些方法与属性，如果子类觉得继承的方法不适合自己，也可以重写方法。

在python中一个类可以同时继承多个类，具体做法如下：

```
class Sub(P1,P2,P3):
    pass
```

### 多态
