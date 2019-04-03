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
2. 学习`printf`函数

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

### Python的注释

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

### 逻辑运算符

- and
- or
- not

```python
# 输入整数，判断是不是0-120
num=int(input("请输入一个整数："))
if num>=0 and num<=120:
    print("符合")
else:
    print("不符合")
```

### Python的变量

python中的变量必须赋值，不需要指定数据类型，学习`type`函数，可以看到变量类型。

#### 基本数据类型

Python的数字类型有int整型、（long长整型【python2.x】）、float浮点数、complex复数、以及布尔值（0和1）。

特殊点：boolean 类型：0为**False**，非0为**True**；complex 复数型用于科学计算。

Python的非数字类型有字符、列表、字典、元组、集合

#### 变量的输入

使用`input`函数

```python
>>> userName=input("请输入用户名：")
请输入用户名：fanl
>>> print(userName)
```

#### 类型转换函数

`int()`函数：将一个变量转换成整型

`float()`函数：将一个变量转换成浮点型

#### 格式化输出

| 格式化符号 | 意义                                        |
| ---------- | ------------------------------------------- |
| %s         | 输出字符串                                  |
| %d         | 输出十进制整数，%06d表示输出的位数，不足补0 |
| %f         | 输出小数，%.02f表示小数点后保留2位          |
| %%         | 输出%                                       |

**格式**

`print(“格式化字符串”%变量)`

`print(“格式化字符串”%(变量1,变量2…))`

```python
# 输入苹果的单价
price = float(input("请输入苹果的单价："))
# 输入购买的数量
weight = float(input("请输入苹果的重量："))
# 计算金额
print("苹果单价是%f元/斤，您购买了%f斤，需要支付%.02f元" % (price, weight, weight * price))
```

#### 变量命名以及标识符

1. 每个单词都用小写字母
2. 单词和单词之间用下划线连接

**关键词查看**

```python
import keyword
print(keyword.kwlist)
```

### 语句结构

- 缩进用空格
- 空格和Tab不能混用

#### if语句

**语法结构**：

```python
if [条件]:
   执行的语句
#-----------------
age =19
if age>18:
    print("hello")
```

#### if-else语句

```python
if [条件]:
	执行的语句
else:
	执行的语句
```

#### if-elif-else语句

```python
holiday_name = input("女朋友问今天是什么节日?")
if holiday_name == "情人节":
    print("买玫瑰，看电影")
elif holiday_name == "平安夜":
    print("买苹果，吃大餐")
else:
    print("每天都是节日")
```

