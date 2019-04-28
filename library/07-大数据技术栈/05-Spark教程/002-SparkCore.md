# SparkCore

## 什么是RDD

RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是Spark中最基本的数据抽象。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。

RDD表示只读的分区的数据集，对RDD进行改动，只能通过RDD的转换操作，由一个RDD得到一个新的RDD，新的RDD包含了从其他RDD衍生所必需的信息。RDDs之间存在依赖，RDD的执行是按照血缘关系延时计算的。如果血缘关系较长，可以通过持久化RDD来切断血缘关系。

### RDD的属性

1. 一组分区（Partition），即数据集的基本组成单位
2. 一个计算每个分区的函数
3. RDD之间的依赖关系
4. 一个Partitioner，即RDD的分片函数
5. 一个列表，存储存取每个Partition的优先位置（preferred location）。

### RDD特点

- 弹性
- 分区
- 只读：RDD是只读的，要想改变RDD中的数据，只能在现有的RDD基础上创建新的RDD，RDD的操作算子包括两类，一类叫做transformations，它是用来将RDD进行转化，构建RDD的血缘关系；另一类叫做actions，它是用来触发RDD的计算，得到RDD的相关计算结果或者将RDD保存的文件系统中
- 依赖：RDDs通过操作算子进行转换，转换得到的新RDD包含了从其他RDDs衍生所必需的信息，RDDs之间维护着这种血缘关系，也称之为依赖
- 缓存
- checkpoint：当RDD的某个分区数据失败或丢失，可以通过血缘关系重建

## RDD编程

### RDD的编程模型

### RDD的创建

#### 集合

从集合中创建RDD，Spark主要提供了两种函数：`parallelize`和`sc.makeRDD`。

#### 外部存储系统

包括本地的文件系统，还有所有Hadoop支持的数据集，比如HDFS、Cassandra、HBase等。

### RDD的转换

RDD整体上分为Value类型和Key-Value类型。

#### Value类型

#### 双Value类型

#### Key-Value类型

### RDD依赖关系

#### Lineage



