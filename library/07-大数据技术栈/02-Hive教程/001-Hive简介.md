# Hive教程

## 什么是Hive

Hive由Facebook开源用于解决海量结构化日志的数据统计，是基于`Hadoop`的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并提供类SQL查询功能。

本质是：将HQL转化成MapReduce程序。

- Hive处理的数据存储在`HDFS`
- Hive分析数据底层的实现是`MapReduce`
- 执行程序运行在`Yarn`上

通俗的说，Hive存的是和HDFS的映射关系，Hive是逻辑上的数据仓库，实际操作的都是HDFS上的文件，HQL就是用SQL语法来写的MR程序。

**优点：**

1. 操作接口采用类SQL语法，避免去写MapReduce，减少开发成本
2. Hive的执行延迟较高，用于对实时性要求不高的场合
3. 支持用户自定义函数

**缺点：**

1. HQL表达能力有限，不擅长数据挖掘
2. Hive粒度较粗，调优困难

### 架构原理

1. 用户接口：Client CLI（hive shell）、JDBC/ODBC(java访问hive)、HWI（浏览器访问hive）
2. 元数据：Metastore元数据包括：表名、表所属的数据库（默认是default）、表的拥有者、列/分区字段、表的类型（是否是外部表）、表的数据所在目录等，默认存储在自带的derby数据库中，推荐使用MySQL存储Metastore
3. 使用HDFS进行存储，使用MapReduce进行计算。
4. 驱动器：Driver
   1. **解析器**（SQL Parser）：将SQL字符串转换成抽象语法树AST，这一步一般都用第三方工具库完成，比如antlr；对AST进行语法分析，比如表是否存在、字段是否存在、SQL语义是否有误。
   2. **编译器**（Physical Plan）：将AST编译生成逻辑执行计划。
   3. **优化器**（Query Optimizer）：对逻辑执行计划进行优化。
   4. **执行器**（Execution）：把逻辑执行计划转换成可以运行的物理计划，对于Hive来说，就是MR/Spark。

### Hive和数据库的比较

Hive为数据仓库而设计，不是数据库。

1. 数据库使用SQL，Hive使用类SQL的HQL
2. 数据库的数据存储在本地文件系统上，Hive数据存储在HDFS上
3. 数据库可对数据进行更新，Hive数据在加载的时候都是确定的，不建议更新数据
4. 数据库有自己的引擎，Hive是通过MapReduce实现

## 面试题

（1）Hive的主要作用是什么？

（2）Hive的体系结构是什么？