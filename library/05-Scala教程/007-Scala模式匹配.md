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

## 匹配

### 匹配数组

1. Array(0) 匹配只有一个元素且为0的数组
2. Array(x,y) 匹配数组有两个元素，并将两个元素赋值为x和y。当然可以依次类推Array(x,y,z) 匹配数组有3个元素的等等....
3. Array(0,_*) 匹配数组以0开始

### 匹配列表

```scala
  def main(args: Array[String]): Unit = {
    for (list <- Array(List(0), List(1, 0), List(88), List(0, 0, 0), List(1, 0, 0))) {
      val result = list match {
        case 0 :: Nil => "0" //
        case x :: y :: Nil => x + " " + y //
        case 0 :: tail => "0 ..." //结尾任意字符
        case x :: Nil => x
        case _ => "something else"
      }
      println(result)
    }
  }
```

### 匹配元组

```scala
  def main(args: Array[String]): Unit = {
    for (pair <- Array((0, 1), (1, 0), (10, 30), (1, 1), (1, 0, 2))) {
      val result = pair match {
        case (0, _) => "0 ..."
        case (y, 0) => y
        case (x, y) => (y, x)
        case _ => "other"
      }
      println(result)

    }
  }
```

### 匹配对象

1. case中对象的unapply方法(对象提取器)返回Some集合则为匹配成功
2. 返回None集合则为匹配失败

```scala
object ObjectMatch {
  def main(args: Array[String]): Unit = {
    val name = "f,a,n"
    name match {
      case Name(a, b, c) => println(a, b, c)
    }
  }
}

object Name {
    //多参数提取器使用序列
    //单参数使用unapply
  def unapplySeq(arg: String): Option[Seq[String]] = {
    if (arg.contains(",")) {
      Some(arg.split(","))
    } else {
      None
    }
  }
}
```

### 变量声明中的匹配模式

match中每一个case都可以单独提取出来。

```scala
  def main(args: Array[String]): Unit = {
    val (x, y, z) = (1, 2, "hello")
    println(x)
    val (q, r) = BigInt(99) /% 3
    println("q=" + q + ",r=" + r)

    val array = Array(1, 3, 4, 5)
    val Array(a, b, _ *) = array
    println(a, b)
  }
```

### for循环中匹配模式

```scala
  def main(args: Array[String]): Unit = {
    val map = Map(("A" -> 0), ("B" -> 1), ("V" -> 2))
    //全部的
    for ((k, v) <- map) {
      println(k, v)
    }
    //只匹配v=0的
    for ((k, 0) <- map) {
      println(k, 0)
    }
  }
```

## case class 样例类

1. 样例类仍然是类，样例类和其他类完全一样。你可以添加方法和字段，扩展它们
2. 样例类用`case`关键字进行声明，是为**模式匹配而优化**的类
3. 构造器中的每一个参数都成为**val**，除非它被显式地声明为var（不建议这样做）
4. 在样例类对应的伴生对象中提供**apply**方法让你不用new关键字就能构造出相应的对象
5. 提供`unapply`方法让模式匹配可以工作
6. 将自动生成`toString`、`equals`、`hashCode`和`copy`方法

```scala
case class Dog{}
```

