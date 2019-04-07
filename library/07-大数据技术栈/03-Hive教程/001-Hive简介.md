# Hive教程

## 什么是Hive

Hive由Facebook开源用于解决海量结构化日志的数据统计，基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。

本质是：将HQL转化成MapReduce程序。

- Hive处理的数据存储在HDFS
- Hive分析数据底层的实现是MapReduce
- 执行程序运行在Yarn上

## Hive数据仓库的操作

### 基本操作

```sql
-- 创建数据库
hive> create database if not exists [名字];
-- 删除数据库
hive> drop database [xx] [cascade 删除空数据库 需要加上];
-- 创建表
hive> create table student(id int,name string) row format delimited fields terminated by '\t';
-- 删除表
hive> drop table xx;
-- 清空表
hive> truncate table xx;
```

### 测试加载数据

```bash
# local可选，local加上去表示从本地加载数据，否则从HDFS上加载数据
# 注意引号，以及全路径
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ data [local] inpath '路径' into table student;
```

### 查看与修改

```sql
-- 查看表的信息
hive> desc (tablename);
hive> desc extended (tablename);
hive> desc foratted (tablename);
-- 查看内置函数信息
hive> show functions;
hive> desc function xxx;
hive> desc function extended xxx;
-- 修改表名【格式】
hive> alter table tablename rename to new_tablename;
-- 添加新列【格式】
hive> alter table tablename add columns(field type);
-- 将某列修改【格式】
hive> alter table tablename change xx xx2 type
-- 替换列

```

