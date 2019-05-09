# Spark 简介

## 什么是Spark

Spark是一种快速、通用、可扩展的大数据分析引擎，是基于内存计算的大数据并行计算框架。

### 一站式的数据处理平台

**Spark Core**：实现了 Spark 的基本功能，包含任务调度、内存管理、错误恢复、与存储系统 交互等模块。Spark Core 中还包含了对弹性分布式数据集（resilient distributed dataset，简称RDD）的 API 定义。

**Spark SQL**：是 Spark 用来操作结构化数据的程序包。通过 Spark SQL，我们可以使用 SQL 或者 Apache Hive 版本的 SQL 方言(HQL)来查询数据。Spark SQL 支持多种数据源，比 如 Hive 表、Parquet 以及 JSON 等。

**Spark Streaming**：是 Spark 提供的对实时数据进行流式计算的组件。提供了用来操作数据流的 API，并且与 Spark Core 中的 RDD API 高度对应。

**Spark MLlib：**提供常见的机器学习(ML)功能的程序库。包括分类、回归、聚类、协同过滤等，还提供了模型评估、数据 导入等额外的支持功能。

**集群管理器**：Spark 设计为可以高效地在一个计算节点到数千个计算节点之间伸缩计 算。为了实现这样的要求，同时获得最大灵活性，Spark 支持在各种集群管理器(cluster manager)上运行，包括 Hadoop YARN、Apache Mesos，以及 Spark 自带的一个简易调度 器，叫作独立调度器。 

### 为什么比MR快

1. 基于内存计算，减少低效的磁盘交互
2. 高效的调度算法，基于DAG
3. 容错机制Linage，精华部分就是DAG和Lingae

## 安装部署Spark

### Local模式

Local模式就是运行在一台计算机上的模式。

local：所有计算都运行在一个线程中

local[n:Int]：指定使用几个线程来运行计算

local[*]：按照CPU最多的核心来设置线程数量

#### 提交流程

`Client`提交应用，`Master`找到一个`Worker`启动`Driver`，`Driver`向 `Master`或者资源管理器申请资源，之后将应用转化为`RDD Graph`，再由`DAGScheduler`将R`DD Graph`转化为`DAG`提交给`TaskScheduler`，由`TaskScheduler`提交任务给 `Executor`执行。在任务执行的过程中，其他组件协同工作，确保整个应用顺利执行。

**Driver（驱动器）**：Spark的驱动器是执行开发程序中的main方法的进程，负责来创建SparkContext、创建RDD，以及进行RDD的转化操作和行动操作代码的执行，把用户程序转为任务，跟踪Executor的运行状况，为执行器节点调度任务。

**Executor（执行器）**：Spark Executor是一个工作进程，负责在 Spark 作业中运行任务，任务间相互独立。

#### 数据流程

- `textFile("input")`：读取本地文件input文件夹数据

- `flatMap(_.split(" "))`：压平操作，按照空格分割符将一行数据映射成一个个单词

- `map((_,1))`：对每一个元素操作，将单词映射为元组

- `reduceByKey(_+_)`：按照key将值进行聚合，相加

- `collect`：将数据收集到Driver端展示。

（1）解压文件后，配置Spark的`spark-env.sh`

```bash
JAVA_HOME=/opt/modules/jdk1.8.0_201
SCALA_HOME=/opt/modules/scala-2.11.8
HADOOP_CONF_DIR=/opt/modules/cdh5.14.2/hadoop-2.6.0-cdh5.14.2/etc/hadoop
SPARK_LOCAL_IP=centos7
```

（2）启动HDFS

（3）测试环境

```bash
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ bin/run-example SparkPi
```

（4）启动命令行界面

```bash
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ bin/spark-shell
# :quit 退出
```

![命令行](assets/20190425210848.png)

### StandAlone

构建一个由Master+Slave构成的Spark集群，Spark运行在集群中。

- Master：管理/申请/调度集群中的资源，比如：CPU，内存，网络

- Worker：管理当前节点上的资源，并且开启Executor

（1）修改spark-env.sh

```bash
SPARK_MASTER_HOST=centos7
SPARK_MASTER_PORT=7070
SPARK_MASTER_WEBUI_PORT=8080
#指定当前集群中每个worker可以使用的cpu的核数
SPARK_WORKER_CORES = 2
#指定当前集群中每个worker可以使用的内存的大小
SPARK_WORKER_MEMORY=2g
SPARK_WORKER_PORT=7071
SPARK_WORKER_WEBUI_PORT=8081
#yarn的nodemanager节点，一台机器只能存在一个，但是StandAlone,一台机器可以存在多个Worker
SPARK_WORKER_INSTANCES=2
```

（2）修改slaves.template为slaves，添加本机，完全分布式，写上全部的主机名

（3）启动服务

```bash
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ sbin/start-master.sh
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ sbin/start-slave.sh spark://centos7:7070
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ jps
10689 Worker
9702 DataNode
10535 Master
9608 NameNode
10745 Worker
10795 Jps
# 关闭
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ sbin/stop-master.sh
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ sbin/stop-slave.sh
```

Master：http://centos7:8080/

启动shell

```bash
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ bin/spark-shell --master spark://centos7:7070
```

### Spark高可用

（1）基于单节点的本地文件系统的master的恢复机制

参考地址：http://spark.apache.org/docs/2.2.1/spark-standalone.html#high-availability

```bash
SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=FILESYSTEM -Dspark.deploy.recoveryDirectory=/SparkRecovery"
```

（2）基于Zookeeper的文件系统的master的恢复机制

```bash
SPARK_DAEMON_JAVA_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=centos7:2181 -Dspark.deploy.zookeeper.dir=/SparkRecovery"
```

（3）日志服务器

因为当spark程序结束之后，4040界面就会消失，为了解决这个问题，我们需要配置spark应用的历史服务，4040界面还有一个特点：当4040端口被占用，则自动往后递增 4041.4042…

首先要开启YARN的日志聚合功能和历史日志服务器，这里不再说明。

接着在HDFS上创建日志服务目录

```bash
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ bin/hdfs dfs -mkdir /spark-2.2.1/historyserver
```

重命名`spark-defaults.conf.template` 为`spark-defaults.conf`

```bash
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://centos7:8020/spark-2.2.1/historyserver
```

修改`spark-env.sh`

```bash
SPARK_HISTORY_OPTS="-Dspark.history.fs.logDirectory=hdfs://centos7:8020/spark-2.2.1/historyserver -Dspark.history.ui.port=18080"
```

开启Spark日志聚合服务

```bash
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ sbin/start-history-server.sh
```

最终启动方式

先启动Hadoop相关服务，然后使用spark的`start-all.sh`和`start-history-server.sh`

测试计算PI

```bash
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ bin/spark-submit --class org.apache.spark.examples.SparkPi --master spark://centos7:7070 --executor-memory 1G --total-executor-cores 2 examples/jars/spark-examples_2.11-2.2.1.jar 100


Pi is roughly 3.141918314191831
19/04/26 10:56:07 INFO server.AbstractConnector: Stopped Spark@3deb2326{HTTP/1.1,[http/1.1]}{centos7:4040}
```

配置YARN服务器

```bash
# spark-defaults.conf
spark.yarn.historyServer.address=centos7:18080
spark.history.ui.port=4000
```
修改 `yarn-site.xml`

```xml
<property>
		<name>yarn.nodemanager.pmem-check-enabled</name>
		<value>false</value>
</property>
<property>
		<name>yarn.nodemanager.vmem-check-enabled</name>
		<value>false</value>
</property>
```

提交到YARN

```bash
[fanl@centos7 spark-2.2.1-bin-2.6.0-cdh5.14.2]$ bin/spark-submit --class org.apache.spark.examples.SparkPi --master yarn --deploy-mode client examples/jars/spark-examples_2.11-2.2.1.jar 100
```

查看日志服务器WEB页面

## Spark简单的操作

Spark读取的文件都在HDFS上，读取本地文件改为`file://`

（1）读取文件案例，需要将需要统计的文本上传到HDFS上

```bash
scala> val path = "file:///home/fanl/data/wc.txt"
path: String = file:///home/fanl/data/wc.txt

scala> val rdd=sc.textFile(path)
rdd: org.apache.spark.rdd.RDD[String] = file:///home/fanl/data/wc.txt MapPartitionsRDD[3] at textFile at <console>:26

scala> rdd.collect
res1: Array[String] = Array(hadoop, hadoop flume oozie hue, hive yarn hbase)

scala> 

```

（2）IDEA示例

```scala
package com.fanling

import org.apache.spark.{SparkConf, SparkContext}

object WordCount {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("wordcount").setMaster("spark://hadoop01:7077")
    val sc = new SparkContext(conf)
    val path = "hdfs://hadoop01:8020/user/fanl/word.txt"
    val rdd = sc.textFile(path)
    for (i <- rdd.collect()) {
      println(i)
    }
    sc.stop()
  }
}
```

如果报错，连接不到Master，检查集群的版本和`pom.xml`文件中版本是否一致



