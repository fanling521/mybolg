# Zookeeper概述

ZooKeeper是一种开源的、分布式协调服务。

## 分布式部署

在第一台上配置，然后分发修改。

#### 下载解压

下载Linux版本的解压，需要安装JDK，过程略。

### 修改配置文件

**（1）配置zoo.cfg文件**

```bash
[fanl@hadoop01 zookeeper-3.4.5]$ cd conf/
[fanl@hadoop01 conf]$ cp zoo_sample.cfg zoo.cfg
# =================================================
# =========修改========
dataDir=/opt/modules/zookeeper-3.4.5/zkData
#=========添加========
server.1=hadoop01:2888:3888
server.2=hadoop02:2888:3888
server.3=hadoop03:2888:3888
```

**（2）创建目录和myid**

```bash
[fanl@hadoop01 zookeeper-3.4.5]$ mkdir zkData
[fanl@hadoop01 zookeeper-3.4.5]$ touch zkData/myid
```

**（3）分发同步修改**

使用`rsync`同步`/opt/modules`

**（4）修改myid中的值**

hadoop01修改为1，hadoop02修改为2，hadoop03修改为3

**（5）启动**

```bash
# 启动
[fanl@hadoop01 zookeeper-3.4.5]$ bin/zkServer.sh start
# 检查状态
[fanl@hadoop01 zookeeper-3.4.5]$ jps
[fanl@hadoop01 zookeeper-3.4.5]$ bin/zkServer.sh status

```

## 基本命令

```bash
[fanl@hadoop01 zookeeper-3.4.5]$ bin/zkCli.sh -server 主机名:2181
# 或者 
[fanl@hadoop01 zookeeper-3.4.5]$ bin/zkCli.sh
#=================================#
#=====help        查看帮助============
[zk: localhost:2181(CONNECTED) 0] help
#=====quit        退出管理============
[zk: localhost:2181(CONNECTED) 0] quit
#=====ls /        查看根目录============
[zk: localhost:2181(CONNECTED) 0] ls /
[zk: localhost:2181(CONNECTED) 1] ls /zookeeper/quota
#=====create -e 目录 备注        查看帮助============
[zk: localhost:2181(CONNECTED) 2] create -e /user 用户
#=====get 目录        查看目录============
[zk: localhost:2181(CONNECTED) 4] get /user
#=====rmr 目录        删除目录============
[zk: localhost:2181(CONNECTED) 4] rmr /user
```

