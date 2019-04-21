# Sqoop教程

## Sqoop简介

Sqoop是一款开源的工具，主要用于在`Hadoop`与传统的数据库间进行数据的传递，可以将一个关系型数据库中的数据导进到`Hadoop`的`HDFS`中，也可以将`HDFS`的数据导进到关系型数据库中。

## Sqoop原理

将导入或导出命令翻译成`MapReduce`程序来实现。

## Sqoop安装

> （1）解压资源包

> （2）配置文件

```bash
[fanl@centos7 sqoop-1.4.6-cdh5.14.2]$ mv conf/sqoop-env-template.sh conf/sqoop-env.sh
[fanl@centos7 sqoop-1.4.6-cdh5.14.2]$ vi conf/sqoop-env.sh
```

修改以下的几个参数

```bash
#Set path to where bin/hadoop is available
export HADOOP_COMMON_HOME=/opt/modules/cdh5.14.2/hadoop-2.6.0-cdh5.14.2

#Set path to where hadoop-*-core.jar is available
export HADOOP_MAPRED_HOME=/opt/modules/cdh5.14.2/hadoop-2.6.0-cdh5.14.2

#set the path to where bin/hbase is available
#export HBASE_HOME=

#Set the path to where bin/hive is available
export HIVE_HOME=/opt/modules/cdh5.14.2/hive-1.1.0-cdh5.14.2
```

> （3）配置JSON解析包和MySQL驱动包

1. 需要下载`json-java.jar`到安装目录的`lib`目录下，[参考下载地址](http://central.maven.org/maven2/org/json/json/20180813/)
2. 拷贝MySQL驱动包到安装目录的`lib`目录下

> （4）使用Sqoop

```bash
[fanl@centos7 sqoop-1.4.6-cdh5.14.2]$ bin/sqoop help
```

> （5）连接数据库测试

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

### 导入数据

从非大数据集群向大数据集群传输数据叫导入，用import。

#### 从关系型数据库到HDFS

（1）在MySQL新建相关数据

```bash
mysql> create database company;
Query OK, 1 row affected (0.00 sec)

mysql> use company;
Database changed
mysql> create table emp(
    -> id int primary key auto_increment not null,
    -> name varchar(50),
    -> sex varchar(50)
    -> );
Query OK, 0 rows affected (0.11 sec)

mysql> select * from emp;
+----+------+------+
| id | name | sex  |
+----+------+------+
|  1 | xg   | f    |
|  2 | xm   | f    |
|  3 | xl   | f    |
|  4 | xh   | m    |
+----+------+------+
4 rows in set (0.00 sec)
```

（2）全部导入到HDFS上

```bash
[fanl@centos7 sqoop-1.4.6-cdh5.14.2]$ bin/sqoop import \
> --connect jdbc:mysql://centos7:3306/company \
> --username root \
> --password 123456 \
> --table emp \
> --target-dir /user/fanl/company/emp1 \
> --delete-target-dir \
> --num-mappers 1 \
> --fields-terminated-by '\t
# 如何指定列
> --columns id,sex \
## 如何指定查询，去掉--table
> --query 'select name,sex from emp where id <=1 and $CONDITIONS;'
```

### 导出数据

从大数据集群向非大数据集群传输数据叫导入，用export。

#### Hive表导出到MySQL

注意：表不存在不会创建

使用脚本导出，命令为`$ bin/sqoop --options-file xx.opt`

（1）编写`.opt`脚本

```bas
[fanl@centos7 data]$ vi sqoop-export-mysql.opt
export
--connect
jdbc:mysql://centos7:3306/company
--username
root
--password
123456
--table
dept
--num-mappers
1
--export-dir
/user/hive/warehouse/dept
--input-fields-terminated-by
'\t'
```

（2）执行脚本

```bash
[fanl@centos7 sqoop-1.4.6-cdh5.14.2]$ bin/sqoop --option-file /home/fanl/data/sqoop-export-mysql.opt
```

注意：其他用法/命令根据实际需求参考文档和help。