# Python简介

本教程不适合编程初学者使用，希望你有其他编程语言的基础，如Java。

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
双引号这是多行注释
双引号这是多行注释
"""

'''
单引号也可以使用多行注释
单引号也可以使用多行注释
'''
```

## Python 基础语法

### 算数运算符

除了正常和其他编程语言一样的以外，说明特有的运算符

- // 取商
- % 取余
- ** 取幂
- 字符串乘法 “hello”* 3，也支持元组和列表

`in`和`not in`判断字符串，列表，字典，元组中元素是否存在。

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

### 赋值运算符

和其他Java一样，如 `c+=a`等效于`c=a+c`

### Python的变量

python中的变量必须赋值，不需要指定数据类型，学习`type`函数，可以看到变量类型。

打印函数`print`

（1）字符串的乘法`print("*" * 5)`

（2）输出不换行`print("hello" , end="")`

#### 基本数据类型

- Numbers（数字）
- String（字符串）
- List（列表）
- Tuple（元组）
- Dictionary（字典）

Python的数字类型有int整型、（long长整型【python2.x】）、float浮点数、complex复数、以及布尔值（0和1），特殊点：boolean 类型：0为**False**，非0为**True**，complex 复数型用于科学计算。

Python的非数字类型有字符、列表、字典、元组、集合

#### 变量的输入

除了和Java一样的赋值以外，Python支持多变量赋值，如a, b = 1, 2。

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

### Python的语句结构

编写代码的注意点：

- 缩进用空格
- 空格和Tab不能混用

**注意**：顺序结构的语句不再说明。

#### 条件语句

##### if单分支语句

```python
if [条件]:
   执行的语句
#-----------------
age =19
if age>18:
    print("hello")
```

##### if-else双分支语句

```python
if [条件]:
	执行的语句
else:
	执行的语句
```

##### if-elif-else多分支语句

```python
holiday_name = input("女朋友问今天是什么节日?")
if holiday_name == "情人节":
    print("买玫瑰，看电影")
elif holiday_name == "平安夜":
    print("买苹果，吃大餐")
else:
    print("每天都是节日")
```

综合案例，石头剪刀布。

新的模块，`random`，使用其中的`randint`函数，`randint(1,3)`，包含1和3

```python
import random
# 提示玩家输入
player = int(input("请出拳1.石头/2.剪刀/3.布"))
# 电脑出拳
computer = random.randint(1,3)
print("玩家出拳：%d-电脑出拳：%d" % (player, computer))
# 判断胜负
"""
玩家赢的情况
石头1  剪刀2
剪刀2  布3
布3    石头1
"""
if ((player == 1 and computer == 2)
        or (player == 2 and computer == 3)
        or (player == 3 and computer == 1)):
    print("玩家胜利，电脑弱爆了！")
elif player == computer:
    print("居然平局了！")
else:
    print("玩家不服，决战到天明！")
```

#### 循环语句

##### while语法

```python
while [条件]:
	条件满足，执行
# ===============	
i = 1
while i <= 5:
    print("hello python")
    i = i + 1
```

##### break和continue的使用

**break**：退出当前循环，不再执行循环语句

**continue**：退出此次循环，继续执行其他循环语句

```python
i = 0
while i <= 5:
    if i == 3:
        # 需要加上计数器
        i += 1
        continue
    print(i)
    i += 1
```

**注意点**：使用`continue`需要保证计数器已经变化，否则会造成死循环。

##### 循环案例

（1）打印*

```python
# 方法1：循环+字符串乘法
row = 1
while row <= 5:
    print("*" * row)
    row += 1
    
# 方法2：循环嵌套
row = 1
while row <= 5:
    col = 1
    while col <= row:
        print("*", end="")
        col += 1
    print("")
    row += 1
```

（2）九九乘法表

```python
row = 1
while row <= 9:
    col = 1
    while col <= row:
        print("%d*%d=%d\t" %(col,row,col*row), end="")
        col += 1
    print("")
    row += 1
```

