# HDFS 教程

## 什么是HDFS

HDFS是分布式文件系统（**H**adoop **D**istributed **F**ile **S**ystem），用于存储文件，通过目录树来定位文件。适合一次写入，多次读出的场景，不支持修改文件。

**HDFS的架构**

![HDFS架构](assets/20190326142740.png)

HDFS中的读/写操作运行在块级。HDFS数据文件被分成块大小的块，这是作为独立的单元存储。默认块大小为**128** MB。

HDFS的块比磁盘块大，其目的是为了最小化寻址开销。如果块设置得足够大，从磁盘传输数据的时间可以明显大于定位这个块开始位置所需的时间。这样，传输一个由多个块组成的文件的时间取决于磁盘传输速率。 

- 块太小，增加寻址时间；块太大，增加内存消耗
- 块的大小取决于磁盘的传输速率

**优点**：

- 高容错性：自动保存数据副本
- 适合处理大数据且可部署在集群服务器上，通过多级副本机制提高可靠性

**缺点**：

- 不适合做延时数据访问
- 对小文件存储不友好，且违反了设计目标
- 一个文件不允许多线程同时写，不支持文件随机修改，只能追加

## HDFS的基本结构

HDFS由NameNode、DataNode和SecondaryNameNode组成。

### NameNode是什么

NameNode可以被认为是系统的主站（Master）。它维护所有系统中存在的文件和目录的文件系统树和元数据。

- 管理HDFS的名称空间
- 配置副本策略
- 管理数据库的映射信息
- 处理客户端的读写请求

### DataNode是什么

DataNode是Slave，接收NameNode命令并执行操作。

- 存储实际的数据块
- 执行数据的读写操作

### Client是什么

Client是客户端，主要的功能：

- 文件切割，讲文件分割成一个一个的块，然后上传
- 与NameNode交互，获取文件的位置信息
- 与DataNode交互，进行读写操作
- 提供一些命令管理和操作HDFS

### SecondaryNameNode是什么

- 分担NameNode 的工作
- 紧急情况下，回复NameNode

## HDFS Shell操作

### 基本语法

`bin/hadoop fs [具体命令]`或者`bin/hdfs dfs [具体命令]`，dfs是fs的实现类。

### 常用命令

（1）-ls: 显示目录信息

```bash
[fanl@centos7 ~]$ hdfs dfs -ls /
=========================================
Found 3 items
drwxr-xr-x   - fanl supergroup          0 2019-03-26 18:26 /opt
drwx------   - fanl supergroup          0 2019-03-26 18:14 /tmp
drwxr-xr-x   - fanl supergroup          0 2019-03-26 18:29 /user

```

（2）-mkdir：在HDFS上创建目录

```bash
[fanl@centos7 ~]$ hdfs dfs -mkdir -p /user/fanl/a/aa
```

（3）-moveFromLocal：从本地剪切粘贴到HDFS

```bash
[fanl@centos7 ~]$ 
```

（4）-appendToFile：追加一个文件到已经存在的文件末尾

```bash
[fanl@centos7 ~]$ 
```

（5）-cat：显示文件内容

```bash
[fanl@centos7 ~]$ hdfs dfs -cat /user/fanl/myfile/LeagueClientSettings.yaml
```

（6）-chgrp 、-chmod、-chown：Linux文件系统中的用法一样，修改文件所属权限

```bash
[fanl@centos7 ~]$ 
```

（9）-cp ：从HDFS的一个路径拷贝到HDFS的另一个路径

```bash
[fanl@centos7 ~]$ [fanl@centos7 ~]$ hdfs dfs -cp \ 
> /user/fanl/myfile/LeagueClientSettings.yaml \
> /user/fanl
```

（10）-mv：在HDFS目录中移动文件

```bash
[fanl@centos7 ~]$ 
```

（11）-get（copyToLocal）：等同于copyToLocal，就是从HDFS下载文件到本地

```bash
[fanl@centos7 ~]$ 
```

（12）-getmerge：合并下载多个文件

```bash
[fanl@centos7 ~]$ 
```

（13）-put（copyFromLocal）：从本地文件系统中拷贝文件到HDFS路径去

```bash
[fanl@centos7 ~]$ 
```

（14）-tail：显示一个文件的末尾

```bash
[fanl@centos7 ~]$ hdfs dfs -tail /user/fanl/myfile/LeagueClientSettings.yaml
```

（15）-rm：删除文件或文件夹

```bash
[fanl@centos7 ~]$ hdfs dfs -rm -f -r /user/fanl/a
=======================================================
19/03/27 18:53:17 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 0 minutes, Emptier interval = 0 minutes.
Deleted /user/fanl/a
```

（16）-rmdir：删除**空目录**

```bash
[fanl@centos7 ~]$ hdfs dfs -rmdir /user/fanl/a/aa
```

（17）-du：统计文件夹的大小信息

```bash
[fanl@centos7 ~]$ hdfs dfs -du /user/fanl
==========================================
1055  /user/fanl/myfile
```

（18）-setrep：设置HDFS中某文件的副本数量

```bash
[fanl@centos7 ~]$ hdfs dfs -setrep 4 /user/fanl/myfile/LeagueClientSettings.yaml
============================================
Replication 4 set: /user/fanl/myfile/LeagueClientSettings.yaml
```

## HDFS客户端开发

### 环境准备

下载Windows版本的Hadoop压缩包，解压，配置环境变量`HADOOP_HOME`

新建maven工程

```xml
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-common</artifactId>
	<version>${hadoop-version}</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-client</artifactId>
	<version>${hadoop-version}</version>
</dependency>
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-hdfs</artifactId>
	<version>${hadoop-version}</version>
</dependency>
```

### HDFS的API

配置文件的优先级：（1）客户端代码中设置的值 >（2）ClassPath下的用户自定义配置文件 >（3）然后是服务器的默认配置

```java
//配置文件
Configuration configuration = new Configuration();
```

实例化文件系统

```java
FileSystem fs=FileSystem.get(new URI("hdfs://centos7:9000"),conf,"fanl");
```

一些操作就可以查看对象`fs`中方法。

定位读取文件

```java
//##########第一块############
//输入流
FSDataInputStream fis=fs.open(xxx);
//输出流
FileOutputStream fos=new FileOutputStream(xxx);
//每次读取1024B
byte[] buf = new byte[1024];	
for(int i =0 ; i < 1024 * 128; i++){
	fis.read(buf);
	fos.write(buf);
}
//##########第二块############
// 定位到128M
fis.seek(1024*1024*128)
```

## HDFS的数据流

### HDFS写流程

1. 客户端通过`Distributed FileSystem`模块向NameNode请求写文件，NameNode检查目标文件，父目录是否存在
2. NameNode返回是否可以上传文件
3. 客户端请求第一个 Block上传到哪几个DataNode服务器上，NameNode返回DataNode列表
4. 客户端通过`FSDataOutputStream`模块请求dn1上传数据，dn1收到请求会继续调用dn2，逐个建立管道，然后逐级应答客户端
5. 客户端按**128MB**的块切分文件，发送第一块，向dn1以packet为单位传输，dn1收到就会传给dn2，依次传输，逐级应答
6. 每写完一个块，返回确认信息，客户端会再次请求，重复执行4-5。
7. 全部写完，关闭输入输出流，发生完成信号给NameNode

#### 机架感知（>2.7.2）

第一个副本在client所处的节点上。如果客户端在集群外，随机选一个。
第二个副本和第一个副本位于相同机架，随机节点。
第三个副本位于不同机架，随机节点。

### HDFS读流程

1. 客户端通过`Distributed FileSystem`模块请求NameNode下载文件，NameNode通过查询元数据，找到对应的DataNode地址
2. 客户端通过`FSDataInputtStream`模块请求DataNode数据读取
3. DataNode传输数据给客户端，客户端以packet接收缓存，然后写入目标文件

## NameNode

### NN和2NN工作机制

首先，我们做个假设，如果存储在`NameNode`节点的磁盘中，因为经常需要进行随机访问，还有响应客户端请求，必然是效率过低。因此，元数据需要存放在内存中。但如果只存在内存中，一旦断电，元数据丢失，整个集群就无法工作了。因此产生在磁盘中备份元数据的`FsImage`。

这样又会带来新的问题，当在内存中的元数据更新时，如果同时更新`FsImage`，就会导致效率过低，但如果不更新，就会发生一致性问题，一旦`NameNode`节点断电，就会产生数据丢失。因此，引入`Edits`文件(只进行追加操作，效率很高)。每当元数据有更新或者添加元数据时，修改内存中的元数据并追加到`Edits`中。这样，一旦`NameNode`节点断电，可以通过`FsImage`和`Edits`的合并，合成元数据。

但是，如果长时间添加数据到`Edits`中，会导致该文件数据过大，效率降低，而且一旦断电，恢复元数据需要的时间过长。因此，需要定期进行`FsImage`和`Edits`的合并，如果这个操作由`NameNode`节点完成，又会效率过低。因此，引入一个新的节点`SecondaryNamenode`，专门用于`FsImage`和`Edits`的合并。

#### NN启动过程

1. 第一次启动NameNode格式化后，创建fsimage和edits文件，如果不是第一次启动，先滚动Edits并生成一个空的edits.inprogress，然后加载编辑日志和镜像文件到内存。
2. 客户端发起对元数据进行**增删改**的请求（查询元数据的操作不会被记录在Edits中，因为查询操作不会更改元数据信息）
3. NameNode编辑日志先记录操作日志，更新滚动日志
4. NameNode在内存中对数据进行增删改

#### 2NN工作过程

1. 询问NameNode是否需要CheckPoint（默认1小时，或者每分钟检查操作数达到一定数值）。直接带回NameNode是否检查结果
2. NameNode将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode
3. Secondary NameNode加载编辑日志和镜像文件到内存，并合并，生成新的镜像文件fsimage.chkpoint
4. 生成新的镜像文件fsimage.chkpoint，NameNode将fsimage.chkpoint重新命名成fsimage

 **oiv**查看Fsimage文件

```bash
[fanl@centos7 current]$ hdfs oiv -p XML -i fsimage_0000000000000000437 -o ./xx.xml
```

**oev**查看edits文件

```bash
[fanl@centos7 current]$ hdfs oev -p XML -i edits_0000000000000000438-0000000000000000439 -o ./xx2.xml
```

### NameNode故障处理

**方法1**：将SecondaryNameNode中数据拷贝到NameNode存储数据的目录

**方法2**：使用`-importCheckpoint`选项启动NameNode守护进程，从而将SecondaryNameNode中数据拷贝到NameNode目录中。

### NameNode多目录

NameNode的本地目录可以配置成多个，且每个目录存放内容相同，增加了可靠性。

在`hdfs-site.xml`中配置

```xml
<property>
    <name>dfs.namenode.name.dir</name>
<value>file:///${hadoop.tmp.dir}/dfs/name1,file:///${hadoop.tmp.dir}/dfs/name2</value>
```

删除data和log，重新格式化NameNode

### 安全模式

NameNode启动时，首先将镜像文件（Fsimage）载入内存，并执行编辑日志（Edits）中的各项操作。一旦在内存中成功建立文件系统元数据的映像，则创建一个新的Fsimage文件和一个空的编辑日志。此时，NameNode开始监听DataNode请求。这个过程期间，NameNode一直运行在安全模式，即NameNode的文件系统对于客户端来说是只读的。

系统中的数据块的位置并不是由NameNode维护的，而是以块列表的形式存储在DataNode中。在系统的正常操作期间，NameNode会在内存中保留所有块位置的映射信息。在安全模式下，各个DataNode会向NameNode发送最新的块列表信息，NameNode了解到足够多的块位置信息之后，即可高效运行文件系统。

如果满足“最小副本条件”，NameNode会在30秒钟之后就退出安全模式。所谓的最小副本条件指的是在整个文件系统中99.9%的块满足最小副本级别（默认值：dfs.replication.min=1）。在启动一个刚刚格式化的HDFS集群时，因为系统中还没有任何块，所以NameNode不会进入安全模式。

（1）`hdfs dfsadmin -safemode get`        （功能描述：查看安全模式状态）

（2）`hdfs dfsadmin -safemode enter`    （功能描述：进入安全模式状态）

（3）`hdfs dfsadmin -safemode leave`     （功能描述：离开安全模式状态）

（4）`hdfs dfsadmin -safemode wait`      （功能描述：等待安全模式状态）

## DataNode

### 工作机制

1. 一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。
2. DataNode启动后向NameNode注册，通过后，周期性（1小时）的向NameNode上报所有的块信息。
3. 心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个DataNode的心跳，则认为该节点不可用。
4. 集群运行中可以安全加入和退出一些机器。

### 数据完整性

DataNode节点保证数据完整性的方法：

1. 当DataNode读取Block的时候，它会计算CheckSum。
2. 如果计算后的CheckSum，与Block创建时值不一样，说明Block已经损坏。
3. Client读取其他DataNode上的Block。
4. DataNode在其文件创建后周期验证CheckSum

