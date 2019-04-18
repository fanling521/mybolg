# Hadoop的部署

## 伪分布部署

在一个节点上启动不同的进程，模拟多节点

### Hadoop配置

修改`hadoop-env.sh`

```bash
# 修改JAVA_HOME 地址
export JAVA_HOME=/opt/module/jdk1.8.0_201
```

修改 `core-site.xml`

```xml
<configuration>
    <!-- 修改Hadoop的NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <!-- 指定Hadoop运行时候的产生文件的临时存储目录 -->
      <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/module/hadoop-2.7.6/data</value>
    </property>
</configuration>
```

修改`hdfs-site.xml`

```xml
<configuration>
    <!-- 指定hdfs副本的数量，默认值3，保存数据的副本，防止数据丢失 -->
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

### 配置和启动HDFS

第一次启动需要格式化，以后不需要再格式化，需要关闭服务，删除`data`和`logs`文件夹再次格式化。

**原因**：格式化`namenode` 会产生新的集群id，导致和`datanode`不一致。

```bash
# 格式化namenode	
$ hdfs namenode -format
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

查看控制面板，端口号为**50070**

![面板](assets/20190308194111.png)

*注意*：**若打不开，请查看防火墙**

#### HDFS的简单操作

**例1：创建文件夹**

```bash
# 创建文件夹
$ hdfs dfs -mkdir -p /user/fanl/input
# 创建完查看目录，已经存在路径，和linux命令是一致的 其中，bin/hdfs dfs 是固定的写法
```

![控制板](assets/20190308195627.png)

**例2：复制本地文件到HDFS**

```bash
$ hdfs dfs -put input/fanl.txt /user/fanl/input
```

**例3：执行wordcount**

```bash
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wordcount /user/fanl/input/ /user/fanl/output
# 查看结果
$ hdfs dfs -cat /user/fanl/output/*
```

### 配置和启动YARN

#### 配置YARN

修改`yarn-env.sh`

```bash
# 修改JAVA_HOME地址
export JAVA_HOME=/opt/module/jdk1.8.0_201
```

修改`yarn-site.xml`

```xml
<configuration>
    <!-- MapReduce获取数据的方式 -->
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <!-- YARN的resourcemanager的地址 -->
     <property>
        <name>yarn.resourcemanager.hoastname</name>
        <value>fanl01</value>
    </property>
</configuration>
```

#### 配置MapReduce

修改`mapred-env.sh`

```bash
# 修改JAVA_HOME地址
export JAVA_HOME=/opt/module/jdk1.8.0_201
```

修改`mapred-site.xml`

```xml
<configuration>
    <!-- 指定MR运行在YARN上 -->
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

查看控制面板，端口号为 **8088**

```bash
$ hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wordcount /user/fanl/input/ /user/fanl/output
# 并且查看控制面板
```

### 配置和启动历史服务器

修改`mapred-site.xml`

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

启动历史服务器`historyserver`

```bash
$  sbin/mr-jobhistory-daemon.sh start historyserver
```

#### 配置日志聚集

修改`yarn-site.xml`，若启动中，需要重启**yarn**和**historyserver**。

```xml
<configuration>
    <!-- 开启日志聚集 -->
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <!-- 日志聚集保留时间 30天,实际可以是180 -->
     <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>2592000</value>
    </property>
</configuration>
```

### 启动和关闭脚本编写

#### 开启伪分布进程

```bash
#!/bin/bash
# 启动hadoop
echo ++++ 启动Hadoop伪分布进程 ++++
echo -----------------------------------
echo Hadoop安装目录： $HADOOP_HOME
$HADOOP_HOME/sbin/hadoop-daemon.sh start namenode
$HADOOP_HOME/sbin/hadoop-daemon.sh start datanode
$HADOOP_HOME/sbin/yarn-daemon.sh start resourcemanager
$HADOOP_HOME/sbin/yarn-daemon.sh start nodemanager
$HADOOP_HOME/sbin/mr-jobhistory-daemon.sh start historyserver
jps
```

#### 关闭伪分布进程

```bash
#!/bin/bash
# 关闭hadoop
echo ++++ 关闭Hadoop伪分布进程 ++++
echo -----------------------------------
echo Hadoop安装目录： $HADOOP_HOME
$HADOOP_HOME/sbin/hadoop-daemon.sh stop namenode
$HADOOP_HOME/sbin/hadoop-daemon.sh stop datanode
$HADOOP_HOME/sbin/yarn-daemon.sh stop resourcemanager
$HADOOP_HOME/sbin/yarn-daemon.sh stop nodemanager
$HADOOP_HOME/sbin/mr-jobhistory-daemon.sh stop historyserver
jps
```

伪分布部署搭建完成，可以使用了。
## 完全分布式部署
### 分发脚本的编写

实现各个服务器于主服务器文件的同步，注意文件夹的权限问题。

```bash
[fanl@hadoop01 ~]$ mkdir bin
[fanl@hadoop01 ~]$ cd bin/
[fanl@hadoop01 bin]$ touch mysync.sh
```

`mysync.sh`脚本：

```bash
#! /bin/bash
# 获取输入的参数，没有参数，直接退出
if [ $# == 0 ];then
   echo "请输需要同步的目录（绝对路径）"
else
   echo "同步的目录为"$1
   user=`whoami`
   echo "当前操作用户："${user}
   # 循环传输文件
   for((i=2;i<=3;i++));do
      echo "=======Hadoop-HOST${i}============"
      rsync -rvl $1 ${user}@hadoop0${i}:$1
   done
   echo "完成！请检查。"
fi
```

### 集群配置和启动

#### 集群规划

假设的3台服务器按照如下规划Hadoop的部署。

| 组件 | hadoop01             | hadoop02        | hadoop03                       |
| ---- | -------------------- | --------------- | ------------------------------ |
| HDFS | NameNode<br>DataNode | DataNode        | SecondaryNameNode<br/>DataNode |
| YARN | NodeManager          | ResourceManager | NodeManager                    |

#### 集群配置

##### 核心文件配置

首先在主服务器上需要修改`*-env.sh`的`JDK`的环境变量。

```bash
[fanl@hadoop02 hadoop-2.7.6]$ vi hadoop-env.sh
[fanl@hadoop02 hadoop-2.7.6]$ vi yarn-env.sh
[fanl@hadoop02 hadoop-2.7.6]$ vi mapred-env.sh
```

然后修改各个配置文件

- core-site.xml
- hdfs-site.xml
- yarn-site.xml

###### Hadoop配置

core-site.xml

```xml
<configuration>
    <!-- 修改Hadoop的NameNode的地址 -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://hadoop01:9000</value>
    </property>
    <!-- 指定Hadoop运行时候的产生文件的临时存储目录 -->
      <property>
        <name>hadoop.tmp.dir</name>
        <value>/opt/modules/hadoop-2.7.6/data</value>
    </property>
</configuration>
```

###### HDFS配置

hdfs-site.xml

```xml
<configuration>
    <!-- 指定hdfs副本的数量，默认值3，保存数据的副本，防止数据丢失 -->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <!-- 指定SecondaryNameNode的节点 -->
     <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>hadoop03:50090</value>
     </property>
    <!-- namenode的web访问主机名:端口号 -->
    <property>
        <name>dfs.namenode.http-address</name>
        <value>hadoop03:50070</value>
	</property>
    <!-- 关闭权限检查用户或用户组 -->
	<property>
        <name>dfs.permissions.enabled</name>
        <value>false</value>
	</property>
</configuration>
```

###### YARN的配置

yarn-site.xml

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
        <value>hadoop02</value>
    </property>
    <!-- 启用日志聚合 -->
	<property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
	<!-- 日志保存时间 -->
	<property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>86400</value>
    </property>
</configuration>
```

mapred-site.xml

```xml
<configuration>
    <!-- 指定MR运行在YARN上 -->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <!-- 历史服务器地址 -->
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>hadoop01:10020</value>
    </property>
    <!-- 历史服务器地址WEB地址 -->
     <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>hadoop01:19888</value>
    </property>
</configuration>
```

**修改集群配置**

修改配置文件 `etc/haddop/slaves` 存放DataNode节点

```
# 注意：增加主机信息 不可有空格
hadoop01
hadoop02
hadoop03
```

##### 集群群起

首先检查是否删除了临时文件和日志信息。然后将修改的内容分发到其他服务器，**并且执行NameNode格式化**

```bash
[fanl@hadoop01 hadoop-2.7.6]$ hdfs namenode -format
```

群起节点

```bash
#===============================================================
# 群起HDFS
[fanl@hadoop01 hadoop-2.7.6]$ sbin/start-dfs.sh
# 群起YARN,必须在规划RM的节点启动
[fanl@hadoop02 hadoop-2.7.6]$ sbin/start-yarn.sh
# 启动日志服务器
[fanl@hadoop02 hadoop-2.7.6]$ sbin/mr-jobhistory-daemon.sh start historyserver
```

##### 集群测试

```bash
[fanl@hadoop02 hadoop-2.7.6]$ bin/hdfs dfs /opt/software/jdk.tar.gz /
```

上传一个大文件和大于128M的文件，观察其存储位置和大小，并且尝试拼接块。

![](assets/20190313183152.png)

实际存储的路径

![](assets/20190313183739.png)

尝试将标记的块合并到一个文件中，并且解压，会发现此文件正是之前上传的大文件。

##### SSH免密码登录

查看Linux配置篇

##### 集群时间同步

配置第一台集群节点为事件同步的服务器，下载安装`ntp`

```bash
[fanl@hadoop01 opt]$ sudo yum install ntp
# ========修改配置文件=============
[root@hadoop01 opt]$ vi /etc/ntp.conf
# 取消注释，修改为自己的网段
restrict 192.168.157.0 mask 255.255.255.0 nomodify notrap
# 注释自带的事件同步地址
# 添加以下信息
server 127.127.1.0
fudge 127.127.1.0  stratum 10
# 启动前，先同步时间
[fanl@hadoop01 opt]$ ntpdate cn.pool.ntp.org
# 启动时间服务，设置开机启动
[fanl@hadoop01 opt]$ service ntpd start
[fanl@hadoop01 opt]$ chkconfig ntpd on
# 启动服务器，只需要安装ntp就可以，不需要配置，设置定时服务器获取最新时间
# 不再说明，如何设置定时服务
[fanl@hadoop02 opt]$ sudo ntpdate hadoop01
```

完全分布式部署完成，遇到的问题，查看ssh登录、防火墙等问题。

## 相关问题

（1）Hadoop中需要配置哪些文件，其作用是什么？

（2）Hadoop集群可以运行的3个模式

