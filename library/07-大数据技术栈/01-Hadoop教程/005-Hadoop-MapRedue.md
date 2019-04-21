# Hadoop-MapRedue

## 什么是MapReduce

MapReduce是一个分布式非实时的运算程序的编程框架。

MapReduce模型只能有1个Map阶段和1个Reduce阶段，如果业务复杂，多增加MapReduce，上一个MapReduce的输出作为输入

### MapReduce编程规范

### 常用数据类型

#### Hadoop序列化

>  为什么不用Java的序列化

重量级序列化框架`Serializable`附带信息较多，不便于在网络中高效传输。
> Haoop序列化的特点

紧凑，快速，可扩展，互操作。

#### 自定义实现Writable的过程

（1）必须实现`Writable`接口

（2）反序列化时，需要反射调用空的构造函数，**所以必须有空的构造函数**

（3）重写序列化方法`write`和反序列方法`readFields`，顺序完全一致

Hadoop中除了`String`类型为`Text`，其余都是其包装类+Writable，如Long—LongWritable

## MapReduce的编程过程

用户编写的程序分成三个部分：`Mapper`、`Reducer`和配置类`Driver`。

### Mapper类的编写

1. 用户自定义的Mapper类要继承父类Mapper
2. 自定义输入输出类型KV对
3. 业务逻辑在map方法中重写，MapTask进程对每一个<K，V>调用一次

### Reducer类的编写

1. 用户自定义的Reducer类要继承父类Reducer
2. 自定义输入输出类型KV对
3. 业务逻辑在reducer方法中重写，ReduceTask进程对每一个<K，V>调用一次

### 配置类的编写

相当于YARN的客户端，提交的是MP程序运行相关的参数和job对象

[手机流量统计的例子](https://github.com/fanling521/hadoop_demo)

## MapReduce框架原理⭐

### MapReduce工作流程

整个过程可以概况为输入->map->shuffle->reduce-输出。

输入文件会被切分成多个块，每一块都有一个mapTask，map阶段的输出结果会先写到环形缓冲区，然后由缓冲区写到磁盘上。默认的缓冲区大小是100M，溢出的百分比是0.8。从缓冲区写到磁盘的时候，会进行分区并排序，分区指的是某个key应该进入到哪个分区，同一分区中的key会进行排序，如果定义了Combiner的话，也会进行combine操作，开始reduce之前，先要从分区中抓取数据，在抓取的过程中伴随着排序、合并，最后输出。

### Shuffle机制

Map方法之后，Reduce方法之前的数据处理过程称之为Shuffle。
在Map端的shuffle过程是对Map的结果进行分区（partition）、排序（sort）和分割（spill），然后将属于同一个划分的输出合并在一起。

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

#### Combiner合并

Combiner是在每一个MapTask节点运行，而Reduce是接收整个Map输出。其意义就是对每一个MapTask的输出进行局部汇总，减少网络传输量。

### MapTask工作机制

1. **Read阶段**：MapTask通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。
2. **Map阶段**：该节点主要是将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。
3. **Collect收集阶段**：在用户编写map()函数中，当数据处理完成后，一般会调用`OutputCollector.collect()`输出结果。在该函数内部，它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。
4. **Spill阶段**：即“溢写”，当环形缓冲区满后，MapReduce会将数据写到本地磁盘上，生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。

### ReduceTask工作机制

1. **Copy阶段**：ReduceTask从各个MapTask上远程拷贝一片数据，并针对某一片数据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。
2. **Merge阶段**：在远程拷贝数据的同时，ReduceTask启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。
3. **Sort阶段**：按照MapReduce语义，用户编写reduce()函数输入数据是按key进行聚集的一组数据。为了将key相同的数据聚在一起，Hadoop采用了基于排序的策略。由于各个MapTask已经实现对自己的处理结果进行了局部排序，因此，ReduceTask只需对所有数据进行一次归并排序即可。
4. **Reduce阶段**：reduce()函数将计算结果写到HDFS上。

### Join的应用

#### Reduce Join

在map阶段， 把关键字作为key输出，并在value中标记出数据是来自data1还是data2。因为在shuffle阶段已经自然按key分组，reduce阶段，判断每一个value是来自data1还是data2，在内部分成2组，做集合的乘积。

#### Map Join

两份数据中，如果有一份数据比较小，小数据全部加载到内存，按关键字建立索引。大数据文件作为map的输入文件，对map()函数每一对输入，都能够方便地和已加载到内存的小数据进行连接。把连接结果按key输出，经过shuffle阶段，reduce端得到的就是已经按key分组的，并且连接好了的数据。

这种方法，要使用Hadoop中的`DistributedCache`把小数据分布到各个计算节点，每个map节点都要把小数据库加载到内存，按关键字建立索引。

## Hadoop压缩

压缩能够有效减少存储的字节数，提高了传输效率，是一种优化策略。

压缩的基本原则：

（1）运算密集型的少用压缩

（2）IO密集型的多用压缩

支持的压缩：

（1）Gzip压缩

（2）Bzip2压缩

（3）Lzo压缩

（4）Snappy压缩

## MapReduce的优化

（1）在输入阶段合并小文件，小文件会增加Map任务数，导致MR变慢

（2）Map阶段优化[Shffle阶段]：减少溢写和合并的次数，提高环形缓冲区的阈(yu)值上限，参数为`io.sort.mb`和`sort.spill.percent`

（3）Reudce阶段：合理设置Map和Reduce数，太少，增加Task等待，太多，导致竞争资源，减少使用Reduce

（4）使用压缩，采用SeqenceFile

### 数据倾斜

（1）自定义分区

（2）采用Combine合并

（3）采用Map Join，尽量避免使用Reduce Join

## 相关问题

（1）简述MapReduce Join的实现

（2）谈谈MapRedcue的运行流程，从输入-map-shffle-reduce-输出

（3）Hadoop的压缩有哪些？

（4）MapReduce过程中Shffle的优化，从环形缓冲区说起

（5）MapReduce怎么处理数据倾斜的？

（6）MapReduce有哪些优化点，从各个阶段说，输入-map-reduce

