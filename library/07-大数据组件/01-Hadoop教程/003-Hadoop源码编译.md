# Hadoop源码编译

## 前期准备

### Linux操作系统

配置CentOS能连接外网，使用root超级管理员编译开发。

### 工具安装

```bash
# =====安装JDK========
# =====安装Maven========
# =====安装ant========
# =====glibc-headers========
[root@centos01 ~]$ yum install glibc-headers
# =====make 和 cmake========
[root@centos01 ~]$ yum install make
[root@centos01 ~]$ yum install cmake
# =====安装openssl库========
[root@centos01 ~]$ yum install openssl-devel
# =====安装ncurses-devel库========
[root@centos01 ~]$ yum install ncurses-devel
```

## 编译源码

### Maven 编译

```bash
[root@centos01 ~]$ mvn package -Pdist,native -DskipTests -Dtar
```

等待漫长时间出现错误，再次执行命令。

编译成功的位置`hadoop-dist/target`





