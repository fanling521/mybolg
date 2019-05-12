# Hive 简介

## 什么是Hive

Hive由`Facebook`开源用于解决海量结构化日志的数据统计，是基于`Hadoop`的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能，本质是：将HQL转化成`MapReduce`程序。

需要知道的是：

- Hive处理的数据存储在`HDFS`
- Hive分析数据底层的实现是`MapReduce`
- 执行程序运行在`Yarn`上

通俗的说，`Hive`存的是和`HDFS`的映射关系，`Hive`是逻辑上的数据仓库，实际操作的都是`HDFS`上的文件，HQL就是用SQL语法来写的MR程序。

### Hive的架构原理

1. **用户接口**：Client CLI（hive shell）、JDBC/ODBC、HWI（浏览器访问hive）
2. **元数据**：Metastore元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等，默认存储在自带的`derby`数据库中，推荐使用`MySQL`存储Metastore
3. 使用**HDFS**进行存储，使用MapReduce进行计算。
4. **驱动器**：Driver
   1. **解析器**（SQL Parser）：将SQL字符串转换成抽象语法树AST，对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。
   2. **编译器**（Physical Plan）：将AST编译生成逻辑执行计划。
   3. **优化器**（Query Optimizer）：对逻辑执行计划进行优化。
   4. **执行器**（Execution）：把逻辑执行计划转换成可以运行的物理计划，对于Hive来说，就是MR/Spark。

### Hive和关系型数据库的比较

`Hive`为数据仓库而设计，不是数据库。

1. 数据库使用SQL，`Hive`使用类SQL的HQL
2. 数据库的数据存储在本地文件系统上，`Hive`数据存储在`HDFS`上
3. 数据库可对数据进行更新，`Hive`数据在加载的时候都是确定的，不建议更新数据
4. 数据库有自己的引擎，`Hive`是通过`MapReduce`实现

## 相关问题

（1）Hive的主要作用是什么？

（2）Hive的体系结构是什么？