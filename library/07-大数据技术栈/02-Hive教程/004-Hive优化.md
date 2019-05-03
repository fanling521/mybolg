# Hive 优化

## Fetch抓取

Fetch抓取是指，Hive中对某些情况的查询可以不必使用MapReduce计算。

配置文件中配置`hive.fetch.task.conversion`默认是more，该属性修改为more以后，在全局查找、字段查找、limit查找等都不走mapreduce。

## 本地模式

Hive可以通过本地模式在单台机器上处理所有的任务。对于小数据集，执行时间可以明显被缩短。

可以通过设置`hive.exec.mode.local.auto`的值为true，来让Hive在适当的时候自动启动这个优化。

```bash
hive (default)> select * from emp;
Time taken: 0.495 seconds, Fetched: 14 row(s)
hive (default)>  set hive.exec.mode.local.auto=true; 
hive (default)> select * from emp;
Time taken: 0.092 seconds, Fetched: 14 row(s)
hive (default)> 
```

## 表的优化

### Map Join

在Reduce阶段完成Join，容易发生数据倾斜。可以用Map Join把小表全部加载到内存在map端进行Join，避免Reduce端处理。

### 动态分区

（1）开启动态分区

```
hive.exec.dynamic.partition=true
```

（2）设置为非严格模式

```
hive.exec.dynamic.partition.mode=nonstrict
# 在所有执行MR的节点上，最大一共可以创建多少个动态分区
hive.exec.max.dynamic.partitions=1000
hive.exec.max.dynamic.partitions.pernode=100
# 整个MR Job中，最大可以创建多少个HDFS文件
hive.exec.max.created.files=100000
# 当有空分区生成时，是否抛出异常。一般不需要设置
hive.error.on.empty.partition=false
```

### 分区分桶

不再说明。

## 数据倾斜

（1）合理设置Map/Reduce数，如`set mapreduce.job.reduces = 15`

（2）小文件合并

（3）复杂文件增加map数，根据`Math.max(minSize,Math.min(maxSize,blocksize))`，让maxSize最大值低于blocksize就可以增加map的个数

## 其他优化点

### 并行执行

通过设置参数`hive.exec.parallel`值为true，就可以开启并发执行。

不过，在共享集群中，需要注意下，如果job中并行阶段增多，那么集群利用率就会增加。

### 严格模式

通过设置属性`hive.mapred.mode`值为默认是非严格模式`nonstrict`

### JVM重用

JVM重用是Hadoop调优参数的内容，其对Hive的性能具有非常大的影响，特别是对于很难避免小文件的场景或task特别多的场景，这类场景大多数执行时间都很短。

### 推测执行

Hadoop采用了推测执行（Speculative Execution）机制，它根据一定的法则推测出“拖后腿”的任务，并为这样的任务启动一个备份任务，让该任务与原始任务同时处理同一份数据，并最终选用最先成功运行完成任务的计算结果作为最终结果。

mapred-site.xml

```xml
<property>
	<name>mapreduce.map.speculative</name>
	<value>true</value>
</property>

<property>
	<name>mapreduce.reduce.speculative</name>
	<value>true</value>
</property>
```

hive本身的配置

```xml
<property>
	<name>hive.mapred.reduce.tasks.speculative.execution</name>
	<value>true</value>
</property>
```

### 压缩

不再说明

### 执行计划

```bash
0: jdbc:hive2://centos7:10000> explain select * from emp;
```

## 相关问题

（1）Hive如何解决数据倾斜？

（2）Hive导致数据倾斜的可能性，从小文件，空值key太多

（3）Hive执行过程中小文件如何处理

（4）Hive中元数据保存在哪里？

（5）Hive中复杂数据类型有哪些？

（6）Hive中表的类型有哪些？

（7）udfs如何自定义，怎么去注册？

（8）Hive的数据导入导出，如何加载？保存在哪里？