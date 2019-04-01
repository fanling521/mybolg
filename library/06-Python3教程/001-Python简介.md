# Python简介

## 什么是Python

Python是*Guido van Rossum*在1989年圣诞节期间，为了打发无聊的圣诞节而编写的一个编程语言。

Python就为我们提供了非常完善的基础代码库，覆盖了网络、文件、GUI、数据库、文本等大量内容，用Python开发，许多功能不必从零编写，直接使用现成的即可。

免费、开源、面向对象，简单易学、可扩展的，丰富的标准库和第三方模块，缺点是运行速度较慢。

### Python的解释器

当我们编写Python代码时，我们得到的是一个包含Python代码的以`.py`为扩展名的文本文件。要运行代码，就需要Python解释器去执行`.py`文件。

官方解释器是`CPython`

## Python的设计目标

### Python的设计哲学

1. 优雅
2. 明确
3. 简单

### Python的特点

1. Python 是完全的面向对象的语言
2. Python 拥有强大的标准库
3. Python 提供了大量第三方库

## 第一个Python程序

### Python的源程序

1. Python 是一个特殊的文本文件，以`.py`结尾

（1）windows下cmd开发

```
C:\Users\fanling>python
Python 3.7.3 (v3.7.3:ef4ec6ed12, Mar 25 2019, 22:22:05) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>> print("hello world")
hello world
>>>
```

（2）新建Hello.py，cmd运行`python Hello.py`

### 注释

```python
# 这是注释
print("hello world")
"""
这是多行注释
"""
```

## Python 基础语法

### 算数运算符

除了正常和其他编程语言一样的以外，说明特有的运算符

- // 取商
- % 取余
- ** 取幂

### Python的变量

python中的变量必须赋值