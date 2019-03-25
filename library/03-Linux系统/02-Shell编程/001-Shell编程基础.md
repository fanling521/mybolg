# Shell编程基础

## 什么是Shell
Shell 是一种脚本语言。

Shell 脚本是实现 Linux 系统自动管理以及自动化运维所必备的工具。

Shell 除了能解释用户输入的命令，将它传递给内核，还可以：

- 调用其他程序，给其他程序传递数据或参数，并获取程序的处理结果；
- 在多个程序之间传递数据，把一个程序的输出作为另一个程序的输入；
- Shell 本身也可以被其他程序调用。

### 常用的Shell

- sh 的全称是 Bourne shell
- csh，这个 shell 的语法有点类似C语言
- tcsh 是 csh 的增强版，加入了命令补全功能，提供了更加强大的语法支持
- ash，一个简单的轻量级的 Shell，占用资源少，适合运行于低内存环境，但是与下面讲到的 bash shell 完全兼容
- bash shell 是 Linux 的默认 shell，bash 由 GNU 组织开发，保持了对 sh shell 的兼容性，是各种 Linux 发行版默认配置的 shell

> 查看 Shell

通过 cat 命令来查看当前 Linux 系统的可用 Shell

```shell
[root@fanl01 ~]$ cat /etc/shells
```

在现代的 Linux 上，sh 已经被 bash 代替，`/bin/sh`往往是指向`/bin/bash`的符号链接。

查看当前 Linux 的默认 Shell

```shell
[root@fanl01 ~]$ echo $SHELL
```

## Shell 基本使用方式

### Shell提示符：#和$

对于普通用户，Base shell 默认的提示符是美元符号$；对于超级用户（root 用户），Bash Shell 默认的提示符是井号#。该符号表示 Shell 等待输入命令。

不同的 Linux 发行版使用的提示符格式不同。

```shell
[root@fanl01 ~]$
# 用户名@主机名 当前目录
# Shell 通过PS1和PS2两个环境变量来控制提示符格式
[root@fanl01 ~]$ echo $PS1
[root@fanl01 ~]$ echo $PS2
# 可参考网上字符格式
```

### 字符打印输出

echo 是一个输出命令，可以用来输出数字、变量、字符串等；本例中，我们使用 echo 来输出字符串。

### 第一个Shell脚本

打开文本编辑器，新建一个文本文件，并命名为 test.sh。

扩展名sh代表 shell，扩展名并不影响脚本执行。

```bash
#!/bin/bash
echo "Hello World !"  # 这是一条语句
```

- 第 1 行的`#!`是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种Shell；后面的`/bin/bash`就是指明了解释器的具体位置。

- ` #`及其后面的内容是注释。

```bash
#!/bin/bash
# Copyright (c) fanl
echo "What is your name?"
read PERSON
echo "Hello, $PERSON"
```

`read` 命令用来从标准输入文件（Standard Input，stdin，一般就是指终端）读取用户输入的数据。

注意在变量名前边要加上`$`

### 执行Shell脚本

> 使用./

```bash
[root@fanl01 ~]$ cd demo  # 切换到 test.sh 所在的目录
[root@fanl01 ~]$ chmod +x ./test.sh  # 使脚本具有执行权限
[root@fanl01 ~]$ ./test.sh  # 执行脚本
[root@fanl01 ~]$ sh test.sh  # 执行脚本
```

如果没有运行权限的话，用`./`执行就会有报错

使用`.`增加 `test.sh `的执行权限，就可以正常运行了

```bash
[root@fanl01 ~]$ . ./test.sh
# 注意有空格
```

> 使用 source 命令

source 命令也可读取并在当前环境中执行脚本，同时还可返回脚本中最后一个命令的返回状态；如果没有返回值则返回 0，代表执行成功；如果未找到指定的脚本则返回 false。

```bash
[root@fanl01 ~]$ source test.sh
```

> 作为解释器参数

这种运行方式是，直接运行解释器，其参数就是 shell 脚本的文件名

```bash
[root@fanl01 ~]$ /bin/bash test.sh
```

### Shell printf 命令

printf 使用引用文本或空格分隔的参数，外面可以在 printf 中使用格式化字符串，还可以制定字符串的宽度、左右对齐方式等

语法：printf  format-string  [arguments...]

```bash
[root@fanl01 ~]$ printf "Hello, Shell\n"

[root@fanl01 ~]$ printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
[root@fanl01 ~]$ printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234 
[root@fanl01 ~]$ printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543 
[root@fanl01 ~]$ printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876 

# -表示左对齐，没有则表示右对齐
# %-4.2f 指格式化为小数，其中.2指保留2位小数。
# d: Decimal 十进制整数
# s: String 字符串 
# c: Char 字符
# f: Float 浮点
```

### Shell test 命令

Shell中的 test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。

```bash
if test $[num1] -eq $[num2]
```

## Shell语法

### Shell变量

在 Bash shell 中，每一个变量的值都是字符串，无论你给变量赋值时有没有使用引号，值都会以字符串的形式存储。

注意，赋值号`=`的周围不能有空格，这可能和你熟悉的大部分编程语言都不一样。

Shell 变量的命名规范和大部分编程语言都一样。

使用一个定义过的变量，只要在变量名前面加美元符号`$`即可

```bash
author="fanl"
echo $author
echo ${author}
```

推荐给所有变量加上花括号`{ }`，这是个良好的编程习惯。

已定义的变量，可以被重新赋值。

```bash
author="fanl"
echo ${author}
author="fanl01"
echo ${author}
```

> 单引号和双引号的区别

以单引号`' '`包围变量的值时，单引号里面是什么就输出什么，这种方式比较适合定义显示纯字符串的情况，即不希望解析变量、命令等的场景。

以双引号`" "`包围变量的值时，输出时会先解析里面的变量和命令。

> 将命令的结果赋值给变量

```bash
variable=`command`
variable=$(command)
```

> 只读变量

使用 `readonly `命令可以将变量定义为只读变量，只读变量的值不能被改变。

```bash
[root@fanl01 ~]$ x="2"
[root@fanl01 ~]$ readonly x
[root@fanl01 ~]$ x="4"
bash: x: readonly variable
```

> 删除变量

使用 `unset `命令可以删除变量

> Shell 变量的作用域

Shell 变量的作用域可以分为三种：

- 有的变量可以在当前 Shell 会话中使用，这叫做全局变量（global variable）；
- 有的变量只能在函数内部使用，这叫做局部变量（local variable）；
- 而有的变量还可以在其它 Shell 中使用，这叫做环境变量（environment variable）。

环境变量被创建时所处的 Shell 被称为父 Shell，如果在父 Shell 中再创建一个 Shell，则该 Shell 被称作子 Shell。当子 Shell 产生时，它会继承父 Shell 的环境变量为自己所用，所以说环境变量可从父 Shell 传给子 Shell。不难理解，环境变量还可以传递给孙 Shell。

使用`export`命令将它导出，那么它就在所有的子 Shell 中也有效了

> Shell位置参数

定义 Shell 函数时不能带参数，但是在调用函数时却可以传递参数，这些传递进来的参数，在函数内部就也使用$n的形式接收，例如，$1 表示第一个参数，$2 表示第二个参数，依次类推。

这种通过`$n`的形式来接收的参数，在 Shell 中称为位置参数。

```bash
#!/bin/bash

function fun(){
	echo "姓名：$1"
	echo "年纪：$2"
}
fun fanl 90
```

保存为test.sh

```bash
[root@fanl01 ~]$ ./test.sh
姓名：fanl
年纪：90
```

> Shell特殊变量

| **变量**  | **含义**                                             |
| --------- | ---------------------------------------------------- |
| $0        | 当前脚本的文件名。                                   |
| $n（n≥1） | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数 |
| $!        | 后台运行最后一个进程PID号                            |
| $#        | 传递给脚本或函数的参数个数。字符串的长度`${#xx}`     |
| $*        | 传递给脚本或函数的所有参数。**[注1]**                |
| $@        | 传递给脚本或函数的所有参数。**[注2]**                |
| $?        | 上个命令的退出状态，或函数的返回值                   |
| $$        | 当前 Shell 进程 ID                                   |

当 $* 和 ​$@ 不被双引号" "包围时，它们之间没有任何区别，都是将接收到的每个参数看做一份数据，彼此之间以空格来分隔。

但是当它们被双引号`" "`包含时，就会有区别了

- [注1]`"$*"`会将所有的参数从整体上看做一份数据，而不是把每个参数都看做一份数据。
- [注2]`"$@"`仍然将每个参数都看作一份数据，彼此之间是独立的。

```bash
#! /bin/bash

for var in "$*"
do 
	echo ${var}
done
echo "--------------"
for var in "$@"
do 
	echo ${var}	
done
```
