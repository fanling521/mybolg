# Hadoop-MapRedue

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

（2）反序列化时，需要反射调用空的构造函数，**所以必须有空的构造函数**

（3）重写序列化方法`write`和反序列方法`readFields`，顺序完全一致

（4）自定义的Bean若在key中使用，需要实现`Comparable`接口，`Shuffle`过程要求对key必须能排序

[手机流量统计的例子](https://github.com/fanling521/hadoop_demo)

## MapReduce框架原理

###  InputFormat数据输入

#### 切片与MapTask并行度决定机制

MapTask的并行度决定Map阶段的任务处理并发度，进而影响到整个Job的处理速度。

**数据**块：Block是HDFS物理上把数据分成一块一块。
**数据切片**：数据切片只是在逻辑上对输入进行分片，并不会在磁盘上将其切分成片进行存储。

- 一个Job的Map阶段的并行数是由其在提交Job时候的切片数决定的
- 每一个Split切片分配一个MapTask并行实例处理
- 默认情况下，切片大小=块大小
- 切片不考虑数据集整体，而是针对单位单独切片

![assets/20190401161710.png]()

#### Job提交流程源码和切片源码

### MapReduce工作原理

### Shuffle机制

Map方法之后，Reduce方法之前的数据处理过程称之为Shuffle。

#### Shuffle机制概述

#### Partition分区

> 默认分区

![](assets/20190401161711.png)

numReducerTasks可以在Job种设置，用户无法控制分区

> 自定义分区

（1）自定义类继承`Partitioner`，重写`getPartition()`方法

（2）在Job中设置自定义类`job.setPartitionerClass()`

（3）需要根据逻辑设置ReduceTask`job.setNumReducerTasks()`

> 注意点

- 分区号是从0开始逐一递增
- ReducerTask数量和getPartition()不匹配都会出现问题或异常

#### WritableComparable排序

MapTask和ReduceTask都会对数据按照key排序，属于默认行为，默认按照字典顺序排序，且排序方式是快速排序

对于MapTask，它会将处理的结果暂时放到环形缓冲区中，当环形缓冲区使用率达到一定阈值后，再对缓冲区中的数据进行一次快速排序，并将这些有序数据溢写到磁盘上，而当数据处理完毕后，它会对磁盘上所有文件进行归并排序。

对于ReduceTask，它从每个MapTask上远程拷贝相应的数据文件，如果文件大小超过一定阈值，则溢写磁盘上，否则存储在内存中。如果磁盘上文件数目达到一定阈值，则进行一次归并排序以生成一个更大文件；如果内存中文件大小或者数目超过一定阈值，则进行一次合并后将数据溢写到磁盘上。当所有数据拷贝完毕后，ReduceTask统一对内存和磁盘上的所有数据进行一次归并排序。

（1）原理解析

bean对象做为key传输，需要实现WritableComparable接口重写compareTo方法，就可以实现排序。

## 相关问题

（1）简述MapReduce Join的实现

（2）谈谈MapRedcue的运行流程