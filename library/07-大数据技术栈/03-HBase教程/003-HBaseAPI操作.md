# HBaseAPI操作

## Java API操作

（1）引入依赖

```xml
	<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <hadoop_version>2.6.0</hadoop_version>
        <hbase_version>1.2.0</hbase_version>
        <hive_version>1.1.0</hive_version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>${hadoop_version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-server</artifactId>
            <version>${hbase_version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>${hbase_version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>${hive_version}</version>
        </dependency>
    </dependencies>
```

## HBase集成

### 集成MapReduce

#### 官方程序

使用HBase自带mr-jar功能包 ，`lib/hbase-server-1.2.0-cdh5.14.2.jar`，实现向HBase中批量的导入或统计数据。

（1）编写脚本设置环境变量

```bash
#! /bin/bash
# 仅仅在当前窗口有效，设置mr的环境信息
export HBASE_HOME=/opt/modules/cdh5.14.2/hadoop-2.6.0-cdh5.14.2
export HADOOP_HOME=/opt/modules/cdh5.14.2/hadoop-2.6.0-cdh5.14.2
export HADOOP_CLASSPATH=`${HBASE_HOME}/bin/hbase  mapredcp`
```

（2）统计hbase表的行-rowcounter

```bash
[fanl@centos7 hadoop-2.6.0-cdh5.14.2]$ bin/hadoop jar /opt/modules/cdh5.14.2/hbase-1.2.0-cdh5.14.2/lib/hbase-server-1.2.0-cdh5.14.2.jar rowcounter ns1:t1
```

（3）上传tsv格式的文件（该文件中的每列数据直接是以制表符分割 ）

```bash
# 创建表 student info
hbase(main):001:0> create 'student','info'
# 新建tsv文件
# 将tsv文件上传到HDFS
[fanl@centos7 hadoop-2.6.0-cdh5.14.2]$ bin/hdfs dfs -put /home/fanl/data/student.tsv /user/fanl
# 执行mr任务
[fanl@centos7 hadoop-2.6.0-cdh5.14.2]$ bin/hadoop jar /opt/modules/cdh5.14.2/hbase-1.2.0-cdh5.14.2/lib/hbase-server-1.2.0-cdh5.14.2.jar importtsv \
> -Dimporttsv.columns=HBASE_ROW_KEY,info:name,info:age \
> student \
> hdfs://centos7:8020/user/fanl/student.tsv
# 查看数据
hbase(main):002:0> scan 'student'
# 显示了数据
```

（4）使用bulk load方式导入，适合大批量的数据，是直接写入storedfile中

```bash
# 创建表 student1 info
hbase(main):001:0> create 'student1','info'
# 将tsv文件转化为storefile
[fanl@centos7 hadoop-2.6.0-cdh5.14.2]$ bin/hadoop jar /opt/modules/cdh5.14.2/hbase-1.2.0-cdh5.14.2/lib/hbase-server-1.2.0-cdh5.14.2.jar importtsv -Dimporttsv.columns=HBASE_ROW_KEY,info:name,info:age -Dimporttsv.bulk.output=/path/for/output student1 hdfs://centos7:8020/user/fanl/student.tsv
# 将转化好的文件移动到HBase中
[fanl@centos7 hadoop-2.6.0-cdh5.14.2]$ bin/hadoop jar /opt/modules/cdh5.14.2/hbase-1.2.0-cdh5.14.2/lib/hbase-server-1.2.0-cdh5.14.2.jar completebulkload  /path/for/output student1
# 查看数据
hbase(main):006:0> scan 'student1'
```

#### 自定义程序

### 集成Hive

利用Hive 的分析功能接口，使用hql语句分析HBase表中的数据。

（1）`hive-env.sh`中声明`HBASE_HOME`路径 

```bash
export HBASE_HOME=/opt/modules/cdh5.14.2/hbase-1.2.0-cdh5.14.2
```

（2）例子：在Hive端执行hql去映射并分析HBase上已经存在的一张表

```bash
# Hive需要创建一个外部表与之映射
# 可以只映射hbase表中的部分列
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hive --service metastore & 
hive (default)> create external table hbase_student(id int,name string,age string)
              > stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
              > with serdeproperties("hbase.columns.mapping" = ":key,info:name,info:age")
              > tblproperties ("hbase.table.name" = "student");
# ！！缺少jar的报错，需要执行临时环境变量的
hive (default)> select * from hbase_student;
OK
hbase_student.id	hbase_student.name	hbase_student.age
10001	tom	22
10002	jack	23
10003	lily	24
10004	lulu	25
10005	kuku	26
Time taken: 0.866 seconds, Fetched: 5 row(s)
hive (default)> 
#可以通过hive作为接口向hbase表中插入数据 
#1. 创建临时表
hive (default)> create table student_temp(id int,name string,age string)
              > row format delimited fields terminated by '\t';
#2. 导入数据
hive (default)> load data local inpath '/home/fanl/data/student_temp.txt' into table student_temp;
Loading data to table default.student_temp
Table default.student_temp stats: [numFiles=1, numRows=0, totalSize=36, rawDataSize=0]
OK
Time taken: 0.356 seconds
hive (default)> select * from student_temp;
OK
student_temp.id	student_temp.name	student_temp.age
10006	kk	21
10007	jj	23
10008	gg	24
Time taken: 0.048 seconds, Fetched: 3 row(s)
hive (default)> 
# 往Hive表的hbase_student插入数据
hive (default)> insert into table hbase_student select * from student_temp;
# 查看HBase数据库已经添加了数据
```

### 集成Sqoop

（1）修改配置文件sqoop-env.sh

```bash
export HBASE_HOME=/opt/modules/cdh5.14.2/hbase-1.2.0-cdh5.14.2
export ZOOCFGDIR=/opt/modules/cdh5.14.2/zookeeper-3.4.5-cdh5.14.2
```

（2）导入数据

```bash
mysql> source /home/fanl/data/so_detail.sql
```

（3）将MySQL数据导入到HBase中

```bash
[fanl@centos7 sqoop-1.4.6-cdh5.14.2]$ bin/sqoop  import  \
--connect jdbc:mysql://centos7:3306/company \
--username root \
--password 123456 \
--table so_detail  \
--columns "id,product_id,price" \
--hbase-table  "so_detail"  \
--hbase-create-table \
--column-family "info" \
--hbase-row-key "id"  \
--num-mappers 1
# 查看数据
hbase(main):002:0> scan 'so_detail',{ LIMIT => 10}
# 已经有了数据
```

### 集成Hue

（1）启动HBase 的thrift server服务进程

```bash
[fanl@centos7 hbase-1.2.0-cdh5.14.2]$ bin/hbase-daemon.sh start thrift
```

（2）修改hue.ini

```ini
hbase_clusters=(Cluster|centos7:9090)
hbase_conf_dir=/opt/modules/cdh5.14.2/hbase-1.2.0-cdh5.14.2/conf
```

（3）启动即可