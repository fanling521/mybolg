# Hive数据操作

## Hive的数据类型

### 基本数据类型

| Hive数据类型 | Java数据类型 | Hive数据类型 | Java数据类型 |
| ------------ | ------------ | ------------ | ------------ |
| TINYINT      | byte         | FLOAT        | float        |
| SMALINT      | short        | DOUBLE       | double       |
| INT          | int          | STRING       | string       |
| BIGINT       | long         | TIMESTAMP    | /            |
| BOOLEAN      | boolean      | BINARY       | /            |

对于Hive的String类型相当于数据库的varchar类型，该类型是一个可变的字符串，不过它不能声明其中最多能存储多少个字符，理论上它可以存储2GB的字符数。

### 集合数据类型

//TODO

## DDL数据定义

### 创建数据库

创建一个数据库，数据库在HDFS上的默认存储路径是/user/hive/warehouse/*.db

```sql
hive(default)> create database school;
-- 标准写法
hive(default)> create database if not exists school;
-- 显示数据库
hive(default)> show databases;
-- 过滤选择数据库
hive(default)> show databases like 'sch*';
-- 查看数据库的详细信息
hive(default)> desc database school;
hive(default)> desc database extended school;
-- 切换数据库
hive(default)> use school;
```

### 修改数据库

用户可以通过以下命令为数据库增加属性值，其他元数据信息是不可修改。

```sql
hive(default)> alter database school set dbproperties('owner'='fanl');
OK
Time taken: 0.08 seconds
hive(default)> desc database extended school;
OK
db_name	comment	location	owner_name	owner_type	parameters
school		hdfs://centos7:8020/user/hive/warehouse/school.db	fanl	USER	{owner=fanling}
Time taken: 0.018 seconds, Fetched: 1 row(s)
```

### 删除数据库

删除一个空数据库，非空数据库

```sql
hive(default)> drop database fanling;
-- 删除前判断是否存在
hive(default)> drop database if exists fanling;
-- 非空强制删除
hive(default)> drop database fanling cascade;
```

### 创建表

`Hive`默认情况下会将这些表的数据存储在由配置项`hive.metastore.warehouse.dir`所定义的目录的子目录下。 

当我们删除一个管理表时，`Hive`也会删除这个表中数据。管理表不适合和其他工具共享数据。

```sql
-- 普通创建表
hive(default)> create table if not exists school(sid int,sname string) row format delimited fields terminated by '\t';
-- 普通创建表指定数据位置
hive(default)> create table if not exists school(sid int,sname string) row format delimited fields terminated by '\t' location '/user/hive/warehouse/default.db/school';
-- as 复制表的结构和数据
hive(default)> create table if not exists student2 as select sid,sname from student;
-- like 复制表的结构
hive(default)> create table if not exists student3 like student;
```

#### 管理表和外部表

默认创建的表都是所谓的**管理表**，有时也被称为*内部表*。因为这种表，`Hive`会（或多或少地）控制着数据的生命周期。

删除该表并不会删除掉这份数据，不过描述表的元数据信息会被删除掉。

> （1）创建外部表

```sql
hive (default)> create external table if not exists emp(empno int,ename string,job string,mgr int,hiredate string,sal double,comm double,deptno int) 
              > row format delimited fields terminated by '\t';
OK
Time taken: 3.542 seconds
```

> （2）查看表的类型

```sql
hive (default)> desc formatted emp;
Table Type:         	EXTERNAL_TABLE
```

> （3）管理表和外部表的转化

注意：('EXTERNAL'='TRUE')和('EXTERNAL'='FALSE')为固定写法，区分大小写

```bash
hive (default)> alter table emp set tblproperties('EXTERNAL'='TRUE');
hive (default)> alter table emp set tblproperties('EXTERNAL'='FALSE');
```

#### 分区表

简单的说，Hive中的分区就是分目录，**支持多级分区。**

> （1）创建分区表，在原来的基础上添加分区字段

```sql
hive (default)> create table student_part(id int,name string) partitioned by(month string) row format delimited fields terminated by '\t';
```

> （2）导入数据到各个分区

```sql
hive (default)> load data local inpath '/home/fanl/student.txt' into table student_part partition(month='201901');
hive (default)> load data local inpath '/home/fanl/student.txt' into table student_part partition(month='201902');
```

> （3）查看HDFS上文件的变，多出了2个按照分区规则的目录

```sql
hive (default)> dfs -ls /user/hive/warehouse/student_partition;
Found 2 items
drwxrwxr-x   - fanl supergroup          0 2019-04-09 17:51 /user/hive/warehouse/student_partition/month=201901
drwxrwxr-x   - fanl supergroup          0 2019-04-09 17:51 /user/hive/warehouse/student_partition/month=201902
```

（4）查看分区的数据

```sql
hive (default)> select * from student_partition where month='201901';
-- 可以使用union all 联合查询多个分区数据
```

（5）为已经存在的表新增分区和将分区删除掉

```sql
hive (default)> alter table student add(drop) partition(month='201901')[,partition(month='201902')];
-- 查看表有多少分区
hive (default)> show partitions student_partition;
OK
partition
month=201901
month=201902
Time taken: 0.113 seconds, Fetched: 2 row(s)
```

#### 修改表

（1）重命名表

```sql
hive (default)> alter table student rename to student_new;
```

（2）增加/删除/替换表的列

```sql
-- 添加列
hive (default)> alter table student_new add columns(sex string);
-- 更新列
hive (default)> alter table student_new change column sex sex2 int;
-- 替换表中所有字段
hive (default)> alter table student_new replace columns(sid int, sname string);
```

#### 删除表

```sql
hive (default)> drop table stundent_new;
-- Truncate只能删除管理表，不能删除外部表中数据
hive(default)> truncate table student;
```

## DML数据操作

### 数据导入

基础语法：

​	load data [local] inpath '路径' [overwrite] into table student [partition(partcol1=val1,…)];

（1）本地文件系统加载

```sql
hive (default)> load data local inpath '/home/fanl/student.txt' into table student;
```

（2）HDFS上加载数据

```sql
hive (default)> load data inpath '/user/fanl/student.txt' into table student;
```

（3）覆盖已经存在的数据

```sql
hive (default)> load data [local] inpath 'path' overwrite into table student;
```

（4）import从HDFS上导入数据，格式要和导出的一致

```sql
hive (default)> import table student from '/home/fanl/student';
```

也可以通过查询语句导入数据，不再说明。

使用`as select`，查看本章节的【创建表】

### 数据导出

（1）使用insert，配置到目录即可。

```sql
-- 将查询的数据导出到本地
hive (default)> insert overwrite local directory '/home/fanling/student' select * from student
-- 将查询的数据导出到HDFS
hive (default)> insert overwrite directory '/user/fanl/student' select * from student;
-- 格式化导出
hive (default)> insert overwrite [local] directory '/home/fanling/student' row format delimited fields terminated by '\t' select * from student
```

（2）Hadoop直接下载文件

```bash
hive (default)> dfs -get 'user/hive/warehouse/xxx' 'xxxx'
```

（3）Hive Shell导出

```sql
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hive -e 'select * from student' > /home/fanl/student/t.txt
```

（4）export导出到HDFS上的目录

```sql
hive (default)> export table student to '/home/fanl/student';
```

**注意**：将在**Sqoop教程**中再次说明导入/导出数据。

### Hive的查询

基本查询的语法和MySQL是一样的，如select、update、limit、between and，仅说明不同的地方。

> （1）like 和rlike

like中 `*`表示多个任意字符，`_`表示一个任意字符

rlike是Hive的扩展，使用Java的正则表达式匹配。

```sql
-- 查看emp表中薪水含有2的数据
hive (default)> select * from emp where sal rlike '[2]';
```

> （2）排序

**全局排序 order By**

全局排序 1个reducer

```sql
-- asc 升序 默认
-- desc 降序
```

每个Reducer内部排序Sort By

```sql
-- 设置Reducer的个数
hive (default)> set mapreduce.job.reduces=3;
hive (default)> select * from emp sort by sal;
hive (default)> insert overwrite local directory '/home/fanl/emp' select * from emp sort by sal;
```

**分区排序 distribute by**

结合sort by使用，**注意**：Hive要求distribute by语句要写在sort by语句之前。

**cluster by** 

当distribute by和sort by字段相同时，可以使用cluster by方式，但是排序只能是升序排序。

### 分桶和抽样查询

TODO

## Hive的函数

### 查看内置函数

```bash
hive> show functions;
# 查看函数的描述
hive> desc function upper;
# 查看函数的详细描述、用法
hive> desc function extended upper;
```

### 自定义函数UDF

根据用户自定义函数类别分为以下三种：UDF（一进一出）、UDAF（多进一出）、UDTF（一进多出）

