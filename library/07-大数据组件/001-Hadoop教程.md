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

## 环境准备

- 修改网络为静态ip
- 修改主机名
- 关闭防火墙
- 创建用户，配置root权限
- 在/opt创建文件/module和/software，并且修改文件拥有者，并且安装了jdk
- 克隆多态虚拟机

## 解压和配置环境变量

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

## 官方案例

> grep

```bash
# 在文件安装路径下新建input
$ cp etc/hadoop/*.xml input/
# 执行命令
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar grep input/ output 'dfs[a-z.]+'
# output 不可以提前创建
```

> word count

### 单机模式

默认情况下，Hadoop被配置成以非分布式模式运行的一个独立Java进程。这对调试非常有帮助。

不会存在守护进程，所有东西都运行在一个 JVM 上，这里同样没有 HDFS，使用的是本地文件系统。



