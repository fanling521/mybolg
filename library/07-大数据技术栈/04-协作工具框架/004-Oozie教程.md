# Oozie教程

## 什么是Oozie

由Cloudera公司贡献给Apache，基于工作流引擎的开源框架，提供对Hadoop MapReduce、Pig Jobs的任务调度与协调。

Oozie需要部署到Java Servlet容器中运行。主要用于定时调度任务，多任务可以按照执行的逻辑顺序调度。

### 物理组件

**1)** Oozieserver / Tomcat server 
Oozie的核心服务进程，提供了与其他框架或客户端的交互通道 
**2)** Oozie database 
Oozie 需要一个数据库的支撑存储到调度的任务元数据信息

### Oozie的模块

**1) **Workflow

顺序执行流程节点，支持fork（分支多个节点），join（合并多个节点为一个）

**2) **Coordinator

定时触发workflow

**3) **Bundle Job

绑定多个Coordinator

###  常用节点

**1) **控制流节点（Control Flow Nodes）

控制流节点一般都是定义在工作流开始或者结束的位置，比如start,end,kill等。以及提供工作流的执行路径机制，如decision，fork，join等。

**2) **动作节点（Action  Nodes）

负责执行具体动作的节点，比如：拷贝文件，执行某个Shell脚本等等。

## Oozie的安装部署

> （1）上传解压tar包

>（2）在Hadoop的core-site.xml中添加代理用户

```xml
	<!-- Oozie -->
	<property>
		<name>hadoop.proxyuser.fanl.hosts</name>
		<value>*</value>
	</property>
	<property>
		<name>hadoop.proxyuser.fanl.groups</name>
		<value>*</value>
	</property>	
```

> （3）解压依赖包

```bash
tar -zxvf oozie-hadooplibs-4.1.0-cdh5.14.2.tar.gz -C /opt/modules/cdh5.14.2/oozie-4.1.0-cdh5.14.2/

[fanl@centos7 oozie-4.1.0-cdh5.14.2]$ mv hadooplibs/ /opt/modules/cdh5.14.2/oozie-4.1.0-cdh5.14.2/
[fanl@centos7 oozie-4.1.0-cdh5.14.2]$ rm -rf oozie-4.1.0-cdh5.14.2/
[fanl@centos7 oozie-4.1.0-cdh5.14.2]$ mkdir libext
[fanl@centos7 oozie-4.1.0-cdh5.14.2]$ cp hadooplibs/hadooplib-2.6.0-cdh5.14.2.oozie-4.1.0-cdh5.14.2/* libext/
# 上传ext-2.2.zip到libext目录下
```

> （4）创建Oozie数据库，并且拷贝驱动包到libext目录下

```bash
mysql> create database oozie;
Query OK, 1 row affected (0.00 sec)

mysql> 
```

> （5）配置Oozie访问数据库，文件为 **conf/oozie-site.xml**

```xml
	<!-- MySQL -->
	<property>
        <name>oozie.service.JPAService.jdbc.driver</name>
        <value>com.mysql.jdbc.Driver</value>
    </property>

    <property>
        <name>oozie.service.JPAService.jdbc.url</name>
        <value>jdbc:mysql://centos7:3306/oozie</value>
    </property>

    <property>
        <name>oozie.service.JPAService.jdbc.username</name>
        <value>root</value>
    </property>

    <property>
        <name>oozie.service.JPAService.jdbc.password</name>
        <value>123456</value>
    </property>
```

> （6）配置与Hadoop的关联

```xml
	<!-- Hadoop -->
	<property>
        <name>oozie.service.HadoopAccessorService.hadoop.configurations</name>
        <value>*=/opt/modules/cdh5.14.2/hadoop-cdh5.14.12/etc/hadoop</value>
    </property>
```

> （7）初始化Oozie

```bash
# 第一步：上传Oozie目录下的yarn.tar.gz文件到HDFS
[fanl@centos7 oozie-4.1.0-cdh5.14.2]$ bin/oozie-setup.sh sharelib create -fs hdfs://centos7:8020 -locallib oozie-sharelib-4.1.0-cdh5.14.2-yarn.tar.gz
# the destination path for sharelib is: /user/fanl/share/lib/lib_20190416120829
# 第二步：初始化MySQL的表
[fanl@centos7 oozie-4.1.0-cdh5.14.2]$ bin/oozie-setup.sh db create -run -sqlfile ./oozie.sql
# 第三步：war包
[fanl@centos7 oozie-4.1.0-cdh5.14.2]$ bin/oozie-setup.sh prepare-war
# 第四步：启动
[fanl@centos7 oozie-4.1.0-cdh5.14.2]$ bin/oozied.sh start
# 地址：
http://centos7:11000/oozie/

```

## Oozie案例实践

### 执行单一Shell任务

（1）解压例子模板，将shell 复制到Oozie根目录下的myapp下

修改job.properties

```properties
nameNode=hdfs://centos7:8020
jobTracker=centos7:8032
queueName=default
examplesRoot=oozieApps
shell_name=check_memery.sh

oozie.wf.application.path=${nameNode}/${examplesRoot}/myshell
```

修改workflow.xml

```xml
<exec>${shell_name}</exec>
<file>/${examplesRoot}/myshell/${shell_name}#${shell_name}</file>
```

新建shell

```bash
#! /bin/bash
/usr/bin/free -m >> /tmp/mem.log
/bin/date +"%Y%m%d-%H:%M:%S" >> /tmp/mem.log
/bin/echo "~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~" >>  /tmp/mem.log
```

（2）上传所需要的文件，job.properties不是必需的文件

```
[fanl@centos7 hadoop-cdh5.14.12]$ bin/hdfs dfs -put /opt/modules/cdh5.14.2/oozie-4.1.0-cdh5.14.2/myapps/shell /oozieApp
```

（3）提交任务

```bash
[fanl@centos7 oozie-4.1.0-cdh5.14.2]$ bin/oozie job -oozie http://centos7:11000/oozie -config /opt/modules/cdh5.14.2/oozie-4.1.0-cdh5.14.2/myapps/shell/job.properties -run
job: 0000000-190416203452824-oozie-fanl-W
```

### 执行多个Shell任务

相比上一个案例，需要多个action节点

