# Scala变量和运算符

## Scala的变量

变量是程序的基本组成单位。

### 变量的使用

```scala
var|val 变量名:变量类型=初始值
```

var 修饰的变量是可以修改的，而val修饰的变量是不可修改的，相当于Java中的final修饰符。

### 变量的注意事项

变量类型可不写，编译器自行推导，即类型推导，IDEA编辑器也可提示，或者**isInstanceOf[Int]**，类型确定就不能修改，Scala是强数据型语言。

### 数据类型

Scala 与 Java有着相同的数据类型，但是Scala的数据类型**都是对象**，数据类型分为两大类：**AnyVal**和**AnyRef**

下表列出了 Scala 支持的数据类型：

| **数据类型** | **描述**                                 | 数据类型 | 描述               |
| :----------- | ---------------------------------------- | -------- | ------------------ |
| Byte         | 8位有符号补码整数                        | Short    | 16位有符号补码整数 |
| Int          | 32位有符号补码整数                       | Long     | 64位有符号补码整数 |
| Float        | 32 位                                    | Double   | 64 位              |
| Char         | 16位无符号Unicode字符                    | String   |                    |
| Boolean      | true或false                              |          |                    |
| Unit         | 表示无值                                 | Null     | null 或空引用      |
| Nothing      | 它是任何其他类型的子类型                 | Any      | 其他类的超类       |
| AnyRef       | Scala里所有引用类(reference class)的基类 |          |                    |

![数据类型](assets/20190402192632.png)

- 在Scala中，Any是一切类的父类
- 在Scala中，Nothing是一切类的子类
- 在Scala中，Null是null，只有一个值null，是AnyRef的子类

### 类型说明

其他的类型和Java是一样的，这里就不再说明了。

#### Char说明

char赋值整型数字，输出对应的unicode编码，包含了ASCII码。

```scala
def main(args: Array[String]): Unit = {
    var char1:Char=97
    println(char1)
  }
```

1. 把一个计算的值赋值给一个变量，编译器会进行类型转换和范围判断
2. 把常量值赋值给变量，编译器只判断范围，所以Char只能接收int的常量值。

```scala
  def main(args: Array[String]): Unit = {
    var char1:Char=97//正常
    var char2:Char='a'+1//报错
    var char3:Char=98+1//报错
  }
```

#### 自动类型转换

Scala进行运算和赋值时候，精度小的类型自动转换成精度大的类型。

容量排序：

![](assets/20190320192444.png)

#### 强制类型转换

将精度高的转为精度小的类型，使用强制转换函数，**但是会造成精度降低和溢出**。

#### 值类型和String的转换

##### 基本类型转String

基本类型+“”即可完成转换

```scala
var i:Int=1
println(i+"")
```

##### String转基本类型

通过String.toXX即可，但是要确保字符串是数字且能预期转成对应的类型。

```scala
val str1:String="12"
str1.toInt  //ok
val str2:String="12.5"
str2.toInt  //error
str2.toDouble //ok
```

### 标识符

```scala
var ++ :String='hello' //ok
var _q:String="helllo"//error 
//操作符要作为标识符必须后续也跟上操作符
```

关键词也能作为变量值，但是必须加上``

```scala
var `true`:String="12"//ok
```

Int，Float 不是保留字，而是预定义标识符，也可以作为变量名，但是不推荐

```scala
var Int:Int=12
```

_不能单独使用

## Scala的运算符

### 算数运算符

和Java一样的，特别的如下：

```scala
var r1:Int=10/3 //保留整数部---3
var r1:Double=10/3 //Int 3转DOuble----3.0
```

取模运算：a%b=a - a/b * b

num+=1替代num++；num1=1替代num–-

### 关系运算符

和Java一样

### 逻辑运算符

和Java一样

### 赋值运算符

和Java一样

### 位运算

和Java一样

### 特殊说明

📍取消了三目运算符，Scala改为了if else

```scala
var res:Int=if(5>4) 5 else 4
```

### 运算符优先级

和Java是一样的，和其他语言也是一样的。