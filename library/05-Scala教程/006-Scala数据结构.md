# Scala数据结构

## 结构特点

Scala同时支持不可变集合和可变集合，不可变集合可以安全的并发访问，Scala同时支持不可变集合和可变集合，不可变集合可以安全的并发访问，Scala的集合有三大类：序列Seq(有序的,Linear Seq)、集Set、映射Map[key->value]，所有的集合都扩展自`Iterable`特质。

## 常见数据结构

### 数组

#### 数组定义声明和使用

（1）定长数组和Java中的类似，将[]改为了()

```scala
  def main(args: Array[String]): Unit = {
    var array = new Array[Int](5)
    array(0) = 0
    array(1) = 1
    array(2) = 2
    array(3) = 3
    array(4) = 4
    println("下标3号；" + array(3))
    for (i <- array) {
      print(i + "\t")
    }
    //第二种方式
    var array2 = Array(2, 3, 4, 5, 6,"name")
    print(array2(5))
  }
```

（2）不定长数组，相当于ArrayList

```scala
  def main(args: Array[String]): Unit = {
    var array = ArrayBuffer(1, 2, 3, "jack")
    //取值
    println(array(3))
    //追加
    array.append("cat")
    //删除
    array.remove(0)
  }
```

arr1.**toBuffer**  //定长数组转可变数组

arr2.**toArray**  //可变数组转定长数组

（3）多维数组的定义

```scala
var array3 = Array.ofDim[Double](3, 4)
```

#### Scala数组和JavaList互转

```scala
def main(args: Array[String]): Unit = {
  var arr = ArrayBuffer("1", "2", "3","name")
  import scala.collection.JavaConversions.bufferAsJavaList
  val jArry = new ProcessBuilder(arr)
  val jList = jArry.command()
  println(jList)
}
//转Buffer
import scala.collection.JavaConversions.asScalaBuffer
val scalaArr: mutable.Buffer[String] = jList
```

### 元组

元组也是可以理解为一个容器，可以存放各种相同或不同类型的数据。

```scala
 val tuple = (1, 2, "hello", 3.43)
```

有几个元素就是TupleN类型，上面tuple是Tuple4类型，元组中最大只能有22个元素。

（1）访问元组中的数据,可以采用顺序号`_`

```scala
println(tuple._1) //第一个元素
println(tuple._2) //第二个元素
```

（2）元组的遍历

Tuple是一个整体，遍历需要调其迭代器。

```scala
for (item <- tuple.productIterator) {
  println("item=" + item)
}
```

### 列表

Scala中的List 和Java List 不一样，在Java中List是一个接口，真正存放数据是ArrayList，而Scala的List可以直接存放数据，就是一个object，默认情况下Scala的List是不可变的，List属于序列Seq。

```scala
val list1 = List(1, 2, "3")
val list2 = Nil //空列表
```

使用索引访问元素，追加元素会产生新的列表，本身不会有变化。

（1）通过 :+ 和 +: 给list追加元素

```scala
println(list1 :+ 12) //在最后加元素
println("h" +: list1)//在最前面加元素
```

（2）符号`::`表示向集合中新建集合添加元素（集合对象一定要放置在最右边），`:::`运算符是将集合中的**每一个元素**加入到集合中，运算规则为**从右向左**

```scala
// ::
val list4=4::5::list1
// :::
val list5 = List(2,3,4):::list1 ::: Nil
```

#### ListBuffer

ListBuffer是可变的list集合，可以添加，删除元素,ListBuffer属于序列。 

（1）支持通过 :+ 和 +: 给ListBuffer追加元素

（2）`+=`表示向集合中新建ListBuffer添加元素，`++=`ListBuffer合并

### 队列

队列是一个**有序列表**，在底层可以用**数组**或是**链表**来实现，其输入和输出要遵循**先入先出的原则**，开发中通常使用可变集合中的队列。

（1）追加元素

```scala
val q1 = new mutable.Queue[Int]
q1 += 4
println(q1)
q1 ++= List(3, 4, 5)
println(q1)
```

（2）入队和出队

```scala
// 出队，队列的元素会减少
val item = q1.dequeue()
println(item + "," + q1)
// 入队
q1.enqueue(56, 78, 77)
```

（3）返回队列的元素

```scala
// 队列的第一个元素
println(q1.head)
// 队列的最后的一个元素
println(q1.last)
// 队尾元素，除了第一个的元素
println(q1.tail.tail)
```

### Map

就是只含有两个数据的元组。

```scala
  def main(args: Array[String]): Unit = {
    // 不可变，输出顺序固定
    val map1 = Map("age" -> 12, "name" -> "fanl")
    println(map1)
    // 可变，输出顺序随机
    val map2=mutable.Map("age" -> 12, "name" -> "fanl")
    // 单独创建
    val mapCount = mutable.Map[Char, Int]()
    println(map2)
  }
```

（1）对偶元组，就是只含有两个数据的元组

```scala
//对偶元组
val map3 = Map(("name", "fanl"), ("age", 12))
println(map3)
```

（2）取值

- 使用map(key)，存在则返回，不存在则抛异常，Java是null。

- contains(key)检查是否存在。
- map.get(key)，存在则返回Some()，不存在返回None
- map.getOrElse(key,DefaultValue)，存在则返回，不存在返回DefaultValue

（3）修改和删除map

- map是可变的才能修改，map(key)=value，存在则修改，不存在则新增
- 增加元素map+=(key->value)，可以增加多个，存在则更新，不存在则新增
- 使用map-=(key)，可以赋值多个，存在删除，不存在不报错

（4）遍历

```scala
    //遍历1
    for ((k, v) <- map2) {
      println("k=" + k + ",v=" + v)
    }
    //遍历2
    for (k <- map2.keys) {
      println("k=" + k)
    }
    for (v <- map2.values) {
      println("v=" + v)
    }
    //遍历3--Tuple2
    for (item <- map2) {
      for (i <- item.productIterator) {
        println(i)
      }
    }
```

### Set

数据是以哈希表的形式存放的，里面的不能包含重复数据。

（1）创建

```scala
    val set1 = Set(1, 2, 4, "hello")
    println(set1)
```

（2）新增，修改和删除

```scala
  def main(args: Array[String]): Unit = {
    val set1 = mutable.Set(1, 2, 4, "hello")
    set1.add(5)
    set1 += "world"
    set1.+=(41)
    println(set1)
    set1 -= 5
    set1.remove(4)
    println(set1)
  }
```

（3）遍历

```scala
    for (item <- set1) {
      println(item)
    }
```

## 数据结构的操作

### 映射map

map()是高阶函数，接收处理函数。

```scala
def main(args: Array[String]): Unit = {
  val list = List(2, 4, 6)
  val list2 = list.map(mult)
  println(list2)
}

def mult(n: Int): Int = {
  2 * n
}
```

### 扁平映射flatmap

将集合中的**每个元素的子元素**映射到某个函数并返回新的集合。

### 筛选filter

将符合要求的数据（筛选）放置到新的集合中

### 化简reduce

将二元函数引用于集合中的函数，有reduce（reduceLeft），reduceRight

```scala
  def main(args: Array[String]): Unit = {
    val list = List(2, 4, 6, 8, 10)
    //println(list.reduce((x, y) => x + y))
    //println(list.reduce((x, y) => sum(x, y)))
    //println(list.reduce(sum))
    println(list.reduce(_ + _))//超简化写法
  }

  def sum(x: Int, y: Int): Unit = {
    x + y
  }
```

### 折叠fold

`fold`函数将上一步返回的值作为函数的第一个参数继续传递参与运算，直到list中的所有元素被遍历，用法和reduce一致。

`(1 /: list4) (minus)` 等价`list4.foldLeft(1)(fn)`

`(list4 :\ 10) (minus)`等价`list4.foldRight(10)(fn)`

### 扫描scan

即对某个集合的所有元素做`fold`操作，但是会把产生的**所有中间结果放置于一个集合**中保存，将结果保存。

### 合并zip

当我们需要将两个集合进行 **对偶元组合并**

```scala
  def main(args: Array[String]): Unit = {
    val list1=List(1,2,3)
    val list2=List("h","e","r")
    val list3=list1.zip(list2)
    println(list3)
  }
//在前面的作为前面一个元素
//List((1,h), (2,e), (3,r))
```

### 迭代器

```scala
  def main(args: Array[String]): Unit = {
    val iter=List(1,2,4,5).iterator
    while(iter.hasNext){
      println(iter.next())
    }
  }
```

### 流

stream是一个集合，可以用于存放**无穷多个元素**，需要用到多大的区间，才会动态的生产，末尾元素遵循lazy规则。

```scala
  def main(args: Array[String]): Unit = {
    def form(n: Int): Stream[Int] = n #:: form(n + 1)

    val strem01 = form(1)
    println(strem01)//Stream(1, ?)
    println(strem01.head)//1
    println(strem01.tail)//Stream(2, ?)
  }
```

### 视图

view方法产出一个总是**被懒执行的集合**，view不会缓存数据，每次都要重新计算。

### 并行集合

`par`提高处理计算效率。

### 操作符

1.  如果想在变量名、类名等定义中使用语法关键字（保留字），可以配合反引号，val `val` = 42
2.  中置操作符：A 操作符 B 等同于 A.操作符(B)  
3. 后置操作符：A操作符 等同于 A.操作符，如果操作符定义的时候不带()则调用时不能加括号
4. 前置操作符，+、-、！、~等操作符A等同于A.unary_操作符
5. 赋值操作符，A 操作符= B 等同于 A = A 操作符 B  ，比如 A += B 等价 A = A + B