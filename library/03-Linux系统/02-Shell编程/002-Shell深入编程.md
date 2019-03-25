# Shell 深入编程

## Shell 字符串

字符串可以由单引号`' '`包围，也可以由双引号`" "`包围，也可以不用引号。它们之间是有区别的

- 单引号`' '`包围，任何字符都会原样输出，在其中使用变量是无效的，字符串中不能出现单引号
- 双引号`" "`包围，变量会被解析，字符串中可以出现双引号，只要它被转义了就行
- 字符串中不能出现空格，否则空格后边的会作为其他变量或者字符串解析

```bash
#！/bin/bash
n=12
str1=fanl,age=${n} str2="is \"good\""   # 被转义的引号，可以打印出来
str3="this_num_is${n}"                  # 引号包裹起来可以解析${n}
str4='num is ${n}'                      # 单引号，原样输出，不解析${n}
echo ${str1}
echo ${str2}
echo ${str3}
echo ${str4}
```

> 在 Shell 中获取字符串长度很简单

```bash
${#string_name}
```

> Shell字符串拼接

将两个字符串并排放在一起就能实现拼接。

```bash
#！/bin/bash
n=12
str1=fanl,age=${n} str2="is \"good\""
str3="this_num_is${n}"
str4='num is ${n}'
echo ${str1}
echo ${str2}
echo ${str3}
echo "${str4}"
str5=${str4}ceshi
echo ${str5}
echo "str5的长度${#str5}"
```

## Shell 数组

数组中可以存放多个值。Bash Shell 只支持一维数组（不支持多维数组），初始化时不需要定义数组大小。

获取数组长度的方法与获取字符串长度的方法相同，`@,*`

在 Shell 中，用括号`( )`来表示数组，数组元素之间用**空格**来分隔。由此，定义数组的一般形式为

```bash
# 空格隔开
array_name=(value1 . . . valuen)
# 也可以按照下面的定义
array_name[0]=value0
# 读取数组
${array_name[index]}
# 例子
#! /bin/bash
array=('A0' 'A1' 'A2' 1 s12d)
echo ${array[0]}
echo ${array[1]}
echo ${array[2]}
echo ${array[3]}
echo ${array[4]}
# 使用@或*可以获取数组中的所有元素
echo ${array[@]}
# 利用@或*，可以将数组扩展成列表，然后使用#来获取数组元素的个数
echo ${#array[@]}
# 如果某个元素是字符串，还可以通过指定下标的方式获得该元素的长度
echo ${#array[4]}
```

## Shell 基本运算符

bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 `awk` 和 `expr`，`expr `最常用。

`expr` 是一款表达式计算工具，使用它能完成表达式的求值操作。

此外，还支持`(())`，如`((3+2))`、`((a+b))`，无需控制空格，但是取值需要加上`$`

例如，两个数相加(**注意使用的是反单引号 ` 而不是单引号 '，而且注意空格**)：

```bash
#!/bin/bash

val=`expr 2 + 2`
echo "两数之和为 : $val"
```

注意点：

- 表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2。
- 完整的表达式要被 **\` \`** 包含，注意这个字符不是常用的单引号。
- **乘号(*)前边必须加反斜杠(\\)才能实现乘法运算**

>  支持常见的算术运算符

```bash
#!/bin/bash
a=10
b=2
# 加法
c1=`expr ${a} + ${b}`
echo "${a} + ${b}=${c1}"
# 减法
c2=`expr ${a} - ${b}`
echo "${a} - ${b}=${c2}"
# 乘法
c3=`expr ${a} \* ${b}`
echo "${a} * ${b}=${c3}"
# 除法
c4=`expr ${a} / ${b}`
echo "${a} / ${b}=${c4}"
# 取余
c4=`expr ${a} % ${b}`
echo "${a} % ${b}=${c4}"
```

> 关系运算符

| **运算符** | **说明**                       | **运算符** | **说明**                       |
| ---------- | ------------------------------ | ---------- | ------------------------------ |
| `-eq`      | 检测两个数是否相等             | `-gt`        | 检测左边的数是否大于右边的     |
| `-ne`        | 检测两个数是否不相等           | `-lt`        | 检测左边的数是否小于右边的     |
| `-ge`        | 检测左边的数是否大于等于右边的 | `-le`        | 检测左边的数是否小于等于右边的 |

> 布尔运算符

- `!`(非运算，表达式为 true 则返回 false，否则返回 true)

- `-a`(与运算，两个表达式都为 true 才返回 true)
- `-0`(或运算，有一个表达式为 true 则返回 true)

> 逻辑运算符

- `&&`逻辑的 AND
- `||` 逻辑的 OR

> 字符串运算符

| **运算符** | **说明**                        | **运算符** | **说明**                              |
| ---------- | ------------------------------- | ---------- | ------------------------------------- |
| `=`        | 检测两个字符串，相等返回 true   | `-n`       | 检测字符串长度是否为0，不为0返回 true |
| `!=`       | 检测两个字符串，不相等返回 true | `$`        | 检测字符串是否为空，不为空返回 true   |
| `-z`       | 检测字符串长度，为0返回 true    |            |                                       |

> 文件测试运算符

| 操作符      | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| -b file     | 检测文件是否是块设备文件，如果是，则返回 true。              |
| -c file     | 检测文件是否是字符设备文件，如果是，则返回 true。            |
| **-d file** | **检测文件是否是目录，如果是，则返回 true。**                |
| **-f file** | **检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。** |
| -g file     | 检测文件是否设置了 SGID 位，如果是，则返回 true。            |
| -k file     | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  |
| -p file     | 检测文件是否是有名管道，如果是，则返回 true。                |
| -u file     | 检测文件是否设置了 SUID 位，如果是，则返回 true。            |
| **-r file** | **检测文件是否可读，如果是，则返回 true。**                  |
| **-w file** | **检测文件是否可写，如果是，则返回 true。**                  |
| **-x file** | **检测文件是否可执行，如果是，则返回 true。**                |
| -s file     | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     |
| -e file     | 检测文件（包括目录）是否存在，如果是，则返回 true。          |

```bash
#!/bin/bash
file="E:\MyProject\fanl-shell\shuzu.sh"
if [ -d ${file} ]
then
	echo "ok"
else
	echo "no"
fi	
```

## Shell流程控制

### 条件语句

  **[ 和 [[ 的区别**

**区别一**：在[中使用逻辑运算符，需要使用-a（and）或者-o（or）。在[[中使用逻辑运算符，需要使用&&或者||。

**区别二**：[是shell命令，在它包围的表达式是它的命令行参数，所以串比较符>和<需要转义，否则就变成IO重定向了。[[是shell关键字，不会做命令行扩展，所以<和>不需要进行转义。但是语法相对严格，如在[中可以用引号括起操作付，[[则不行。如if [ "-z" "ab" ]。

**区别三**：[[可以做算术扩展，[则不行。如if [[ 11+1 -eq 100 ]]，而if [ 11+1 -eq 100 ]则会报错。

```bash
# if else
if [ condition ]
then
    command1
    ...
fi
# 写成一行
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi
# if else-if else
if [ condition1 ]
then
    command1
elif condition2 
then 
    command2
else
    commandN
fi
```

### for循环

```bash
# 有范围的循环
for var in item1 item2 ... itemN
do
    command1
    ...
done
# 写成一行
for var in item1 item2 ... itemN; do command1; command2… done;
# 其他形式
for((i=1;i<=5;i++))
do
    echo "这是第 $i 次调用"
done

```

### while 语句

```bash
while condition
do
    command
done
# 无限循环
while :
do
    command
done
# 或者
while true
do
    command
done
# 或者
for (( ; ; ))
```

### until 循环

condition 一般为条件表达式，如果返回值为 false，则继续执行循环体内的语句，否则跳出循环。

```bash
until condition
do
    command
done
```

### case语句

```bash
case 值 in
模式1)
    command1
    command2
    ...
    commandN
    ;;
模式2）
    command1
    command2
    ...
    commandN
    ;;
esac
# 例子
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac
```

> 跳出循环

- break命令：break命令允许跳出所有循环（终止执行后面的所有循环）。
- continue命令与break命令类似，只有一点差别，它不会跳出所有循环，仅仅跳出当前循环。

## Shell 函数

- 可以带function fun() 定义，也可以直接fun() 定义,不带任何参数[**Shell位置参数**]。
- 参数返回，可以显示加：return 返回，如果不加，将以最后一条命令运行结果，作为返回值。 return后跟数值n(0-255)

```bash
#!/bin/bash
demoFun(){
    echo "这是我的第一个 shell 函数!"
}
echo "-----函数开始执行-----"
demoFun
echo "-----函数执行完毕-----"
```

## Shell编程实例

1. if else的语句的写法，空格
2. 变量的写法
3. for循环的写法

```bash
#!/bin/bash
# 获取输入的参数，没有参数，直接退出
param_count=$#
if [ ${param_count} == 0 ];then
   echo "请输路径参数"
else
   # 获取文件名
   filename=`basename $1`
   echo "文件名为："${filename}
   # 取得绝对路径
   real_path=`cd -P $(dirname $1);pwd`
   echo "真实路径为"${real_path}
   # 获取当前登录用户
   user=`whoami`
   # 循环传输文件
   for((i=152;i<=153;i++));do
      echo "=======Hadoop-HOST${i}============"
   done
fi
```

