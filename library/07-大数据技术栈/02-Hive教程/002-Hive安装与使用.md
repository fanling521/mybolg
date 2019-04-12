# Hive 安装与使用

## Hive的安装

（1）上传解压

（2）修改`conf/hive-env.sh`

```bash
# =================修改环境变量=================
HADOOP_HOME=/opt/modules/cdh5.14.2/hadoop-cdh5.14.12
export HIVE_CONF_DIR=/opt/modules/cdh5.14.2/hive-1.1.0-cdh5.14.2/conf
```

（2）自定义日志位置`hive-log4j.properties`

```bash
# =================需要新建logs文件夹=================
hive.log.dir=/opt/modules/cdh5.14.2/hive-1.1.0-cdh5.14.2/logs
```

（3）创建仓库和授权

```bash
# =================创建目录和授权=================
[fanl@centos7 hadoop-cdh5.14.12]$ bin/hdfs dfs -mkdir -p /user/hive/warehouse
[fanl@centos7 hadoop-cdh5.14.12]$ bin/hdfs dfs -chmod g+w /user/hive/warehouse /tmp
# 注意：
# Hive的配置参数   hive.metastore.warehouse.dir
# 可以配置数据地址，配置完成后和上面的操作一致
```

（4）使用`Hive`

```bash
# =================首先需要启动hadoop相关进程=================
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hive
```

## Hive的使用

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

`Metastore`默认存储在自带的`derby`数据库中，不支持多人操作。

### 安装Mysql5.7

**CentOS7**最小化安装需要重新安装Mysql源，5.7版本，本段编写时间*2019年4月4日*

```bash
[fanl@centos7 ~]$ wget -P /usr/mysql/ http://repo.mysql.com/mysql57-community-release-el7.rpm
[fanl@centos7 ~]$ rpm -ivh /usr/mysql/mysql57-community-release-el7.rpm
[fanl@centos7 ~]$ yum -y install mysql-server
```

**注意：**`Mysql5.7`默认安装之后root是有密码的。

```bash
[fanl@centos7 ~]$ grep 'temporary password' /var/log/mysqld.log
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
     <!-- 配置元数据库存储 -->
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
    <!-- 查询后信息显示配置 -->
    <property>
		<name>hive.cli.print.header</name>
		<value>true</value>	
	</property>
	<property>
		<name>hive.cli.print.current.db</name>
		<value>true</value>
	</property>
</configuration>
```

（2）拷贝驱动包到hive安装包下的lib目录下

完成以上步骤，完成元数据的配置，可以多窗口操作Hive。

### HiveJDBC访问

（1）启动hiveserver2

```bash
# 窗口不可用
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hiveserver2
# ----后台进程--------------
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hiveserver2 &
```

（2）启动beeline

```bash
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/beeline
beeline> !connect jdbc:hive2://centos7:10000
beeline> 
# 登录成功后即可进行操作
```

### Hive常用的交互命令

```bash
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hive -help # 用来查看hive的帮助信息
usage: hive
 -d,--define <key=value>          Variable subsitution to apply to hive
                                  commands. e.g. -d A=B or --define A=B
    --database <databasename>     Specify the database to use
 -e <quoted-query-string>         SQL from command line
 -f <filename>                    SQL from files
 -H,--help                        Print help information
    --hiveconf <property=value>   Use value for given property
    --hivevar <key=value>         Variable subsitution to apply to hive
                                  commands. e.g. --hivevar A=B
 -i <filename>                    Initialization SQL file
 -S,--silent                      Silent mode in interactive shell
 -v,--verbose                     Verbose mode (echo executed SQL to the
                                  console)
```

（1）`-e`不进入Hive使用HQL语句

```bash
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hive -e "show databases;use school;select * from student"
```

（2）`-f`执行脚本中HQL语句

```bash
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hive -f ~/hql
# 执行完毕后不进入客户端，
# 需要执行完毕后进入客户端，需要使用-i
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hive -i ~/hql
```

（3）`-S`静默模式，不打印日志

```bash
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hive -S -f ~/hql >> ~/hql.txt
```

其他的看上面的帮助。

### Hive其他操作

（1）临时设置配置信息,，优先级最高

其他方式还有hive-site.xml的配置以及启动参数配置，不再说明。

```bash
[fanl@centos7 hive-1.1.0-cdh5.14.2]$ bin/hive
hive (default)> set hive.cli.print.current.db=false;
hive> 
```

（2）操作HDFS - dfs xx

```bash
hive> dfs -ls /;
Found 2 items
drwxrwx---   - fanl supergroup          0 2019-04-08 11:28 /tmp
drwxr-xr-x   - fanl supergroup          0 2019-04-04 14:26 /user
hive> 
```

（3）操作Shell

```bash
hive> ! ls /opt;
modules
software
hive> 
```

## 面试题

（1）配置`hive-env.sh`都涉及到哪些属性？

（2）HiveServer2的作用是什么？如何连接HiveServer2？