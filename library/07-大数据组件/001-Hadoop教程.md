# Hadoop教程

Hadoop是一个由Apache基金会所开发的分布式系统基础架构。

主要解决海量数据存储和海量数据分析计算问题。

由Doug Cutting 设计开发，参考了谷歌的3篇论文，（1）The Google File System；（2）MapReduce: Simpliﬁed Data Processing on Large Clusters；（3）Bigtable: A Distributed Storage System for Structured Data

## Haddop的发行版本

- Apache 版本，原始版本
- Cloudera Hadoop版本，企业用的多
- Hortonworks Hadoop版本，文档完善

## Hadoop的优势

- 高可靠性：底层维护（默认3）个数据副本，即使节点故障也不会丢失数据
- 高扩展性：在集群间分配任务数据，可方便扩展数以千计的节点
- 高效性：在MapReduce的思想下，Hadoop是并行运行的，以加快处理速度
- 高容错性：能够自动将失败的任务重新分配

## Hadoop版本区别

问题：Hadoop 1.x 和2.x 有什么区别？

> Hadoop 1.x

由以下组件组成：

- Common 辅助工具
- HDFS 数据存储
- MapReduce 计算并且资源调度

> Hadoop 2.x

由以下组件组成：

- Common 辅助工具
- HDFS 数据存储
- Yarn 资源调度
- MapReduce 计算

由此可以看到，在Hadoop 1.x时代，Hadoop中的MapReduce同时处理业务逻辑运算和资源的调度，耦合性较大，在2.x 时代增加了Yarn，Yarn 只负责资源调度，MapReduce只负责运算。

## Hadoop 组成

### HDFS架构概述

> NameNode(nn)

存储文件的元数据，如文件名，文件的目录结构，文件属性等，以及每个文件的块列表和块所在的DataNode等

> DataNode(dn)

在本地文件系统存储文件快数据，以及块数据的校验和

> Secondary NameNode(2nn)

用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照。

### YARN架构

> ResourceManager

处理客户端的请求，监控NodeManager，启动和监控Application Master(任务job)，负责资源的分配和调度

> NodeManager

管理单个节点上的资源，处理来自RM的命令和来自Application Master的命令。

> Application Master

负责数据的切分，为应用程序申请资源并且分配给内部的任务，并且对任务进行监控和容错。

> Container

封装了节点的资源维度（CPU，内存，磁盘，网络等）。

### MapReduce

- Map阶段并行处理输入数据
- Reduce阶段对Map结果进行汇总

## Hadoop的环境搭建

### 环境准备

- 修改网络为静态ip
- 修改主机名
- 关闭防火墙
- 创建用户，配置root权限
- 在/opt创建文件/module和/software，并且修改文件拥有者，并且安装了jdk
- 克隆多态虚拟机

### 解压和配置环境变量

```bash
$ sudo vi /etc/profile

# HADOOP_HOME
$ export HADOOP_HOME=/opt/module/hadoop-2.9.2
$ export PATH=$PATH:$HADOOP_HOME/bin
$ export PATH=$PATH:$HADOOP_HOME/sbin

$ source /etc/profile
# 检测
$ hadoop
```

## Hadoop 目录结构

- /bin：存放命令
- /etc/hadoop：配置文件信息
- /sbin：启动关闭集群等命令
- /share：手册

## Hadoop启动模式

Hadoop 集群可以运行的 3 个模式有：

- 单机模式
- 伪分布模式
- 全分布模式

### 单机模式

默认情况下，Hadoop被配置成以非分布式模式运行的一个独立Java进程。这对调试非常有帮助。

不会存在守护进程，所有东西都运行在一个 JVM 上，这里同样没有 HDFS，使用的是本地文件系统。

> 官方案例 grep

```bash
# 在文件安装路径下新建input
$ cp etc/hadoop/*.xml input/
# 执行命令
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar grep input/ output 'dfs[a-z.]+'
# output 不可以提前创建
```

> 官方案例 word count

```bash
$ mkdir wcinput
$ vi wc.input
# 输入一些字母
# 执行
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wordcount wcinput/ wcoutput
```

### 伪分布模式

#### Hadoop配置

**修改hadoop-env.sh**

```bash
# 修改JAVA_HOME 地址
export JAVA_HOME=/opt/module/jdk1.8.0_201
```

**配置 core-site.xml**

```xml
<configuration>
    <!--修改Hadoop的NameNode的地址-->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <!--指定Hadoop运行时候的产生文件的临时存储目录-->
      <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-2.9.2/data/tmp</value>
    </property>
</configuration>
```

**配置hdfs-site.xml**

```xml
<configuration>
    <!--指定hdfs副本的数量，默认值3，保存数据的副本，防止数据丢失-->
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

#### 启动HDFS

第一次启动需要格式化，以后不需要再格式化，需要关闭服务，删除data和log文件夹再次格式化。

**原因**：格式化namenode 会产生新的集群id，导致和datanode不一致。

```bash
# 格式化namenode	
$ bin/hdfs namenode -format
# 启动namenode
$ sbin/hadoop-daemon.sh start namenode
# 启动datanode
$ sbin/hadoop-daemon.sh start datanode
# 查看是否启动
$ jps
# 结果
7634 NameNode
7690 DataNode
7770 Jps
```

查看控制面板，端口号为50070

![面板](assets/20190308194111.png)

若打不开，请查看防火墙

```bash
systemctl start firewalld.service     # 启动firewall
systemctl stop firewalld.service      # 停止firewall
systemctl disable firewalld.service   # 禁止firewall开机启动
```

##### HDFS的操作

**创建文件夹**

```bash
# 创建文件夹
$ bin/hdfs dfs -mkdir -p /user/fanl/input
# 创建完查看目录，已经存在路径，和linux命令是一致的 其中，bin/hdfs dfs 是固定的写法
```

![控制板](assets/20190308195627.png)

**复制本地文件到HDFS**

```bash
$ bin/hdfs dfs -put input/fanl.txt /user/fanl/input
```

**执行wordcount**

```bash
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wordcount /user/fanl/input/ /user/fanl/output
# 查看结果
$ bin/hdfs dfs -cat /user/fanl/output/*
```

#### 启动YARN

yarn-env.sh配置

```bash
# 修改JAVA_HOME地址
export JAVA_HOME=/opt/module/jdk1.8.0_201
```

yarn-site.xml配置

```xml
<configuration>
    <!--MapReduce获取数据的方式-->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <!--YARN的resourcemanager的地址-->
     <property>
        <name>yarn.resourcemanager.hoastname</name>
        <value>fanl01</value>
    </property>
</configuration>
```

mapred-env.sh的修改

```bash
# 修改JAVA_HOME地址
export JAVA_HOME=/opt/module/jdk1.8.0_201
```

mapred-site.xml的修改

```xml
<configuration>
    <!--指定MR运行在YARN上-->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

启动yarn

```bash
$ sbin/yarn-daemon.sh start resourcemanager
$ sbin/yarn-daemon.sh start nodemanager
# 查看启动
8272 ResourceManager
7634 NameNode
8322 NodeManager
7690 DataNode
8397 Jps
```

查看控制面板，端口号为 8088

```bash
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wordcount /user/fanl/input/ /user/fanl/output
# 并且查看控制面板
```

##### 配置历史服务器

修改mapred-site.xml

```xml
<configuration>
    <!--历史服务器地址-->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>fanl01:10020</value>
    </property>
    <!--历史服务器地址WEB地址-->
     <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>fanl01:19888</value>
    </property>
</configuration>
```

启动历史服务器

```bash
$  sbin/mr-jobhistory-daemon.sh start historyserver
```