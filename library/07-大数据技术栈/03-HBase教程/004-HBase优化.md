# HBase优化

## HBase表的设计

### 表的预分区的设计

默认建表时不进行预分区则表的region只有1个，并且该region没有start key和 end key，该region持有的数据量达到一定的阈值则会被动split分割。

以上会导致的问题：

1. 大量的读写操作都集中在HBase集群上某1台或几台RegionServer节点上
2. region的被动split分割会消耗集群的资源

**解决方案**

（1）创建表预分区

```bash
create 't11', 'f1', SPLITS => ['10', '20', '30', '40']
```

向表中写入数据时的cell的rowkey值与region的key的范围默认是前缀匹配。

（2）通过文本

```bash
[fanl@centos7 hadoop-2.6.0-cdh5.14.2]$ vi /opt/splits.txt	
create 't12', 'f1', SPLITS_FILE => '/opt/splits.txt'
```

**rowkey设计原则**

1. rowkey必须唯一
2. rowkey的长度官方建议10-100字节之间
3. rowkey设计时要考虑集群在读写数据时避免数据热点产生
4. rowkey的设计要考虑实际的业务场景

**rowkey设计时可以选择的思路** 

1. 生成随机数（通常不会将rowkey全部使用随机数，会拼接一些与业务需求相关的数据，如与业务需求相关的数据+固定长度的随机数字）
2. 翻转字符串

## HBase表的属性

|             |                     |                     |                              |
| ----------- | ------------------- | ------------------- | ---------------------------- |
| NAME        | 列簇名称            | KEEP_DELETED_CELLS  | FALSE，彻底删除前可以被访问  |
| COMPRESSION | Hfile文件的压缩格式 | MIN_VERSIONS        | 最少保留版本数               |
| BLOOMFILTER | 过滤器类型          | DATA_BLOCK_ENCODING | 数据块的编码方式             |
| VERSIONS    | 可保留版本数        | TTL                 | 数据存活期                   |
| BLOCKSIZE   | Hfile数据块的大小   | REPLICATION_SCOPE   | 副本                         |
| BLOCKCACHE  | 数据块读缓存        | IN_MEMORY           | 在BLOCKCACHE中享有较高的等级 |

## HBase压缩

```bash
# 检查Hadoop是否支持压缩
[fanl@centos7 hadoop-2.6.0-cdh5.14.2]$ bin/hadoop checknative
# 检查HBase是否支持引用Hadoop的压缩
[fanl@centos7 hbase-1.2.0-cdh5.14.2]$ bin/hbase --config conf/ org.apache.hadoop.util.NativeLibraryChecker
# 配置支持snappy
# 1. 上传jar
# 2. 
[fanl@centos7 hbase-1.2.0-cdh5.14.2]$ export HBASE_HOME=/opt/modules/cdh5.14.2/hbase-1.2.0-cdh5.14.2
[fanl@centos7 hbase-1.2.0-cdh5.14.2]$ export HADOOP_HOME=/opt/modules/cdh5.14.2/hadoop-2.6.0-cdh5.14.2
[fanl@centos7 hbase-1.2.0-cdh5.14.2]$ mkdir -p $HBASE_HOME/lib/native
[fanl@centos7 hbase-1.2.0-cdh5.14.2]$ ln -s $HADOOP_HOME/lib/native $HBASE_HOME/lib/native/Linux-amd64-64
# 再次检查
snappy:  true /opt/modules/cdh5.14.2/hadoop-2.6.0-cdh5.14.2/lib/native/libsnappy.so.1
```

使用压缩

```bash
create 't13' ,{NAME => 'f1', VERSIONS => 5,COMPRESSION=>'SNAPPY'}  
```

## BLOCKCACHE读缓存

BlockCache读缓存空间的分区分级

（1）**single区**：默认大小为BlockCache_SIZE*25%，被第一次读取则该块中的cell会被加载到BlockCache的此single区中

（2）**multi区**：默认大小为BlockCache_SIZE*50% ，当数据近期被多次读取则会被加载到multi区中

（3）**in-memory区**：默认大小为BlockCache_SIZE*25% ，不会参与LRU缓存淘汰

## HBase表的管理

看具体命令的帮助信息

（1）flush操作

```bash
hbase> flush 'xxx'
```

（2）compaction操作

```bash
hbase> major_compact 'xx'
```

（3）split操作

```bash
hbase> split 'xxx'
# 当一个表的region下的store下的storeFile文件大小小于一个block_size（64kb）时将不能被split分割
```

（4）将一个region移动到其他的RegionServer节点上 

```bash
hbase> move 'xxx'
```

（5）查看负载均衡策略

```bash
hbase> balancer
```

（6）下线region

```bash
hbase> close_region 'xxx'
```

什么情况下需要进行region的手动管理操作

（1）更换RegionServer节点服务器时

先对此台节点上的所有的region进行flush，再使用move将region移动到其他RegionServer节点上 

（2）针对写入及读取比较频繁的在线查询业务场景

写入数据过程中可能会发生大合并及split机制，可能会影响到HBase集群的稳定性

## HMaster HA

防止master单节点故障，Master HA的实现是借助于Zookeeper基于观察者模式监控master状态，一旦active master节点宕机后Zookeeper会第一时间感知并从其他的多个备份master节点（backup-master）中选举出一个master作为后续的active master节点 。

（1）搭建分布式Hadoop和Zookeeper

（2）部署HBase集群

RegionServer服务器主机名或IP，`conf/backup-masters` 里面定义哪些服务器是备用master

```xml
<property>  
	<name>hbase.rootdir</name>  
	<value>hdfs://centos7:8020/hbase</value>
</property>
<property>
	<name>hbase.cluster.distributed</name>
	<value>true</value>
</property>			
<property>
	<name>hbase.zookeeper.quorum</name>
	<value>hadoop01,hadoop02,hadoop03</value>
</property>
<property>
    <!-- 最初的active -->
	<name>hbase.master</name>
	<value>hdfs://hadoop01:60000</value>
</property>
```

地址：http://centos7:60010/master-status

## HBase优化指南

（1）针对客户端的优化

1. 批量写和读，并发和多线程
2. 客户端手动缓存查询结果，及时关闭资源

（2）针对集群资源和表设计

1. 合理设置预分区及Rowkey设计
2. 控制自动`split\major-compact`改为手动地在访问低峰时段进行
3. 合理设置列族的最大支持版本数和存活期
4. 合理设置BlockCache缓存策略
5. 设置RegionServer处理客户端IO请求的线程数
6. 合理配置BlockCache和memstore的可以占用的`RegionServer-jvm-heap`空间的阈值
7. WAL预写日志机制设置
8. 合理使用Compression压缩