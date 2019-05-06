# Flume教程

## 什么是Flume

Flume是Cloudera提供的一个高可用的，高可靠的，分布式的海量日志采集、聚合和传输的系统。Flume基于流式架构，灵活简单。

### Flume的组成架构

![架构](assets/20190415151428.png)

（1）**Agent**是一个JVM进程，它以事件的形式将数据从源头送至目的，是Flume数据传输的基本单元。Agent主要有3个部分组成，Source、Channel、Sink。

（2）**Source**是负责接收数据到Flume Agent的组件。Source组件可以处理各种类型、各种格式的日志数据，包括avro、thrift、exec、jms、spooling directory、netcat、sequence generator、syslog、http、legacy。

（3）**Channel**是位于Source和Sink之间的缓冲区。因此，Channel允许Source和Sink运作在不同的速率上。Channel是线程安全的，可以同时处理几个Source的写入操作和几个Sink的读取操作。
Flume自带两种Channel：Memory Channel和File Channel。
Memory Channel是内存中的队列。Memory Channel在不需要关心数据丢失的情景下适用。如果需要关心数据丢失，那么Memory Channel就不应该使用，因为程序死亡、机器宕机或者重启都会导致数据丢失。
File Channel将所有事件写到磁盘。因此在程序关闭或机器宕机的情况下不会丢失数据。

（4）**Sink**不断地轮询Channel中的事件且批量地移除它们，并将这些事件批量写入到存储或索引系统、或者被发送到另一个Flume Agent。

Sink是完全事务性的。在从Channel批量删除数据之前，每个Sink用Channel启动一个事务。批量事件一旦成功写出到存储系统或下一个Flume Agent，Sink就利用Channel提交事务。事务一旦被提交，该Channel从自己的内部缓冲区删除事件。

Sink组件目的地包括hdfs、logger、avro、thrift、ipc、file、null、HBase、solr、自定义。

（5） **Event**是传输单元，Flume数据传输的基本单元，以事件的形式将数据从源头送至目的地。

### Flume的安装部署

解压文件，移动文件，将/conf下的`flume-env.sh.template`文件修改为`flume-env.sh`，并配置`flume-env.sh`文件。

其余操作和以前安装工具框架是一样的。

```bash
# 启动案例
# bin/flume-ng agent 启动代理
# -c 配置文件目录
# -n 代理名称
# -f 指定配置文件
[fanl@centos7 flume-1.6.0-cdh5.14.2]$ nohup bin/flume-ng agent -c conf/ -n a1 -f conf/flume-port.properties &
```

## Flume案例

### 1. 实时监听端口数据案例

**案例**：定义一个flume-agent代理去监听读取某台服务器上的某个端口中的数据，并将监听读取到的数据最终写入到flume框架自己的日志文件中 ，这里使用的是4545。

> （1）首先安装telnet工具

不再说明。

> （2）检查4545端口是否被占用

```bash
[fanl@centos7 flume-1.6.0-cdh5.14.2]$ netstat -tunlp | grep 4545
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
[fanl@centos7 flume-1.6.0-cdh5.14.2]$ 
```

> （3）创建本次案例的配置文件

```bash
[fanl@centos7 conf]$ cp flume-conf.properties.template flume-poert.properties
# A netcat-like source that listens on a given port and turns each line of text into an event. 
# 将内容按如下替换
a1.sources = s1
a1.channels = c1
a1.sinks = k1

# For each one of the sources, the type is defined
a1.sources.s1.type = netcat  # 源类型。下面的配置是文档手册规定的必须提供的属性值
a1.sources.s1.bind = 192.168.157.160
a1.sources.s1.port = 4545

# The channel can be defined as follows.
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100  

# Each sink's type must be defined
a1.sinks.k1.type = logger

#Specify the channel the sink should use
a1.sources.s1.channels = c1
a1.sinks.k1.channel = c1
```

> （4）后台启动Flume

```bash
# 按照上面的帮助信息，启动进程
[fanl@centos7 flume-1.6.0-cdh5.14.2]$ nohup bin/flume-ng agent -c conf/ -n a1 -f conf/flume-port.properties &
[2] 12136
[fanl@centos7 flume-1.6.0-cdh5.14.2]$ nohup: 忽略输入并把输出追加到"nohup.out"

[fanl@centos7 flume-1.6.0-cdh5.14.2]$ jps
7377 NodeManager
7058 NameNode
7111 DataNode
7191 SecondaryNameNode
7319 ResourceManager
12136 Application
12169 Jps
7615 JobHistoryServer
```

> （5）telnet发送数据并且查看Flume的日志

首先打开一个新窗口，将监控`logs/flume.log` 

```bash
[fanl@centos7 flume-1.6.0-cdh5.14.2]$ tail -F logs/flume.log 
```

然后打开telnet客户端输入信息

```bash
[fanl@centos7 flume-1.6.0-cdh5.14.2]$ telnet 192.168.157.160 4545
Trying 192.168.157.160...
Connected to 192.168.157.160.
Escape character is '^]'.
```

在telnet端输入“hello world”，查看*flume.log*的变化

```
15 四月 2019 18:04:42,387 INFO  [SinkRunner-PollingRunner-DefaultSinkProcessor] (org.apache.flume.sink.LoggerSink.process:95)  - Event: { headers:{} body: 68 65 6C 6C 6F 20 77 6F 72 6C 64 0D             hello world. }
```

### 2. 实时读取本地文件到HDFS

**案例**：使用Flume实时监控读取系统本地一个日志文件中动态追加的日志数据并实时写入到HDFS上的某个目录下，并且进行优化：

1. 解决生成的文件过多过小的问题（希望文件的大小=128M） 
2. 将日志文件按照日期分目录存储（按照天分目录存储）  
3. 将生成的日志文件的格式改为Text文本格式 

> （1）在保证Flume能操作HDFS前需要将相关jar包引入到Flume的lib目录下。

```bash
[fanl@centos7 flume-1.6.0-cdh5.14.2]$ cd /opt/modules/cdh5.14.2/hadoop-cdh5.14.12/share/hadoop/
[fanl@centos7 hadoop]$ cp hdfs/hadoop-hdfs-2.6.0-cdh5.14.2.jar /opt/modules/cdh5.14.2/flume-1.6.0-cdh5.14.2/lib/
[fanl@centos7 hadoop]$ cp common/hadoop-common-2.6.0-cdh5.14.2.jar /opt/modules/cdh5.14.2/flume-1.6.0-cdh5.14.2/lib/
[fanl@centos7 hadoop]$ cp common/lib/htrace-core4-4.0.1-incubating.jar /opt/modules/cdh5.14.2/flume-1.6.0-cdh5.14.2/lib/
[fanl@centos7 hadoop]$ cp tools/lib/commons-configuration-1.6.jar /opt/modules/cdh5.14.2/flume-1.6.0-cdh5.14.2/lib/
[fanl@centos7 hadoop]$ cp tools/lib/hadoop-auth-2.6.0-cdh5.14.2.jar /opt/modules/cdh5.14.2/flume-1.6.0-cdh5.14.2/lib/

```

> （2）启动新的Flume之前需要新增配置文件为`flume-file.properties`，并且agent的别名需要唯一指定。

在官网文档中查找源类型的配置信息为`exec`。

```bash
a2.sources = s2
a2.channels = c2
a2.sinks = k2

# For each one of the sources, the type is defined
a2.sources.s2.type = exec
a2.sources.s2.command = tail -F /home/fanl/nginx.log
a2.sources.s2..shell = /bin/bash -c

# The channel can be defined as follows.
a2.channels.c2.type = memory
a2.channels.c2.capacity = 1000
a2.channels.c2.transactionCapacity = 100   

# Each sink's type must be defined
a2.sinks.k2.type = hdfs
a2.sinks.k2.hdfs.path = hdfs://centos7:8020/user/flume/%Y%d%m
a2.sinks.k2.hdfs.round = true
a2.sinks.k2.hdfs.useLocalTimeStamp = true
a2.sinks.k2.hdfs.filePrefix = NgnixLog

# file too little
a2.sinks.k2.hdfs.rollInterval = 0
a2.sinks.k2.hdfs.rollSize = 128000000
a2.sinks.k2.hdfs.rollCount = 0
# required
a2.sinks.k2.hdfs.minBlockReplicas = 1

# default SequenceFile
a2.sinks.k2.hdfs.fileType = DataStream
# format
a2.sinks.k2.hdfs.writeFormat = Text 
a2.sinks.k2.hdfs.batchSize = 100
#Specify the channel the sink should use
a2.sources.s2.channels = c2
a2.sinks.k2.channel = c2
```

> （3）启动测试

```bash
[fanl@centos7 flume-1.6.0-cdh5.14.2]$ nohup bin/flume-ng agent -n a2 -c conf/ -f conf/flume-file.properties &
```

![结果](assets/20190415183755.png)

### 3. 实时读取目录文件到某目录

实时读取`/home/fanl/data`目录下的日志到`/home/fanl/flume-logs`上

注意：需要先创建`/home/fanl/flume-logs`

```properties
a3.sources = s3
a3.channels = c3
a3.sinks = k3
# For each one of the sources, the type is defined
a3.sources.s3.type = spooldir
a3.sources.s3.spoolDir=/home/fanl/data
a3.sources.s3.fileSuffix = .complete
a3.sources.s3.ignorePattern = ^.*\.complete$
# The channel can be defined as follows.
a3.channels.c3.type = file
# Each sink's type must be defined
a3.sinks.k3.type = file_roll
a3.sinks.k3.sink.directory = /home/fanl/flume-logs
#Specify the channel the sink should use
a3.sources.s3.channels = c3
a3.sinks.k3.channel = c3
```

启动测试

```bash
[fanl@centos7 flume-1.6.0-cdh5.14.2]$ nohup bin/flume-ng agent -n a3 -c conf/ -f conf/flume-dir.properties &
```

### 4. 单数据源多选择器出口案例

![单数据源多选择器出口案例](assets/20190416180339.png)

### 5. 单数据源多Sinks出口案例

![单数据源多Sinks出口案例](assets/20190416180315.png)

### 6. 多数据源汇总案例

![多数据源汇总案例](assets/20190416180236.png)

以上的几个案例注意配置文件的作用，不要写错了即可。

## 相关问题

（1）Flume 采集数据会丢失吗?

