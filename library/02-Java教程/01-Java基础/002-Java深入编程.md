# Java 深入编程

这里使用的是JDK8

## Java的数组

### 声明数组

```java
String[] name;// 首选的方法
String name[];// 效果相同，但不是首选方法
```

数组的元素是通过索引访问的。数组索引从 0 开始，所以索引值从 0 到 length-1

遍历数组

```java
for(String e: name)
{
    System.out.println(e);
}
```

### 多维数组

```java
int a[][] = new int[2][3];
```

## 异常处理

Checked Expection：编译时异常，需要处理

Runntime Expection：运行时异常

### 自定义异常

需要继承Excption

### throws/throw关键词

```java
// 类 或者 方法
throws IOException
// 方法内部
throw new IOException();
```



## 常用的类

### Statement类

Statement 对象用于将 SQL 语句发送到数据库中。实际上有三种 Statement 对象，它们都作为在给定连接上执行 SQL 语句的包容器

1. Statement：发送简单的不带参数的、或者拼接的SQL，存在SQL注入风险
2. PreparedStatement：预编译，防止SQL注入
3. CallableStatement：存储过程

## Java集合

![集合](assets/20190417182646.png)

Java 集合框架主要包括两种类型的容器，一种是集合（Collection），存储一个元素集合，另一种是图（Map），存储键/值对映射。

Collection 接口又有 3 种子类型，List、Set 和 Queue，再下面是一些抽象类，最后是具体实现类，常用的有 ArrayList、LinkedList、HashSet、LinkedHashSet、HashMap、LinkedHashMap 等等。

> Collection 接口继承Iterable接口

Collection 是最基本的集合接口，一个 Collection 代表一组 Object，即 Collection 的元素, Java不提供直接继承自Collection的类，只提供继承于的子接口(如List和set)。

Collection 接口存储一组不唯一，无序的对象。

> List 接口继承Collection接口

List 接口存储一组不唯一，有序（插入顺序）的对象。

> Set继承Collection接口

Set 接口存储一组唯一，无序的对象。

（1）**Set和List的区别**

1. Set 接口实例存储的是无序的，不重复的数据。List 接口实例存储的是有序的，可以重复的元素。
2. Set检索效率低下，删除和插入效率高，插入和删除不会引起元素位置改变 **<实现类有HashSet,TreeSet>**
3. List和数组类似，可以动态增长，根据实际存储的数据的长度自动增长List的长度。查找元素效率高，插入删除效率低，因为会引起其他元素位置改变 **<实现类有ArrayList,LinkedList,Vector>** 

### ArrayList和LinkedList

（1）ArrayList底层是数组，新增元素的时候先判断容量，再新增元素，指定位置新增/删除元素，要移动数据，效率较慢，查询较快。每次扩容是当前的一半。

```java
int newCapacity = oldCapacity + (oldCapacity >> 1);
```

（2）LinkedList底层是双向循环链表，新增的时候将节点作为最后的一个节点，所以插入效率较快，查询较慢。

### ArrayList 和 Vector的区别

线程：ArrayList 是线程不安全，Vector是线程安全的

增长率：ArrayList增加当前size的一半，而Vector增长率可设置。

```java
int newCapacity = oldCapacity + ((capacityIncrement > 0) ? capacityIncrement : oldCapacity);
```

### HashMap 和 Hashtable

HashMap是最常用的Map，它根据键的HashCode值存储数据，根据键可以直接获取它的值，具有很快的访问速度，遍历时，取得数据的顺序是完全随机的。因为键对象不可以重复，所以HashMap最多只允许一条记录的键为Null，允许多条记录的值为Null，是非同步的。

Hashtable与HashMap类似，是HashMap的线程安全版，它支持线程的同步，即任一时刻只有一个线程能写Hashtable，因此也导致了Hashtale在写入时会比较慢，它继承自Dictionary类，不同的是它不允许记录的键或者值为null，同时效率较低。

ConcurrentHashMap线程安全，并且锁分离。ConcurrentHashMap内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个小的hash table，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。

## 相关问题

### 线程

1. 进程和线程有什么区别
2. 创建线程的几种方式和优缺点
3. 线程有哪些状态
4. 什么是死锁

### 集合⭐

1. Java集合类框架的基本接口有哪些
2. 说出ArrayList,Vector, LinkedList 的存储性能和特性？
3. .Collection 和Collections 的区别？
4. HashMap 和Hashtable 的区别
5. Arraylist 与Vector 区别？（线程，数据增长）
6. List、Map、Set 三个接口，存取元素时，各有什么特点
7. Set 里的元素是不能重复的，那么用什么方法来区分重复与否呢? 是用==还是equals()? 它们有何区别?