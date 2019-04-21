# Scala面向对象

## 基础篇

### 类与对象

此描述和Java中是一样的

Scala语言是纯粹的面向对象的，在Scala中一切皆对象。

#### 类的定义

注意事项：Scala中类不需要声明为public，默认为public，一个文件中可以有多个类。

```scala
class Xxx{
    
}
// 使用
val|var xx:类型=new 类型()
val|var xx:Xxx=new Xxx()
// 可以省略类型，但是多态适合需要指明。把子类赋值给父类对象。
```

#### 对象

大体的思和Java是一样

对象的创建过程：

1. 加载类的信息
2. 在内存开辟空间
3. 使用父类的构造器开始初始化
4. 使用主/辅助构造器为属性初始化
5. 将此对象的地址赋值给变量

#### 方法

Scala类的方法就是函数

#### 构造器

Scala的构造器和Java一样构造对象的适合需要调用构造方法，也可以有多个构造方法，但是在写法和逻辑上有一定的区别，Scala的构造器分为主构造器和辅助构造器。

##### 主构造器

主构造器直接放在类名后面，如果需要设置为私有，可以添加private，主构造器会执行类中所有的语句，即构造器也是函数。没有参数的主构造器可以省略()

```scala
class Dog(){
    ......
}

class Dog(str:String){
    ......
}

class Dog private(){
    ......
}
```

##### 辅助构造器📍

辅助构造器的名称就是this，都会直接或者间接调用主构造器，this()需要放在第一行。

```scala
def this(name:String){
    this()
    .......
}
```

#### 属性

Scala类中的主构造器的形参没有使用任何修饰符，那么这个形参就是局部变量，实例化对象无法访问到的。

如果使用val 修饰，这个形参就是类的私有的只读属性，而用var修饰则是私有的可读可写的属性。

```scala
object ClassDemo {

  def main(args: Array[String]): Unit = {
    val dog = new Dog("tom")
    dog.DogName
    dog.name
  }

  class Dog(val name: String) {
    var DogName = name
  }
}
```

注意：给某个属性添加@BeanProperty会自动生产get/set方法。

```scala
  class Cat{
    @BeanProperty
    var catName:String=""
  }
```

### 包

Scala和Java的包管理是一样的，但是其更加复杂。

源文件和指定包路径可以不匹配，编译后会按照源文件的包路径存放。

```scala
// com.fanling
package com.fanling{
    // 一个源文件多个package
    // com.fanling.demo1
	package demo1{
    
	}
	// com.fanling.demo2
    package demo2{
    
	}
}
```

#### 包对象

```scala
package com.fanling {
    
  package object scala {
    var name = "fanl"
  }
    
  package scala {
    class App {
      def main(args: Array[String]): Unit = {
        val m = new My
        m.test()
      }
    }
    class My {
      def test(): Unit = {
        println(name)
      }
    }
  }
}

```

注意事项：

1. 包对象需要在父类包中定义
2. private修饰包对象，只能在类内部和伴生对象中使用
3. protected 修饰的只能子类中使用
4. 可以定义访问范围，如 `private[com] val name=“name”`

#### 包的引入📍

1. 包的引入可以出现在任何地方，缩小包的作用范围，提高效率
2. 所有的包的通配符是`_`
3. 可以用选择器选择引入的类，如`import scala.collection.mutable.{HashMap,HashSet}`
4. 可以重命名引用的类，防止出现歧义，如`import java.util.{HashMap=>JavaHashMap,List}`
5. 不想使用某个类，可以隐藏掉，如`import java.util.{HashMap=>_,_}`

### 面向对象的特征

#### 封装

封装就是抽象的数据和对数据的操作封装在一起，数据被保护在内部，只能通过被授权的操作才能对数据进行访问。

#### 继承

和Java一样是单继承，重写需要用`override`，调用超类方法需要用`super`

##### 类型检查和转换

要测试某个对象是否属于某个给定的类，可以用`isInstanceOf[]`方法，`asInstanceOf[]`方法将引用转换为子类的引用，用`classOf[]`获取对象的类名。

##### 超类的构造

只有子类的主构造器才能调用父类的构造器。

##### 覆写字段📍

Scala中可以通过使用override覆写父类的字段，这和Java中是不同的

#### 多态



