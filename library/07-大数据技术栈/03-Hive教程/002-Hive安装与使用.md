# Hive 安装与使用

## Hive 伪分布安装

（1）上传解压

（2）修改`conf/hive-env.sh`

```bash
HADOOP_HOME=/opt/modules/cdh5.14.2/hadoop-cdh5.14.12
export HIVE_CONF_DIR=/opt/modules/cdh5.14.2/hive-1.1.0-cdh5.14.2/conf
```

（2）自定义日志位置`hive-log4j.properties`

```bash
# 需要新建logs文件夹
hive.log.dir=/opt/modules/cdh5.14.2/hive-1.1.0-cdh5.14.2/logs
```

（3）创建仓库和授权

```bash
[fanl@centos7 hadoop-cdh5.14.12]$ bin/hdfs dfs -mkdir -p /user/hive/warehouse
[fanl@centos7 hadoop-cdh5.14.12]$ bin/hdfs dfs -chmod g+w /user/hive/warehouse /tmp
```

（4）使用`Hive`

```bash
# =================首先需要启动hadoop相关进程=================
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hive
```

## Hive简单操作

>  案例：将本地文件student.txt导入Hive

```bash
# 显示所有数据库
hive>show databases;
# 使用数据库
hive>use default;
# 显示所有表
hive>show tables;
# 新建数据库
hive>create database fanling;
# 新建表，需要指定文件的分隔符（需要先有文件再建立表的结构）
hive>create table student(id int,name string) row format delimited fields terminated by '\t';
# 导入数据
hive>load data local inpath '/home/fanl/student.txt' into table student;
```

## Hive元数据配置到MySql

Metastore默认存储在自带的derby数据库中，不支持多人操作。

### 安装Mysql5.7

CentOS7最小化安装需要重新安装Mysql源，5.7版本，本段编写时间*2019年4月4日*

```bash
[fanl@centos7 ~]$ wget -P /usr/mysql/ http://repo.mysql.com/mysql57-community-release-el7.rpm
[fanl@centos7 ~]$ rpm -ivh /usr/mysql/mysql57-community-release-el7.rpm
[fanl@centos7 ~]$ yum -y install mysql-server
```

**注意：**Mysql5.7默认安装之后root是有密码的。

```bash
grep 'temporary password' /var/log/mysqld.log
```
（1）配置启动Mysql

```bash
[root@centos7 ~]$ service mysqld start
# 或者
[root@centos7 ~]$ systemctl mysqld start
# 开机启动
[root@centos7 ~]$ chkconfig mysqld on
# 修改密码
[root@centos7 ~]$ mysql -uroot -p密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';

```

（2）解决：ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

```bash
mysql> set global validate_password_policy=0; 
mysql> set global validate_password_length=1;
```

（3）授权其他机器

```bash
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
mysql> FLUSH  PRIVILEGES;
```

### 配置元数据库存储

（1）新增hive-site.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
  <property>
	<name>javax.jdo.option.ConnectionURL</name>
	<value>jdbc:mysql://127.0.0.1:3306/metastore?createDatabaseIfNotExist=true</value>
  </property>
	<property>
	<name>javax.jdo.option.ConnectionDriverName</name>
	<value>com.mysql.jdbc.Driver</value>
  </property>
	<property>
	<name>javax.jdo.option.ConnectionUserName</name>
	<value>root</value>
  </property>
	<property>
	<name>javax.jdo.option.ConnectionPassword</name>
	<value>123456</value>
  </property>
</configuration>
```

（2）拷贝驱动包到hive安装包下的lib目录下