# Hadoop伪分布部署

在一个节点上启动不同的进程，模拟多节点

## Hadoop配置

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

## 配置和启动HDFS

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

### HDFS的简单操作

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

## 配置和启动YARN

### 配置YARN

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

### 配置MapReduce

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

## 配置和启动历史服务器

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

### 配置日志聚集

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

## 启动和关闭脚本编写

### 开启伪分布进程

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

### 关闭伪分布进程

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