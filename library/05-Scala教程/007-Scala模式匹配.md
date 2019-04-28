# Scala模式匹配

Scala中的模式匹配类似于Java中的switch语法，但是更加强大。

## 基本语法介绍

```scala

  def main(args: Array[String]): Unit = {
    val oper = '-'
    val n1 = 10
    val n2 = 4
    var res = 0
    oper match {
      case '+' => res = n1 + n2
      case '-' => res = n1 - n2
      case _ => println("error")
    }
    println(res)
  }
```

`case _`都不符合的时候执行，不写且找不到会报错。

### 变量

如果在case关键字后跟变量名，那么match前表达式的值会赋给那个变量。

```scala
// 下面 case mychar 含义是 mychar = ch
case mychar => println("ok~" + mychar)
```

### 类型的匹配

```scala
case a: Int => a
case b: Map[String, Int] => "对象是一个字符串-数字的Map集合"
```

### 隐藏的变量

```scala
case _: Map[String, Int] => "对象是一个字符串-数字的Map集合"
```

### case后的值

```scala
//意思讲obj传给i，obj match{}
case i => println(i)
```

## 守卫

如果想要表达匹配某个范围的数据，就需要在模式匹配中增加条件守卫。

```scala
  def main(args: Array[String]): Unit = {
    for (c <- "+-!3") {
      var sign = 0
      var digit = 0
      c match {
        case '+' => sign = 3
        case '-' => digit = 1
        case _ if (c.toString.equals("3")) => sign = 5
        case _ => digit = 10
      }
      println(sign)
      println(digit)
    }

  }
```



