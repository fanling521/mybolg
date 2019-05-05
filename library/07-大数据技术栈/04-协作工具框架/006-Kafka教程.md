# Kafka教程

## 什么是Kafka

Apache Kafka是一个**分布式消息队列**，Kafka对消息保存时根据Topic进行归类，发送消息者称为Producer，消息接受者称为Consumer，此外kafka集群有多个kafka实例组成，每个实例称为broker。

### Kafka架构

1. **Producer** ：消息生产者，就是向kafka broker发消息的客户端
2. **Consumer** ：消息消费者，向kafka broker取消息的客户端
3. **Topic** ：可以理解为一个队列
4.  **Consumer Group （CG）**：这是kafka用来实现一个topic消息的广播（发给所有的consumer）和单播（发给任意一个consumer）的手段。一个topic可以有多个CG。topic的消息会复制（不是真的复制，是概念上的）到所有的CG，但每个partion只会把消息发给该CG中的一个consumer。如果需要实现广播，只要每个consumer有一个独立的CG就可以了。要实现单播只要所有的consumer在同一个CG。用CG还可以将consumer进行自由的分组而不需要多次发送消息到不同的topic
5. **Broker** ：一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic
6. **Partition**：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列。partition中的每条消息都会被分配一个有序的id（offset）。kafka只保证按一个partition中的顺序将消息发给consumer，不保证一个topic的整体（多个partition间）的顺序
7. **Offset**：kafka的存储文件都是按照offset.kafka来命名，用offset做名字的好处是方便查找。例如你想找位于2049的位置，只要找到2048.kafka的文件即可。当然the first offset就是00000000000.kafka

## Kafka安装部署

（1）安装Scala环境

```bash
export SCALA_HOME=/opt/cdh-5.14.2/scala-2.11.8
export PATH=$PATH:$JAVA_HOME/bin:$SCALA_HOME/bin
```

（2）broker节点配置文件server.properties

```properties
# 一个kakfa集群上的每个broker节点的id号必须唯一整数
broker.id=0		
# 配置当前broker节点服务器的主机名及kafka server监听端口
listeners=PLAINTEXT://centos7:9092		
# 声明当前broker节点上消息数据的本地存储目录 
log.dirs=/opt/cdh-5.14.2/kafka_2.11-1.0.0/data		
# 声明zookeeper集群的通讯地址
zookeeper.connect=centos7:2181	
```

（2）生产者配置文件

```properties
# 声明当前kafka集群的所有的broker节点通讯地址
bootstrap.servers=centos7:9092
```

（3）消费者配置文件consumer.properties

```properties
# 声明当前kafka集群的所有的broker节点通讯地址
bootstrap.servers=centos7:9092
```

（4）启动zookeeper服务和kfaka服务

```bash
[fanl@centos7 zookeeper-3.4.5-cdh5.14.2]$ bin/zkServer.sh start
[fanl@centos7 kafka_2.11-1.0.0]$ bin/kafka-server-start.sh  config/server.properties &
# 进程名称
[fanl@centos7 kafka_2.11-1.0.0]$ jps
11873 QuorumPeerMain
12148 Kafka
```

## Kfaka操作案例

（1）创建一个topic主题 

```bash
[fanl@centos7 kafka_2.11-1.0.0]$ bin/kafka-topics.sh  \
--create \
--zookeeper centos7:2181 \
--replication-factor 1 \
--partitions 1 \
--topic source
```

（2）查看主题列表

```bash
[fanl@centos7 kafka_2.11-1.0.0]$ bin/kafka-topics.sh \
--list \
--zookeeper centos7:2181
```

（3）启动前端命令行生产者

```bash
[fanl@centos7 kafka_2.11-1.0.0]$ bin/kafka-console-producer.sh \
--broker-list centos7:9092 \
--topic source
```

（4）启动前端命令行消费者

```bash
[fanl@centos7 kafka_2.11-1.0.0]$ bin/kafka-console-consumer.sh \
--bootstrap-server centos7:9092 \
--topic testTopics \
--from-beginning
```

