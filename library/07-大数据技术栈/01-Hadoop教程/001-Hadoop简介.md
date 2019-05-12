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

有HDFS，MapReduce，Yarn组成。

### Hadoop的发行版本

- Apache 版本，原始版本
- Cloudera Hadoop版本，企业用的多
- Hortonworks Hadoop版本，文档完善

### Hadoop的优势

- 高可靠性：底层维护（默认3）个数据副本，即使节点故障也不会丢失数据
- 高扩展性：在集群间分配任务数据，可方便扩展数以千计的节点
- 高效性：在MapReduce的思想下，Hadoop是并行运行的，以加快处理速度
- 高容错性：能够自动将失败的任务重新分配

### Hadoop 目录结构

- /bin：存放命令
- /etc/hadoop：配置文件信息
- /sbin：启动关闭集群等命令
- /share：手册

## 单机部署方式

Hadoop 集群可以运行的 3 个模式有：

- 单机模式
- 伪分布模式
- 全分布模式

默认情况下，Hadoop被配置成以非分布式模式运行的一个独立Java进程，这对调试非常有帮助。

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

## 相关问题

- 简述Hadoop平台的发展过程
- 简述Hadoop名称及技术来源
- 简述Hadoop的体系结构
- 简述MapReduce的体系结构
- 简述HDFS和MapReduce在Hadoop中的角色