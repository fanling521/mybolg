# HBase简介

## 什么是HBase

HBase是一个高可靠性、高性能、面向列、可伸缩的分布式存储系统，利用HBase技术可在廉价PC Server上搭建起大规模结构化存储集群。

## HBase的架构

![架构](assets/20190418110129.png)

## HBase中的角色

（1）Client：HBase的接口信息

（2）Master：监控管理RegionServer，处理元数据的变更，处理region的分配和转移

（3）Region Server：维护Master分配的Region，处理IO请求，切分Region

（4）Zookeeper：保证Master的运行，Region的寻址入口，监控region server的状态，存储HBase的元数据

## HBase的安装

（1）安装Zookeeper

（2）修改`hbase-env.sh`

```bash
export JAVA_HOME=/opt/modules/jdk1.8.0_201
# 不使用内置的zk管理
export HBASE_MANAGES_ZK=false
```

（3）修改`hbase-site.xml`

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
		<value>centos7</value>
	</property>
```

（4）修改`regionservers` 

```
centos7
```

（5）启动HBase

```bash
[fanl@centos7 hbase-1.2.0-cdh5.14.2]$ bin/hbase-daemon.sh start master	
# 在所有从服务器上启动regionserver
[fanl@centos7 hbase-1.2.0-cdh5.14.2]$ bin/hbase-daemon.sh start regionserver
# 集群启动
# HBase启动需要Zookeeper的支持！！
[fanl@centos7 hbase-1.2.0-cdh5.14.2]$ bin/start-hbase.sh
# 访问web控制台
http://centos7:60010
# 进入shell
[fanl@centos7 hbase-1.2.0-cdh5.14.2]$ bin/hbase shell
```

## HBase Shell操作

### 基本操作

（1）创建表

```bash
# 创建命名空间
hbase(main):003:0> create_namespace 'ns1'
# 查看命名空间
hbase(main):004:0> list_namespace
# 在ns1下创建表t1,有一个列族info,历史版本可存5
hbase(main):005:0> create 'ns1:t1',{NAME=>'info',VERSIONS=>5}\
# 描述一张表
hbase(main):007:0> desc 'ns1:t1'
# 创建一张不同列族，和不同保存版本的表
hbase(main):010:0> create 'ns1:t2',{NAME=>'f1',VERSIONS=>2},{NAME=>'f2',VERSIONS=>4},{NAME=>'f3'}
# 创建表，默认版本数，可以缩写
hbase(main):011:0> create 'ns1:t3','f1','f2'
# 列出表信息
hbase(main):012:0>list
```

（2）put插入数据和更新数据

```bash
hbase(main):013:0> create 'ns1:emp',{NAME=>'info',VERSIONS=>5}
# 表名-row_key-列族和列名-值
hbase(main):014:0> put 'ns1:emp','10010','info:name','zhangsan'
```

（3）scan全表扫描

```bash
# 查询整个表
hbase(main):025:0> scan 'ns1:emp'
# 查询某个列族下的数据
hbase(main):026:0> scan 'ns1:emp',{COLUMNS=>'info'}
# 查询某个列的数据，多列用[]
hbase(main):028:0> scan 'ns1:emp',{COLUMNS=>'info:name'}
hbase(main):028:0> scan 'ns1:emp',{COLUMNS=>['info:name','info:age']}
# 查询前多少行的数据
hbase(main):031:0> scan 'ns1:emp',{LIMIT=>2}
# 查询指定行范围的数据
hbase(main):033:0> scan 'ns1:emp',{STARTROW=>'10011',STOPROW=>'10012'}
```

（4）get查询

```bash
# 查询某行
hbase(main):036:0> get 'ns1:emp','10010'
# 查询某行某列族的数据
hbase(main):038:0> get 'ns1:emp','10010'
# 查询某行某个单元格的数据
hbase(main):039:0> get 'ns1:emp','10010',{COLUMNS=>'info:name'}
# 按时间戳查询指定版本
hbase(main):040:0> get 'ns1:emp','10010',{COLUMNS=>'info:name',TIMESTAMP=>1555579957502}
# 按VERSIONS查询指定版本范围内的，比如3，查询的是1，2，3的版本的值
hbase(main):047:0> get 'ns1:emp','10010',{COLUMNS=>'info:name',VERSIONS=>2}
```

（5）delete 和deleteall

```bash
# delete 删除的是value
# 删除指定时间版本的数据
hbase(main):053:0> delete 'ns1:emp','10010','info:name',1555579957502
# 不指定时间就是删除最新版本
hbase(main):054:0> delete 'ns1:emp','10010','info:name'
# delete 删除的是行或者单元格内容
hbase(main):056:0> deleteall 'ns1:emp','10010'
```

### 实际需求操作

（1）将查询的结果导出到一个文件中

```bash
# 文件要先存在
[root@slave004] echo "scan 'kjt_patent',{LIMIT=>1}" | hbase shell > /opt/fanl/hbase.txt
```

但是结果中存在中文，无法显示中文信息，需要进行转码（HBase没有数据类型，数据均以字节存储）

（2）部署问题

1. 不使用部署的zk，将hbase.cluster.distributed改为true
2. 无法访问namenode，检查配置信息，是否地址不一致，比如：core-site配置了localhost，hbase-site配置了主机名。这样会导致无法访问。