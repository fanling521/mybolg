# Zookeeper教程

## 什么是Zookeeper

ZooKeeper是一种开源分布式协调服务，是Hadoop、Kafka、HBase的重要组件。

### Zookeeper数据模型

Zookeeper的结构类似标准的文件系统， 但是没有文件和目录，而是统一使用node的概念，成为Znode。

（1）Znode有唯一的路径标识，可以自动编号和被监控，一旦数据变化，通过设置监控的客户端

（2）Znode可以有字节点，可以存数据，但是`Ephemeral`类型的节点不能有子节点

（3）Znode中的数据可以有很多版本，查询此节点需要带版本

（4）Zookeeper可以有临时节点，客户端和服务器通信采用长连接方式，每个客户端服务器通过心跳来保持连接，这个连接状态成为session

### Zookeeper的特征

（1）Zookeeper的数据模型

（2）Zookeeper的会话，Zk客户端通过句柄建立会话

（3）Zk watches：是一次性触发器，当watch的对象发生改变时，将会触发事件

（4）Zk ACL：是对Znode的访问控制

（5）Zk的一致性保证：Zk读写非常快，读的速度比写更快

### Zookeeper的工作原理

Zk的核心是原子广播，保证了各个server之间的同步，此机制通过Zab协议实现。Zab协议有两种模式：广播模式和恢复模式，当服务启动或领导者崩溃后，进入恢复模式，完成选举，结束恢复模式。

一旦leader和多数的follower进行了状态同步，就开始广播消息了。当新的server加入，会在恢复模式下启动并和leader同步状态，Zk服务一直处于广播模式，直到leader失去大多数的支持。

Zk采用了递增的事务id来处理提议。

**Zk的角色**：leader（负责进行投票发起和决议，更新系统状态）、learner、follower（接受客户端请求和参与投票）、observer（接受请求转发给leader，但不参与投票）

**leader的选举**：集群中，Server群发自己的选票(myid,zxid)，每个Server接受其他选票和自己相比较，若对方结果更大，则更新自己的选票，重复执行，直到产生leader。

## Zookeeper的部署

在第一台上配置，然后分发修改，可以实现集群部署。

### 下载解压

下载Linux版本的解压，需要安装JDK，过程略。

### 修改配置文件

> （1）配置zoo.cfg文件

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

> （2）创建目录和myid

```bash
[fanl@hadoop01 zookeeper-3.4.5]$ mkdir zkData
[fanl@hadoop01 zookeeper-3.4.5]$ touch zkData/myid
```

> （3）分发同步修改

使用`rsync`同步`/opt/modules`

> （4）修改myid中的值

hadoop01修改为1，hadoop02修改为2，hadoop03修改为3

> （5）启动

```bash
# 启动
[fanl@hadoop01 zookeeper-3.4.5]$ bin/zkServer.sh start|stop|restart|status
# 检查状态
[fanl@hadoop01 zookeeper-3.4.5]$ jps
[fanl@hadoop01 zookeeper-3.4.5]$ bin/zkServer.sh status

```

## Zookeeper的Shell操作

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
[zk: localhost:2181(CONNECTED) 1] ls2 /zookeeper/quota
#=====create -e 目录 备注        查看帮助============
[zk: localhost:2181(CONNECTED) 2] create -e /user 用户
#=====get 文件内容 查看目录============
[zk: localhost:2181(CONNECTED) 4] get /user
#=====rmr 目录        删除目录============
[zk: localhost:2181(CONNECTED) 4] rmr /user
```

## Zookeeper API操作

（1）案例：创建会话

首先启动Zk服务，在IDEA中引用相关包，如下

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.6</version>
</dependency>
```

```java
public static void main(String[] args) {
    try {
        zooKeeper = new ZooKeeper("centos7:2181", 500, new CreateSession());
        System.out.println("zk状态：" + zooKeeper.getState());
    } catch (IOException e) {
        e.printStackTrace();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

（2）案例：创建节点

```java
public void createNode() {
        System.out.println("创建节点");
        try {
            //path 节点
            //data 数据
            //权限列表
            //  OPEN_ACL_UNSAFE：完全开放
            //  CREATOR_ALL_ACL：创建该znode的连接拥有所有权限
            //  READ_ACL_UNSAFE：所有的客户端都可读
            //节点类型
            //  PERSISTENT：持久化节点
            //  PERSISTENT_SEQUENTIAL：持久化有序节点
            //  EPHEMERAL：临时节点（连接断开自动删除）
            //  EPHEMERAL_SEQUENTIAL：临时有序节点（连接断开自动删除）
            String path1 = zooKeeper.create("/flnode1", "hello".getBytes(),
                    ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
            System.out.println("创建节点：" + path1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (KeeperException e) {
            e.printStackTrace();
        }
    }
```

（3）案例：删除节点

```java
public void delNode() {
    try {
        //节点
        //版本
        zooKeeper.delete("/flnode", -1);
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (KeeperException e) {
        e.printStackTrace();
    }
}
```

（4）案例：修改节点

```java
public void setZkDate() {
    try {
        //路径
        //数据
        //版本
        Stat stat = zooKeeper.setData("/flnode2", "hello".getBytes(), -1);
        System.out.println(stat);
    } catch (KeeperException e) {
        e.printStackTrace();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

（5）案例：获取子节点

```java
public void childNode() {
    try {
        List<String> list = zooKeeper.getChildren("/", true);
        list.forEach(x -> System.out.println(x));
    } catch (KeeperException e) {
        e.printStackTrace();
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

