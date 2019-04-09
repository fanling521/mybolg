# Sqoop教程

## Sqoop简介

Sqoop（SQL to Hadoop）是一款开源的工具，主要用于在`Hadoop`与传统的数据库间进行数据的传递，可以将一个关系型数据库中的数据导进到`Hadoop`的`HDFS`中，也可以将`HDFS`的数据导进到关系型数据库中。

## Sqoop原理

将导入或导出命令翻译成`MapReduce`程序来实现。

## Sqoop安装

（1）解压资源包

（2）配置文件

```bash
[fanl@centos7 sqoop-1.4.6-cdh5.14.2]$ mv conf/sqoop-env-template.sh conf/sqoop-env.sh
[fanl@centos7 sqoop-1.4.6-cdh5.14.2]$ vi conf/sqoop-env.sh
```

修改以下的几个参数

```bash
#Set path to where bin/hadoop is available
export HADOOP_COMMON_HOME=/opt/modules/cdh5.14.2/hadoop-cdh5.14.12

#Set path to where hadoop-*-core.jar is available
export HADOOP_MAPRED_HOME=/opt/modules/cdh5.14.2/hadoop-cdh5.14.12

#set the path to where bin/hbase is available
#export HBASE_HOME=

#Set the path to where bin/hive is available
export HIVE_HOME=/opt/modules/cdh5.14.2/hive-1.1.0-cdh5.14.2
```

（3）配置JSON解析包和MySQL驱动包

1. 需要下载`json-java.jar`到安装目录的`lib`目录下，[参考下载地址](http://central.maven.org/maven2/org/json/json/20180813/)
2. 拷贝MySQL驱动包到安装目录的`lib`目录下

（4）使用Sqoop

```bash
[fanl@centos7 sqoop-1.4.6-cdh5.14.2]$ bin/sqoop help
```

（5）连接数据库测试

```bash
[fanl@centos7 sqoop-1.4.6-cdh5.14.2]$ bin/sqoop list-databases \
> --connect 'jdbc:mysql://centos7:3306' \
> --username 'root' \
> --password '123456'

information_schema
metastore
mysql
performance_schema
sys
[fanl@centos7 sqoop-1.4.6-cdh5.14.2]$ 
```

## Sqoop应用

