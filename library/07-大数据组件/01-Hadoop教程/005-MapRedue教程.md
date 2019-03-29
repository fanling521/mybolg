# MapRedue教程

## 什么是MapReduce

MapReduce是一个分布式运算程序的编程框架。它的核心功能是将用于编写的业务逻辑代码和默认组件整合成一个完整的分布式运算程序，并发运行在一个Hadoop集群上。

**优点**：

- 高容错性
- 适合PB级数据离线处理

**缺点**：

- 不适合实时计算
- 不适合流式计算

## MapReduce核心思想

- Map阶段，并发执行MapTask，互不干扰
- Reduce阶段，并发执行ReduceTask，互不干扰，但是他们依赖Map阶段的输出
- MapReduce模型只能有1个Map阶段和1个Reduce阶段，如果业务复杂，多增加MapReduce，上一个MapReduce的输出作为输入

## MapReduce编程规范

### 常用数据序列化类型

| Java 类型 | Hadoop Writable类型 |
| --------- | ------------------- |
| boolean   | BooleanWritable     |
| byte      | ByteWritable        |
| int       | IntWritable         |
| float     | FloatWritable       |
| long      | LongWritable        |
| double    | DoubleWritable      |
| String    | Text                |
| map       | MapWritable         |
| array     | ArrayWritable       |

### MapReduce编程过程

用户编写的程序分成三个部分：`Mapper`、`Reducer`和`Driver`。

#### MapReduce之Mapper

1. 用户自定义的Mapper类要继承父类Mapper
2. 自定义输入输出类型KV对
3. 业务逻辑在map方法中重写，MapTask进程对每一个<K，V>调用一次

#### MapReduce之Reducer

1. 用户自定义的Reducer类要继承父类Reducer
2. 自定义输入输出类型KV对
3. 业务逻辑在reducer方法中重写，ReduceTask进程对每一个<K，V>调用一次

#### MapReduce之Driver

相当于YARN的客户端，提交的是MP程序运行相关的参数和job对象

### Hadoop 序列化

- **序列化**：将内存中的对象转换成字节序列（或者其他输出协议）以便持久化和网络传输。

- **反序列化**：将持久化的磁盘数据或者字节序列转化成内存中的对象。

- **为什么要序列化**：方便的存储或者在网络中传输。

#### 什么是Hadoop序列化

- **为什么不用Java的序列化**：重量级序列化框架`Serializable`附带信息较多，不便于在网络中高效传输。

- **Haoop序列化的特点**：紧凑，快速，可扩展，互操作。

#### 自定义实现Writable

（1）必须实现`Writable`接口

（2）反序列化时，需要反射调用空的构造函数，所以必须有空的构造函数

（3）重写序列化方法`write`和反序列方法`readFields`，顺序完全一致

（4）自定义的Bean若在key中使用，需要实现`Comparable`接口，Shuffle过程要求对key必须能排序

[手机流量统计的例子](https://github.com/fanling521/hadoop_demo)

## MapReduce框架原理

