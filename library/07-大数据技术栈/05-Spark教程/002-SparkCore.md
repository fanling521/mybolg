# SparkCore

## 什么是RDD

`RDD（Resilient Distributed Dataset）`叫做弹性分布式数据集，是Spark中最基本的数据抽象。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。

RDD表示只读的分区的数据集，对RDD进行改动，只能通过RDD的转换操作，由一个RDD得到一个新的RDD，新的RDD包含了从其他RDD衍生所必需的信息。

RDDs之间存在依赖，RDD的执行是按照血缘关系延时计算的。如果血缘关系较长，可以通过持久化RDD来切断血缘关系。

### RDD的属性

1. 一组分区（`Partition`），即数据集的基本组成单位
2. 一个计算每个分区的函数
3. RDD之间的依赖关系
4. 一个`Partitioner`，即RDD的分片函数
5. 一个列表，存储存取每个Partition的优先位置（preferred location）。

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

（1）集合的遍历map,mapPartitons,mapPartitonsWithIndex,flatMap

```scala
val rdd=sc.parallelize(1 to 10)
//map每次处理一条数据
val rdd2=rdd.map(_*2)
rdd2.collect
// mapPartitions(func) 独立运行在Spark的分区上，每次处理1个分区的数据，处理完才释放内存
val rdd3=rdd.mapPartitios(x=>x.map(_*2))
rdd3.collect
// mapPartitionsWithIndex 和mapPartitions类似，但是带有一个索引值
val rdd4=rdd.mapPartitionsWithIndex((index,item)=>item.map((index,_)))
// flatMap 返回0到多个元素
val rdd5=rdd.flatMap(1 to _)
```

（2）分组glom,groupBy

```scala
// glom是将一个分区形成一个数组，形成新的RDD
// parallelize是怎么分区的
val rdd=sc.parallelize(1 to 10,4)
rdd.glom.collect
// groupBy 按照函数的返回值进行分组
 val rdd2=sc.parallelize(1 to 10)
rdd2.groupBy(_%2).collect
```

（3）过滤filter

```scala
//经过转换形成新的RDD
val rdd=sc.parallelize(Array("h1","h2","h3","hr","div"))
//过滤有h的字符
val rdd2=rdd.filter(_.contanins("h"))
rdd2.collect
```

（4）抽样sample

```scala
//sample(是否放回,抽样比例，随机数)
val rdd=sc.parallelize(1 to 10)
val rdd2=rdd.sample(true,0.2,10)
```

（7）去重distinct

```scala
//distince(numTasks) 默认只有8个并行任务，可以设置
val rdd=sc.parallelize(List(1,2,11,2,1,2,33,22,11,21,1,33))
rdd.distinct.collect
```

（8）缩减分区，coalesce和repartition

```scala
//大数据集过滤后，缩减分区，提高小数据集的效率，可选是否进行shuffle
// coalesce(4,shuffle=true)=repartiton
val rdd=sc.parallelize(1 to 10,4)
rdd.partitions.size
//4
val rdd2=rdd.coalesce(2)
rdd2.partitions.size
//2
```

（9）排序sortBy

```scala
// 先处理结果，再按照结果排序，默认正序
val rdd=sc.parallelize(List(2,1,4,3))
// 按照自身排序
rdd.sortBy(x+>x).collect
// 按照与3的余数排序
rdd.sortBy(_%2).collect
// 降序
rdd.sortBy(_%2,ascending=false).collect
```

（10）管道pipe

```scala
// 针对每个分区都执行一个shell脚本
// #! /bin/bash
// while read LINE;do
//    echo ">>>"${LINE}
// done
val rdd=sc.parallelize(List("hello","world"))
rdd.pipe("/home/fanl/data/spark-pipe.sh").collect
```

#### 双Value类型

（1）union并集，合并

```scala
val rdd1=sc.parallelize(1 to 5)
val rdd2=sc.parallelize(5 to 10)
val rdd3=rdd1.union(rdd2)
rdd3.collect
```

（2）subtract求差集

```scala
val rdd1=sc.parallelize(1 to 5)
val rdd2=sc.parallelize(5 to 10)
// 计算的rdd1的和rdd2的差值，返回的是rdd1中和rdd2中不重复的元素
rdd1.subtract(rdd2).collect
// 返回rdd2中和rdd1中不重复的元素
```

（3）intersection求交集

```scala
val rdd1=sc.parallelize(1 to 5)
val rdd2=sc.parallelize(5 to 10)
rdd1.intersection(rdd2).collect
res0: Array[Int] = Array(5)
```

（4）cartesian笛卡尔积

**注意**：尽量避免使用笛卡尔积

（5）zip将两个RDD组合成Key/Value，两个分区，元素数量需要一致，否则抛出异常

```scala
val rdd1=sc.parallelize(Array(1,2,3,4,5)) 
val rdd2=sc.parallelize(Array("a","b","c","d","e"))
rdd1.zip(rdd2).collect
res0: Array[(Int, String)] = Array((1,a), (2,b), (3,c), (4,d), (5,e))
```

#### Key-Value类型

（1）partitionBy分区，若两次分区一致则不分区，若不一样，产生shuffle过程，进行分区

```scala
val rdd1=sc.parallelize(Array(("a",1),("b",3),("c",5),("d",7)),4)
rdd1.partitions.size
//hash分区
val rdd2=rdd1.partitionBy(new org.apache.spark.HashPartitioner(2))
rdd2.partitions.size
```

（2）reduceByKey(func,numtask)，将相同key的值聚合到一起，在shuffle前有combine操作

```scala
val rdd = sc.parallelize(Array(("a", 1), ("b", 3), ("a", 3)))
val rdd2 = rdd.reduceByKey(_ + _)
```

（3）groupByKey对key操作，将相同key聚合到一个Seq中

```scala
val rdd1 = sc.parallelize(Array("spark","hadoop","spark","hive","hadoop"))
// 遍历成元组
val rdd2=rdd1.map((_,1))
// 将相同key聚合到Seq中
val rdd3=rdd2.groupByKey
// 打印序列
rdd3.collect
//res5: Array[(String, Iterable[Int])] = Array((hive,CompactBuffer(1)), (spark,CompactBuffer(1, 1)), (hadoop,CompactBuffer(1, 1)))
//再将元组遍历组合
rdd3.map(x=>(x._1,x._2.sum)).collect
```

（4）aggregateByKey(初始值)(SeqOp,ComOp)，foldByKey(初始值)(SeqOp)

```scala
//aggregateByKey按key将value进行分组合并，合并时，将每个value和初始值作为seq函数的参数，进行计算，返回的结果作为一个新的kv对，然后再将结果按照key进行合并，最后将每个分组的value传递给combine函数进行计算
//（1）zeroValue：给每一个分区中的每一个key一个初始值；
//（2）seqOp：函数用于在每一个分区中用初始值逐步迭代value；
//（3）combOp：函数用于合并每个分区中的结果。

//foldByKey的作用：aggregateByKey的简化操作，seqop和combop相同
val rdd = sc.parallelize(List(("a",3),("a",2),("c",4),("b",3),("c",6),("c",8)),2)
//赋初值0，将每个相同的key的value和初值比较，选择最大的作为新的输出，然后进行合并相加
rdd.aggregateByKey(0)(math.max(_,_),_+_).collect
res0: Array[(String, Int)] = Array((b,3), (a,3), (c,12))
//当seqOp和combOp一样时候使用foldByKey
rdd.foldByKey(0)(_+_).collect
res1: Array[(String, Int)] = Array((b,3), (a,5), (c,18))
```

（5）combineByKey，针对相同K，将V合并成一个集合

```scala
rdd.combineByKey().collect
```

（6）sortByKey([ascending],[numTasks])在一个(K,V)的RDD上调用，K必须实现Ordered接口，返回一个按照key进行排序的(K,V)的RDD

```scala
val rdd = sc.parallelize(Array((3,"aa"),(6,"cc"),(2,"bb"),(1,"dd")))
//正序
rdd.sortByKey(true).collect()
//res3: Array[(Int, String)] = Array((1,dd), (2,bb), (3,aa), (6,cc))
```

（7）mapValues，只针对value操作

```scala
val rdd = sc.parallelize(Array((1,"a"),(1,"d"),(2,"b"),(3,"c")))
rdd.mapValues(_+"|||").collect()
//res4: Array[(Int, String)] = Array((1,a|||), (1,d|||), (2,b|||), (3,c|||))
```

（8）join在类型为(K,V)和(K,W)的RDD上调用，返回一个相同key对应的所有元素对在一起的(K,(V,W))的RDD

```scala
val rdd = sc.parallelize(Array((1,"a"),(2,"b"),(3,"c")))
val rdd1 = sc.parallelize(Array((1,4),(2,5),(3,6)))
rdd.join(rdd1).collect()
//res5: Array[(Int, (String, Int))] = Array((1,(a,4)), (2,(b,5)), (3,(c,6)))
```

（9）cogroup(otherDataset,[numTasks])在类型为(K,V)和(K,W)的RDD上调用，返回一个(K,(Iterable\<V>,Iterable\<W>))类型的RDD

```scala
val rdd = sc.parallelize(Array((1,"a"),(2,"b"),(3,"c")))
val rdd1 = sc.parallelize(Array((1,4),(2,5),(3,6)))
rdd.cogroup(rdd1).collect()
//res6: Array[(Int, (Iterable[String], Iterable[Int]))] = Array((1,(CompactBuffer(a),CompactBuffer(4))), (2,(CompactBuffer(b),CompactBuffer(5))), (3,(CompactBuffer(c),CompactBuffer(6))))
```

### Action

（1）reduce(func)，通过func函数聚集RDD中的所有元素，先聚合分区内数据，再聚合分区间数据。

（2）collect()，在驱动程序中，以数组的形式返回数据集的所有元素。

（3）first()，返回RDD中的第一个元素

（4）take(n)，返回一个由RDD的前n个元素组成的数组

（5）takeOrdered(n)，返回该RDD排序后的前n个元素组成的数组

（6）aggregate和fold，aggregate函数将每个分区里面的元素通过seqOp和初始值进行聚合，然后用combine函数将每个分区的结果和初始值(zeroValue)进行combine操作。这个函数最终返回的类型不需要和RDD中元素类型一致

（7）saveAsTextFile(path)，将数据集的元素以textfile的形式保存到HDFS文件系统或者其他支持的文件系统

（8）saveAsSequenceFile(path) ，将数据集中的元素以Hadoop sequencefile的格式保存到指定的目录

（9）saveAsObjectFile(path) ，用于将RDD中的元素序列化成对象，存储到文件中

（10）countByKey()，针对(K,V)类型的RDD，返回一个(K,Int)的map，表示每一个key对应的元素个数

```scala
val rdd = sc.parallelize(List((1,3),(1,2),(1,4),(2,3),(3,6),(3,8)),3)
rdd.countByKey
```

（11）foreach(func)，在数据集的每一个元素上，运行函数func进行更新

在实际开发中我们往往需要自己定义一些对于RDD的操作，那么此时需要主要的是，初始化工作是在Driver端进行的，而实际运行程序是在Executor端进行的，这就涉及到了跨进程通信，是需要序列化的，使类继承`scala.Serializable`即可。

### RDD依赖关系

#### Lineage

RDD只支持粗粒度转换，即在大量记录上执行的单个操作。将创建RDD的一系列Lineage（血统）记录下来，以便恢复丢失的分区。

RDD的Lineage会记录RDD的元数据信息和转换行为，当该RDD的部分分区数据丢失时，它可以根据这些信息来重新运算和恢复丢失的数据分区。

#### 窄依赖

窄依赖指的是每一个父RDD的Partition最多被子RDD的一个Partition使用。

#### 宽依赖

宽依赖指的是多个子RDD的Partition会依赖同一个父RDD的Partition，会引起shuffle。

#### DAG

`DAG(Directed Acyclic Graph)`叫做有向无环图，原始的RDD通过一系列的转换就就形成了DAG，根据RDD之间的依赖关系的不同将DAG划分成不同的Stage，对于窄依赖，partition的转换处理在Stage中完成计算。对于宽依赖，由于有Shuffle的存在，只能在parent RDD处理完成后，才能开始接下来的计算，因此宽依赖是划分Stage的依据。

#### 任务划分

RDD任务切分中间分为：Application、Job、Stage和Task。

1. **Application**：初始化一个SparkContext即生成一个Application
2. **Job**：一个Action算子就会生成一个Job
3. **Stage**：根据RDD之间的依赖关系的不同将Job划分成不同的Stage，遇到一个宽依赖则划分一个Stage
4. **Task**：Stage是一个TaskSet，将Stage划分的结果发送到不同的Executor执行即为一个Task

### RDD缓存

RDD通过persist方法或cache方法可以将前面的计算结果缓存，默认情况下persist() 会把数据以序列化的形式缓存在JVM 的堆空间中。

### RDD CheckPoint
Spark中对于数据的保存除了持久化操作之外，还提供了一种检查点的机制，检查点（本质是通过将RDD写入Disk做检查点）是为了通过`lineage`做容错的辅助，`lineage`过长会造成容错成本过高，这样就不如在中间阶段做检查点容错，如果之后有节点出现问题而丢失分区，从做检查点的RDD开始重做`Lineage`，就会减少开销。检查点通过将数据写入到HDFS文件系统实现了RDD的检查点功能。

为当前RDD设置检查点。该函数将会创建一个二进制的文件，并存储到checkpoint目录中，该目录是用`setCheckpointDir()`设置的。在checkpoint的过程中，该RDD的所有依赖于父RDD中的信息将全部被移除。对RDD进行checkpoint操作并不会马上被执行，必须执行Action操作才能触发。

```scala
sc.setCheckpointDir("hdfs://hadoop01:8020/checkpoint")
val rdd = sc.parallelize(Array("hello"))
val ch = rdd.map(_+System.currentTimeMillis)
ch.checkpoint
ch.collect
```

## 键值对RDD数据分区

Spark目前支持`Hash`分区和`Range`分区，用户也可以自定义分区，Hash分区为当前的默认分区，Spark中分区器直接决定了RDD中分区的个数、RDD中每条数据经过Shuffle过程属于哪个分区和Reduce的个数。

只有`Key-Value`类型的RDD才有分区的，非Key-Value类型的RDD分区的值是None。

#### 获取分区

```scala
pairs.partitioner
```

#### Hash分区

`HashPartitioner`分区的原理：对于给定的key，计算其hashCode，并除以分区的个数取余，如果余数小于0，则用余数+分区的个数（否则加0），最后返回的值就是这个key所属的分区ID。

#### Ranger分区

`HashPartitioner`分区弊端：可能导致每个分区中数据量的不均匀，极端情况下会导致某些分区拥有RDD的全部数据。

`RangePartitioner`作用：将一定范围内的数映射到某一个分区内，尽量保证每个分区中数据量的均匀，而且分区与分区之间是有序的。

**第一步**：先重整个RDD中抽取出样本数据，将样本数据排序，计算出每个分区的最大key值，形成一个Array[KEY]类型的数组变量rangeBounds；

**第二步**：判断key在rangeBounds中所处的范围，给出该key值在下一个RDD中的分区id下标；该分区器要求RDD中的KEY类型必须是可以排序的

#### 自定义分区

要实现自定义的分区器，你需要继承 `org.apache.spark.Partitioner` 类并实现下面三个方法：

（1）`numPartitions`: Int:返回创建出来的分区数。

（2）`getPartition(key: Any): Int`:返回给定键的分区编号(0到numPartitions-1)。 

（3）`equals()`:Java 判断相等性的标准方法。这个方法的实现非常重要，Spark 需要用这个方法来检查你的分区器对象是否和其他分区器实例相同，这样 Spark 才可以判断两个 RDD 的分区方式是否相同。

## RDD编程进阶

### 累加器

累加器用来对信息进行聚合，通常在向Spark传递函数时，比如使用map() 函数或者用 filter() 传条件时，可以使用驱动器程序中定义的变量，但是集群中运行的每个任务都会得到这些变量的一份新的副本，更新这些副本的值也不会影响驱动器中的对应变量。如果我们想实现所有分片处理时更新共享变量的功能，那么累加器可以实现我们想要的效果。

```scala
 sc.accumulator(0)
```

### 广播变量

广播变量用来高效分发较大的对象。向所有工作节点发送一个较大的只读值，以供一个或多个Spark操作使用。

```scala
val broadcastVar = sc.broadcast(Array(1, 2, 3))
```

## SparkCore优化

