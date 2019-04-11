# Scalla简介

Scala是一门以JVM为运行环境并将面向对象和函数式编程的结合一起的静态型编程语言。

## Scala和Java以及JVM的关系

1. Java使用JDK大量类库完成开发，通过Java编译器编译成字节码，运行在JVM上
2. Scala也使用ScalaSDK完成开发，ScalaSDK封装了对Java类库的支持，通过Scala编译器编译成字节码，运行在虚拟机上

## 开始编写Scala程序

### Scala的Hello编写、编译、运行

在Hello.scala中编写

```scala
object Hello{
	def main(args:Array[String]):Unit={
		println("Hello");
	}
}
```

接下来我们使用 Scalac 命令编译它：

```shell
> scalac Hello.scala
> scala Hello
```

通过编译后会生成Hello.class 和 Hello$.class

它实际的运行方式模拟如下：

```java
public class Hello {
    public static void main(String[] args) {
        Hello$.MODULES$.main(args);
    }
}

public class Hello$ {
    public static Hello$ MODULES$;

    static {
        MODULES$=new Hello$();
    }

    public static void main(String[] args) {
        System.out.println("hello");
    }
}

```
说明：

- def 表示这是一个方法
- main 是程序的入口
- args:Array[String] 是形参，将变量写在前面，类型写在后面
- Array[String] 是字符数组类型
- :Unit= 表示没有返回值，相当于Java的void

### Scala的编程注意点

- Scala语言严格**区分大小写** 
- **类名** - 对于所有的类名的第一个字母要大写
- **方法名称** - 所有的方法名称的第一个字母用小写
- **程序文件名** - 程序文件的名称应该与对象名称完全匹配（新版本不需要了，但建议保留这种习惯）。
- **def main(args: Array[String])** - Scala程序从main()方法开始处理，这是每一个Scala程序的强制程序入口部分，但是不是最终的入口，**是被包装的入口**
- Scala是面向行的语言，语句可以用分号（;）结束或换行符,如果一行里写多个语句那么分号是需要的
- 支持直接运行`*.scala`文件

### Scala的转义符

和Java一样，不再说明。

### Scala的字符串输出

（1）类似Java的写法，通过+号连接

```scala
def main(args: Array[String]): Unit = {
  var str1: String = "i"
  var str2: String = "am"
  println(str1+str2)
}
```

（2）printf格式化输出

```scala
var name:String ="fanl"
var age:Int=12
var sal:Float=11.21f
var height:Double=180.21
printf("姓名%s，年龄%d，薪水：%.2f，身高%.3f",name,age,sal,height)
```

（3）通过$取值

```scala
var str3:String ="hello"
var str4:String="world"
println(s"第一个字段$str3，第二个字段$str4")
println(s"2年后的年龄是${age+2}")
```

### Scala的注释

和Java类似，不再说明。

