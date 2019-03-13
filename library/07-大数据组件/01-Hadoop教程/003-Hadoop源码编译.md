# Hadoop源码编译

## 前期准备

### CentOS

配置CentOS能连接外网，使用root角色。

#### 工具

##### JDK

##### Maven

##### ant

##### glibc-headers

```bash
$ yum install glibc-headers
```

##### make 和 cmake

```bash
$ yum install make
$ yum install cmake
```

##### protobuf

##### 安装openssl库

```bash
$ yum install openssl-devel
```

##### 安装ncurses-devel库

```bash
$ yum install ncurses-devel
```

## 编译源码

#### 解压源码

### Maven 编译

```bash
$ mvn package -Pdist,native -DskipTests -Dtar
```

等待漫长时间出现错误，再次执行命令。

编译成功的位置`hadoop-dist/target`





