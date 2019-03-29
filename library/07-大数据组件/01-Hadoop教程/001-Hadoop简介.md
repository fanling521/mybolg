# Hadoop简介

## 什么是大数据

大数据（*big data*），指无法在一定时间范围内用常规软件工具进行捕捉、管理和处理的数据集合，是需要新处理模式才能具有更强的决策力、洞察发现力和流程优化能力的海量、高增长率和多样化的信息资产。

### 大数据的特征

- **Volume**：数据量大
- **Variety**：种类和来源多样化
- **Value**：数据价值密度相对较低
- **Velocity**：数据增长速度快
- **Veracity**：数据的可靠性

## 什么是Hadoop

Hadoop是一个由Apache基金会所开发的分布式系统基础架构，主要解决海量数据的采集，存储和海量数据分析计算问题。

由*Doug Cutting* 设计开发，参考了谷歌的3篇论文，

（1）The Google File System

（2）MapReduce: Simpliﬁed Data Processing on Large Clusters

（3）Bigtable: A Distributed Storage System for Structured Data

## Hadoop的发行版本

- Apache 版本，原始版本
- Cloudera Hadoop版本，企业用的多
- Hortonworks Hadoop版本，文档完善

## Hadoop的优势

- 高可靠性：底层维护（默认3）个数据副本，即使节点故障也不会丢失数据
- 高扩展性：在集群间分配任务数据，可方便扩展数以千计的节点
- 高效性：在MapReduce的思想下，Hadoop是并行运行的，以加快处理速度
- 高容错性：能够自动将失败的任务重新分配

## Hadoop的组成

### Hadoop Common

Hadoop基础设施和辅助工具

### HDFS架构概述

HDFS是分布式文件系统（**H**adoop **D**istributed **F**ile **S**ystem），用于存储文件，通过目录树来定位文件。适合一次写入，多次读出的场景，不支持修改文件。

**NameNode(nn)：**存储文件的元数据，如文件名，文件的目录结构，文件属性等，以及每个文件的块列表和块所在的DataNode等

**DataNode(dn)：**在本地文件系统存储文件快数据，以及块数据的校验和

**SecondaryNameNode(2nn)：**用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据的快照。

### YARN架构

**ResourceManager：**处理客户端的请求，监控NodeManager，启动和监控Application Master(任务)，负责资源的分配和调度

**NodeManager：**管理单个节点上的资源，处理来自RM的命令和来自Application Master的命令。

**Application Master：**负责数据的切分，为应用程序申请资源并且分配给内部的任务，并且对任务进行监控和容错。

**Container：**封装了节点的资源维度（CPU，内存，磁盘，网络等）。

### MapReduce

- Map阶段并行处理输入数据
- Reduce阶段对Map结果进行汇总

## Hadoop的环境搭建

### 集群节点环境准备

- 修改网络为静态IP
- 修改主机名
- 关闭防火墙和安全子系统
- 创建用户，配置`sudo`权限
- 在`/opt`创建文件`/modules`和`/software`，并且修改文件拥有者，并且安装了jdk
- 安装`ntp`、`rsync`克隆多台虚拟机

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

## 部署方式

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

其他模式单独说明

## Hadoop源码编译

### 工具安装

需要Linux系统

```bash
# =====安装JDK========
# =====安装Maven========
# =====安装ant========
# =====glibc-headers========
[root@centos01 ~]$ yum install glibc-headers
# =====make 和 cmake========
[root@centos01 ~]$ yum install make
[root@centos01 ~]$ yum install cmake
# =====安装openssl库========
[root@centos01 ~]$ yum install openssl-devel
# =====安装ncurses-devel库========
[root@centos01 ~]$ yum install ncurses-devel
```

### Maven 编译

```bash
[root@centos01 ~]$ mvn package -Pdist,native -DskipTests -Dtar
```

等待漫长时间出现错误，再次执行命令。

编译成功的位置`hadoop-dist/target`