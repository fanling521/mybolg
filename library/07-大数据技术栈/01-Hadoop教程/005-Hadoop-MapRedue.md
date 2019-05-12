# Hadoop-MapRedue

## 什么是MapReduce

MapReduce是一个分布式非实时的运算程序的编程框架。

MapReduce由map()和reduce()两个阶段组成，这两个函数的形参是`key-value`，MapReduce模型只能有1个Map阶段和1个Reduce阶段，如果业务复杂，多增加MapReduce，上一个MapReduce的输出作为输入。

### MapReduce编程规范

#### 常用的数据类型

Hadoop框架是在Java的基础上进行开发的，Java的数据类型在Hadoop中也能使用，但是具有一定的局限性，为了提高Hadoop对数据的处理能力，增加了Hadoop特有的数据类型，这些数据类型通过实现接口Writeable接口。如Boolean-BooleanWritable，String-Text。

#### Hadoop的序列化

为什么不用Java的序列化？重量级序列化框架`Serializable`附带信息较多，不便于在网络中高效传输，Hadoop序列化机制中，可以复用对象，这样就减少了Java对象的分配和回收，提高了应用效率。

#### 自定义Writable类型

（1）编写一个类实现`Writable`接口

（2）反序列化时，需要反射调用空的构造函数，**所以必须有空的构造函数**

（3）重写序列化方法`write`和反序列方法`readFields`，顺序要完全一致

```java
public class PhoneWritable implements Writable {
    public PhoneWritable() {
        super();
    }
   @Override
    public void write(DataOutput out) throws IOException {
        out.writeLong(xx);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        this.xx = in.readLong();
    }
}
```

## MapReduce的原理⭐

整个过程可以概况为**输入**->**Mapper**->**Shuffle**->**Reducer**-**输出**。

### 数据分片

*数据分片*是逻辑上的分片，大小计算为`Math.max(minSize,Math.min(taskNums,blockSize))`

### Mapper执行过程

每个Mapper任务是一个Java进程，它会读取`HDFS`上的文件，解析成很多的键值对，经过`map`方法处理后转换为很多的键值对再输出。

第一阶段：将输入的文件按照一定的条件分片，分片的大小默认是**128M**，每个分片由一个`Mapper`进程处理。

第二阶段：将输入片中的数据按照规则解析成键值对，如将一行文本解析，`K`为字节起始位置，`V`为本行文本。

第三阶段：调用`map`方法，将第二段的输出按照规则解析成更多的键值对。

第四阶段：将第三阶段的键值按照规则进行分区，如按照省份，每一个分区对应一个`Reducer`任务。

第五阶段：对每个分区的数据进行排序，第六阶段可选`reduce`归纳处理过程。

### Reducer执行过程

第一阶段：主动复制Mapper任务中的输出。

第二阶段：合并，排序。

第三阶段：排序后的数据再进行`reduce`归纳处理，最终写入HDFS文件。

### Shuffle过程

map方法之后，reduce方法之前的数据处理过程称之为`Shuffle`，分为**Mapper端的Shuffle**和**Reducer端Shuffle**。
Mapper端产生的数据输出结果会先写到环形缓冲区，默认的缓冲区大小是100M，溢出的百分比是0.8，然后由缓冲区写到磁盘上。

从缓冲区写到磁盘的时候，会进行分区并排序，分区指的是某个key应该进入到哪个分区，同一分区中的key会进行排序，如果定义了Combiner的话，也会进行combine操作，开始reduce之前，先要从分区中抓取数据，在抓取的过程中伴随着排序、合并，最后输出。

Reducer端Shuffle对输出数据进行归并排序，将相同key的数据排序集中到一起。

#### Partition分区

按照Key值将中间结果分成R份，其中每份都有一个Reduce去负责，可以通过job.setPartitionerClass()方法进行设置，默认的使用hashPartitioner类。

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

合并相同的key键，通过job.setCombinerClass()方法设置

Combiner是在每一个MapTask节点运行，而Reduce是接收整个Map输出。其意义就是对每一个MapTask的输出进行局部汇总，减少网络传输量。

### MapReduce接口类

#### 输入的处理类

1. FileInputFormat：是所有作为数据源的InputFormat实现的基类，实现了对输入文件计算splits的方法，只划分比Block大的文件，这就是为何处理大文件效率比小文件效率高的原因
2. TextInputFormat：处理普通文本，默认以\n为换行
3. CombineFileInputFormat：适用于大量小文件
4. KeyValueTextInputFormat：适合输入数据的每一行为两列，制表符分割的文件
5. NLineInputFormat：控制每个split的行数

#### 输出的处理类

1. OutputFormat：能够将k-v对写入特定格式的文件中
2. TextOutputFormat：k-v中间值用制表符分割
3. 其他格式

### Join的应用（详细见Hive）

#### Reduce Join

在map阶段， 把关键字作为key输出，并在value中标记出数据是来自data1还是data2。因为在shuffle阶段已经自然按key分组，reduce阶段，判断每一个value是来自data1还是data2，在内部分成2组，做集合的乘积。

#### Map Join

两份数据中，如果有一份数据比较小，小数据全部加载到内存，按关键字建立索引。大数据文件作为map的输入文件，对map()函数每一对输入，都能够方便地和已加载到内存的小数据进行连接。把连接结果按key输出，经过shuffle阶段，reduce端得到的就是已经按key分组的，并且连接好了的数据。

这种方法，要使用Hadoop中的`DistributedCache`把小数据分布到各个计算节点，每个map节点都要把小数据库加载到内存，按关键字建立索引。

## Hadoop的压缩

压缩能够有效减少存储的字节数，提高了传输效率，是一种优化策略。

压缩的基本原则：

（1）运算密集型的少用压缩

（2）IO密集型的多用压缩

支持的压缩：

（1）Gzip压缩，**不支持split**

（2）Bzip2压缩

（3）Lzo压缩

（4）Snappy压缩，**不支持split**

## MapReduce的优化

（1）在输入阶段合并小文件，小文件会增加Map任务数，导致MR变慢

（2）Map阶段优化[Shffle阶段]：减少溢写和合并的次数，提高环形缓冲区的阈(yu)值上限，参数为`io.sort.mb`和`sort.spill.percent`

（3）Reudce阶段：合理设置Map和Reduce数，太少，增加Task等待，太多，导致竞争资源，减少使用Reduce

（4）使用压缩，采用SeqenceFile

### 数据倾斜（详细见Hive）

#### 数据倾斜的原因（hive）

**情形1**：其中一个表较小，但是key集中
**后果1**：分发到某一个或几个Reduce上的数据远高于平均值

**情形2**：大表与大表，但是分桶的判断字段0值或空值过多
**后果2**：这些空值都由一个reduce处理，非常慢

#### 解决数据倾斜

（1）自定义分区

（2）采用Combine合并

（3）采用Map Join，尽量避免使用Reduce Join

## 相关问题

（1）简述MapReduce Join的实现

（2）谈谈MapRedcue的运行流程，从输入-map-shffle-reduce-输出

（3）Hadoop的压缩有哪些？

（4）MapReduce过程中Shffle的优化，（从环形缓冲区说起）

（5）MapReduce怎么处理数据倾斜的？

（6）MapReduce有哪些优化点，从各个阶段说说

（7）HDFS 有一个 `gzip` 文件大小 75MB，客户端设置 Block 大小为 64MB。当运行MapReduce 任务读取该文件时`input split`大小为多少