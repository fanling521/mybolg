# Spark Streaming

## 什么是Spark Streaming

Spark Streaming用于流式数据的处理。

### 形成流的方式

#### 使用数据接收器

receiver在接收数据的时候，把输入的数据以数据块（block）的形式存在内存或者磁盘上，`spark.streaming.receiver.writeAheadLog=true`表示把块的元数据信息保留下来，直到这些块被全部处理完毕之后，才会被删除。

默认情况下，200ms就会产生一个block块，在RDD执行的过程中，一个block块对应一个task任务，参数`spark.streaming.blockInterval`

每隔一个batchDuration（批次产生的间隔时间），就会产生一个新的批次，这个批次中的数据就是这段时间内接受到的所有的数据。

RDD的一个task就是一个block块，一个streaming程序可以包含多条流，一个DStream对象中，每个批次只会产生一个RDD。

一个job就是一个rdd的action算子触发产生的。

#### 直接获取数据

direct模式不会启动receiver，就不会产生block块，每隔一个batchDuration的时间，就会产生一个待运行的批次，批次就是RDD，产生的RDD中保存时数据的元数据信息。

## DStream

Spark Streaming使用离散化流(discretized stream)作为抽象表示，叫作DStream。DStream 是随时间推移而收到的数据的序列。

DStream 可以从各种输入源创建，比如 Flume、Kafka 或者 HDFS。创建出来的DStream
支持两种操作，一种是转化操作(transformation)，会生成一个新的DStream，另一种是输出操作(output operation)，可以把数据写入外部系统中

### 入门案例

使用工具向9999端口不断的发送数据，通过Spark Streaming读取端口数据并统计不同单词出现的次数。

```scala
  def main(args: Array[String]): Unit = {
    //1.初始化Spark配置信息
    val sparkConf = new SparkConf().setMaster("local[*]").setAppName("StreamWordCount")
    //2.初始化SparkStreamingContext
    val ssc = new StreamingContext(sparkConf, Seconds(5))
    //3.通过监控端口创建DStream，读进来的数据为一行行
    val lineStreams = ssc.socketTextStream("centos7", 9999)
    //将每一行数据做切分，形成一个个单词
    val wordStreams = lineStreams.flatMap(_.split(" "))
    //将单词映射成元组（word,1）
    val wordAndOneStreams = wordStreams.map((_, 1))
    //将相同的单词次数做统计
    val wordAndCountStreams = wordAndOneStreams.reduceByKey(_ + _)
    //打印
    wordAndCountStreams.print()
    //启动SparkStreamingContext
    ssc.start()
    //如果上一个批次没有处理完成，后面的批次处于阻塞状态
    ssc.awaitTermination()
  }
```

nc启动端口，并且发送数据

```bash
[fanl@centos7 conf]$ nc -lk 9999
hello world
```

### DStream创建

#### 文件数据源

能够读取所有HDFS API兼容的文件系统文件，通过`fileStream`方法进行读取，Spark Streaming 将会监控 dataDirectory 目录并不断处理**移动**进来的文件，**记住目前不支持嵌套目录**。

```scala
ssc.textFileStream("hdfs://hadoop01:8020/fileStream")
```

#### RDD队列

可以通过使用`ssc.queueStream(queueOfRDDs)`来创建DStream。

```scala
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local[*]").setAppName("demo2")
    val ssc = new StreamingContext(conf, Seconds(5))
    val rddQueue = new mutable.Queue[RDD[Int]]()
    val inStream = ssc.queueStream(rddQueue, oneAtATime = false)
    val inStream2 = inStream.map((_, 1))
    val inStream3 = inStream2.reduceByKey(_ + _)
    inStream3.print()
    ssc.start()
    for (i <- 1 to 5) {
      rddQueue += ssc.sparkContext.makeRDD(1 to 100)
      Thread.sleep(1000)
    }
    ssc.awaitTermination()
  }
```

#### 自定义数据源

需要继承Receiver，并实现onStart、onStop方法来自定义数据源采集。

```scala
class CustomerReceiver(host: String, port: Int) extends Receiver[String](storageLevel = StorageLevel.MEMORY_ONLY) {}
```

#### Kafka数据源⭐

在工程中需要引入 Maven 工件`spark- streaming-kafka_2.11` 来使用它。包内提供的 `KafkaUtils` 对象可以在StreamingContext 和 JavaStreamingContext 中以你的Kafka 消息创建出 DStream。

```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka_2.11</artifactId>
    <version>1.6.3</version>
</dependency>
<!-- 使用其他版本 [推荐] -->
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming-kafka-0-8_2.11</artifactId>
    <version>2.2.1</version>
</dependency>
```

使用接收器模式

```scala
  def main(args: Array[String]): Unit = {
    //1.创建SparkConf并初始化SSC
    val sparkConf: SparkConf = new SparkConf().setMaster("local[*]").setAppName("KafkaSparkStreaming")
    val ssc = new StreamingContext(sparkConf, Seconds(5))

    //2.定义kafka参数
    val zookeeper = "centos7:2181"
    val topic = "source"
    val consumerGroup = "spark"

    //3.将kafka参数映射为map
    val kafkaParam: Map[String, String] = Map[String, String](
      ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG -> "org.apache.kafka.common.serialization.StringDeserializer",
      ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG -> "org.apache.kafka.common.serialization.StringDeserializer",
      ConsumerConfig.GROUP_ID_CONFIG -> consumerGroup,
      "zookeeper.connect" -> zookeeper
    )

    //4.通过KafkaUtil创建kafkaDSteam
    val kafkaDSteam: ReceiverInputDStream[(String, String)] = KafkaUtils.createStream[String, String, StringDecoder, StringDecoder](
      ssc,
      kafkaParam,
      Map[String, Int](topic -> 3),
      StorageLevel.MEMORY_ONLY
    )

    //5.对kafkaDSteam做计算（WordCount）
    kafkaDSteam.foreachRDD {
      rdd => {
        val word: RDD[String] = rdd.flatMap(_._2.split(" "))
        val wordAndOne: RDD[(String, Int)] = word.map((_, 1))
        val wordAndCount: RDD[(String, Int)] = wordAndOne.reduceByKey(_ + _)
        wordAndCount.collect().foreach(println)
      }
    }

    //6.启动SparkStreaming
    ssc.start()
    ssc.awaitTermination()

  }
```

### DSteam转换

### 无状态转化操作

无状态转化操作就是把简单的RDD转化操作应用到每个批次上，也就是转化DStream中的每一个RDD

### 有状态转化操作

#### UpdateStateByKey

`UpdateStateByKey`原语用于记录历史记录，有时，我们需要在DStream中跨批次维护状态。针对这种情况，updateStateByKey()为我们提供了对一个状态变量的访问，用于键值对形式的DStream。给定一个由(键，事件)对构成的 DStream，并传递一个指定如何根据新的事件更新每个键对应状态的函数，它可以构建出一个新的 DStream，其内部数据为(键，状态) 对。 

使用`updateStateByKey`的时候必须开启checkpoint机制。应用场景：需要对数据进行累计的操作。

```scala
val result = dstream.flatMap(line => line.split("\t")).map(word => (word,1))
    .updateStateByKey(
      (seq:Seq[Int],state:Option[Int]) => {
       //获取当前批次的value
        val sum = seq.sum
        //获取上一个批次中的数据
        val prestate = state.getOrElse(0)
        //更新状态
        Some(sum + prestate)
    },4)
```

#### Window Operations

可以设置窗口的大小和滑动窗口的间隔来动态的获取当前Steaming的允许状态。

```scala
val result = dstream.flatMap(line => line.split("\t")).map(word => (word,1))
      .reduceByKeyAndWindow(
        (a:Int,b:Int) => a + b,//window中的批次的聚合
        (c:Int,d:Int) => c - d, //c:表示上一个window和新产生的批次的和 d:表示上一个window和当前window不重叠的批次
        Seconds(30), //表示窗口的大小
        Seconds(20)//表示窗口的滑动时间
      )
```

### 其他操作

#### Transform

Transform原语允许DStream上执行任意的RDD-to-RDD函数。即使这些函数并没有在DStream的API中暴露出来，通过该函数可以方便的扩展Spark API。该函数每一批次调度一次。其实也就是对DStream中的RDD应用转换。

####  Join

连接操作（leftOuterJoin, rightOuterJoin, fullOuterJoin也可以），可以连接Stream-Stream，windows-stream to windows-stream、stream-dataset。

### DStream输出

1. `print()`：在运行流程序的驱动结点上打印DStream中每一批次数据的最开始10个元素。这用于开发和调试。在Python API中，同样的操作叫print()
2. `saveAsTextFiles(prefix, [suffix])`：以text文件形式存储这个DStream的内容。每一批次的存储文件名基于参数中的prefix和suffix。”prefix-Time_IN_MS[.suffix]”.
3. `saveAsObjectFiles(prefix, [suffix])`：以Java对象序列化的方式将Stream中的数据保存为 SequenceFiles . 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]". Python中目前不可用
4. `saveAsHadoopFiles(prefix, [suffix])`：将Stream中的数据保存为 Hadoop files. 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]"。Python API Python中目前不可用
5. `foreachRDD(func)`：这是最通用的输出操作，即将函数 func 用于产生于 stream的每一个RDD。其中参数传入的函数func应该实现将每一个RDD中数据推送到外部系统，如将RDD存入文件或者通过网络将其写入数据库。注意：函数func在运行流应用的驱动中被执行，同时其中一般函数RDD操作从而强制其对于流RDD的/  8