# Linux基础知识

## 什么是Linux

 Linux是一个开源的操作系统。由内核、shell、文件系统和应用程序组成，Linux是指Linux内核，有很多发行版，常见有Ubantu、CentOS、RedHat等。

后续文章中默认将服务器称作为一个节点。

## 虚拟机安装Linux节点

### 最小化安装

下载CentOS7的官方网站最小化的安装源，启动最小化安装，过程不再说明。

### 虚拟机网络查看和配置

安装和配置网路模式为`NAT`，当前的电脑安装虚拟机的虚拟网卡网段为`192.168.157.0`，查看虚拟机的网络设置，查看NAT网卡的网段和网关。

### 节点名修改

```bash
# 查看主机名
[root@fanl01 ~]$ hostname
# 修改主机名
[root@fanl01 ~]$ hostnamectl set-hostname yourname
# 安装的过程也可以指定主机名
# 添加映射
[root@fanl01 ~]$ vi /etc/hosts
192.168.177.130 fanl01
192.168.177.151 hadoop1
192.168.177.152 hadoop2
192.168.177.153 hadoop3
```
### 节点网络配置

```bash
# 查看网络地址
[root@fanl01 ~]$ ip addr
# 添加静态ip
[root@fanl01 ~]$ vi /etc/sysconfig/network-scripts/ifcfg-ens33
# 添加
IPADDR=192.168.157.131
NETMASK=255.255.255.0
GATEWAY=192.168.157.2
DNS1=192.168.157.2
# 修改
ONBOOT=yes
BOOTPROTO=static
# 重启网络
[root@fanl01 ~]$ service network restart|stop|start
```

![1551271233048](assets\1551271233048.png)

### 节点防火墙配置

Centos升级到7之后，内置的防火墙已经从`iptables`变成了`firewalld`

```bash
# 非生产环境可以直接关闭防火墙
# 查看防火墙
[root@fanl01 ~]$ systemctl status firewalld
# 开机禁用，启用 enable
[root@fanl01 ~]$ systemctl disable firewalld
# 关闭防火墙
[root@fanl01 ~]$ systemctl stop firewalld
# 新增规则，这里新增redis端口访问
[root@fanl01 ~]$ firewall-cmd --zone=public --add-port=6379/tcp --permanent
```

### 关闭节点的Slinux

```bash
# 查看状态
[root@fanl01 ~]$ sestatus
# 禁用
[root@fanl01 ~]$ vi /etc/sysconfig/selinux
SELINUX=disabled
# 临时关闭
[root@fanl01 ~]$ setenforce 0
```

## Linux管理

### 节点SSH免密登录

通过RSA加密算生成了密钥，包括私钥和公钥，我们把公钥追加到用来认证授权的key中去。

每台机器配置本地免密登录，然后将其余每台机器生成的~/.ssh/id_rsa.pub先统一以hostname用scp命令发给Master主机，由Master追加到authorized_keys中，然后再将此文件发给其他Slave主机。

> 简单操作步骤

目前有3台服务器，**hadoop1**为主服务器，使用**非root账号**的操作步骤

确保以下配置文件已经修改 `/etc/ssh/sshd_conf`

```bash
AuthorizedKeysFile      .ssh/authorized_keys
PubkeyAuthentication yes
```

第一步：

进入**hadoop1**，使用命令`ssh-keygen`生成rsa密钥。

第二步：

进入**hadoop2**，**hadoop3** 生成密钥后传输到**hadoop1**，使用命令：

```bash
# hadoop2
$ scp .ssh/id_rsa.pub fanl@hadoop1:/home/fanl/.ssh/hadoop2key
# hadoop3
$ scp .ssh/id_rsa.pub fanl@hadoop1:/home/fanl/.ssh/hadoop3key
```

第三步：在**hadoop1**中将授权信息追加到`authorized_keys`

```bash
# 组合到authorized_keys中
[fanl@hadoop1 .ssh]$ cd /home/fanl/.ssh
[fanl@hadoop1 .ssh]$ touch authorized_keys
[fanl@hadoop1 .ssh]$ cat id_rsa.pub >> authorized_keys # 添加自身
[fanl@hadoop1 .ssh]$ cat hadoop2key >> authorized_keys
[fanl@hadoop1 .ssh]$ cat hadoop3key >> authorized_keys
```

第四步：在hadoop1中执行，将授权文件传输到其他服务器

```bash
[fanl@hadoop1 .ssh]$ scp authorized_keys fanl@hadoop2:/home/fanl/.ssh/
[fanl@hadoop1 .ssh]$ scp authorized_keys fanl@hadoop3:/home/fanl/.ssh/
```

如果是root账号，完成上面的步骤即可完成免密登录

```bash
[fanl@hadoop1 ~]$ chmod 700  ~/.ssh
[fanl@hadoop1 ~]$ chmod 600  ~/.ssh/authorized_keys
```

使用其他命令

```bash
ssh-copy-id hostname
```

连接服务器的命令是：`ssh hadoop2`，退出ssh的命令： `logout`

### Linux管理命令

#### 用户管理

```bash
# 新增用户
[root@fanl01 ~]$ adduser fanl
# 新建用户组
[root@fanl01 ~]$ groupadd hadoop
# 新建用户同时增加工作组 
[root@fanl01 ~]$ useradd -g hadoop fanl
# 给已有的用户增加工作组 
[root@fanl01 ~]$ usermod -G hadoop fanl	
# 修改密码
[root@fanl01 ~]$ passwd fanl
# 删除用户
[root@fanl01 ~]$ userdel fanl 
[root@fanl01 ~]$ groupdel hadoop
[root@fanl01 ~]$ usermod –G hadoop fanl
# 显示用户信息
[root@fanl01 ~]$ id fanl
[root@fanl01 ~]$ cat /etc/passwd
# 查看
[root@fanl01 ~]$ who
# 查看登录记录
[root@fanl01 ~]$ last
# 查看某一个用户
[root@fanl01 ~]$ W fanl
```

#### 普通用户赋予root权限

```bash
# 先查看文件权限
[root@fanl01 ~]$ ls -all /etc/sudoers
# 赋予写权限
[root@fanl01 ~]$ chmode 777 /etc/sudoers
[root@fanl01 ~]$ vi /etc/sudoers
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
# 添加一行
fanl    ALL=(ALL)       ALL
fanl    ALL=(ALL)    NOPASSWD: ALL  
# 将权限改为 440
[root@fanl01 ~]$ chmod 440 /etc/sudoers
###################################################################
# 新建文件夹测试
[fanl@fanl01 opt]$ mkdir soft
mkdir: 无法创建目录"soft": 权限不够
[fanl@fanl01 opt]$ sudo mkdir soft

我们信任您已经从系统管理员那里了解了日常注意事项。
总结起来无外乎这三点：

    #1) 尊重别人的隐私。
    #2) 输入前要先考虑(后果和风险)。
    #3) 权力越大，责任越大。

[sudo] fanl 的密码：
[fanl@fanl01 opt]$ ll
总用量 0
drwxr-xr-x. 2 root root 6 3月   5 19:32 soft
# 前面加sudo
```

#### 文件拥有者

```bash
# 修改文件拥有者
[fanl@fanl01 opt]$ chown fanl:fanl /software /module
```

### Linux文件命令

#### 切换目录

```bash
# 切换目录
[root@fanl01 ~]$ cd ..        # 切换到上层目录
[root@fanl01 ~]$ cd /         # 切换到系统跟目录
[root@fanl01 ~]$ cd ~         # 切换到用户主目录
[root@fanl01 ~]$ cd xx        # 切换到xx目录
[root@fanl01 ~]$ cd /xx/yy/zz #  切换到xx/yy/zz目录
```

#### 创建删除目录

```bash
# 创建和删除文件夹
[root@fanl01 ~]$ mkdir xx     # 创建xx目录
[root@fanl01 ~]$ rmdir xx     # 删除空目录xx
```

#### 创建和查看文件

```bash
# 浏览文件命令
[root@fanl01 ~]$ cat xx       # 查看xx文件
[root@fanl01 ~]$ cat >xx       # 新建xx文件
[root@fanl01 ~]$ cat xx1>>yy       # 将xx1追加到yy中

# 创建文件
[root@fanl01 ~]$ touch fanl.txt

# 文件操作命令
[root@fanl01 ~]$ rm -rf xx    # 删除xx文件或者目录
```

#### 文件复制和移动

```bash
# 复制命令
[root@fanl01 ~]$ cp a.txt b.txt # 将a.txt复制为b.txt
[root@fanl01 ~]$ scp 文件 用户名@IP/hostname:路径 # 远程复制

# 移动或重命名
[root@fanl01 ~]$ mv a.txt ../xx # 将a.txt 移动到上层xx目录中
[root@fanl01 ~]$ mv a.txt b.txt # 将a.txt 重命名为b.txt
```

#### 文件的解压和压缩

```shell
# 常见的压缩和解压，其他的自查
# tar
[root@fanl01 ~]tar -cvf filename.tar dir # 将目录dir中压缩到filename.tar中，保留原文件
[root@fanl01 ~]tar xvf filename.tar  # 将filename.tar解压到当前文件夹，保留原文件
# gz
[root@fanl01 ~]gzip filename # 不保留原文件
[root@fanl01 ~]gzip -c filename > filename.gz # 保留原文件
[root@fanl01 ~]gunzip filename.gz # 不保留原文件
[root@fanl01 ~]gunzip -c filename.gz > filename # 保留原文件 
# 常用的tar.gz / tgz
[root@fanl01 ~]tar zcvf filename.tgz  dir   # 将dir目录压缩到filename.tgz，dir也可以是文件名
[root@fanl01 ~]tar -zxvf filename.tar.gz # 解压到当前目录，保留原文件
[root@fanl01 ~]tar -zxvf filename.tar.gz -C dir # 解压到dir目录，保留原文件
```

#### 目录和文件的权限修改

```bash
# 权限修改
# -rwxrw-r--
# 第1位    文件类型 d 目录，-普通文件，|链接文件
# 第2-4位  所属用户权限 u
# 第5-7位  所属组权限 g
# 第8-10位 其他用户权限 o
# r：读取权限，数字代号为“4”
# w：写入权限，数字代号为“2”
# x：执行权限，数字代号为“1”
# -：没有权限，数字代号为“0”
# chmod ［who］ ［+ | - | =］ ［mode］ 文件名 
# 数字设定法我们必须首先了解用数字表示的属性的含义：0表示没有权限，1表示可执行权限，2表示可写权限，4表示可读权限，然后将其相加。所以数字属性的格式应为3个从0到7的八进制数，其顺序是（u）（g）（o）。 
[root@fanl01 ~] chomd 777 test.sh
```

[linux速查手册](https://jaywcjlove.gitee.io/linux-command)

### Linux软件安装

**安装wget**

```bash
[root@fanl01 ~]$ yum install wget
```
**替换yum源**

```bash
[root@fanl01 ~]$ cd /etc/yum.repos.d/
[root@fanl01 ~]$ mv CentOS-Base.repo CentOS-Base.repo.bak

[root@fanl01 ~]$ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
**安装gcc**

```bash
[root@fanl01 ~]$ yum install gcc
# 检查是否安装
[root@fanl01 ~]$ gcc -v
```

#### 离线安装JDK

**第一步**：下载压缩包，并且上传到linux中的/opt/software

**第二步**：解压

```bash
[fanl@fanl01 software]$ tar -zxvf jdk-8u201-linux-x64.tar_2.gz -C /opt/module/
# 记住当前路径
[fanl@fanl01 jdk1.8.0_201]$ pwd
/opt/module/jdk1.8.0_201
```

**第三步**：设置环境变量

```bash
[fanl@fanl01 jdk1.8.0_201]$ sudo vi /etc/profile
# shift+g到最后一行
# 添加以下内容
export JAVA_HOME=/opt/module/jdk1.8.0_201
export PATH=$PATH:$JAVA_HOME/bin
##############################################
# 报存后使其生效
[fanl@fanl01 jdk1.8.0_201]$ source /etc/profile
# 检测JAVA环境
[fanl@fanl01 jdk1.8.0_201]$ java -version
```

### 安装远程同步工具rsync

```bash
[fanl@hadoop2 module]$ sudo yum -y install rsync
# 安装完毕后配置文件
[fanl@hadoop2 module]$ sudo vi /etc/rsyncd.conf
uid = nobody
gid = nobody
use chroot = no
max connections = 200
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsync.lock
log file = /var/log/rsyncd.log
# 常见错误-权限不足，查看对应的目录是否用用户的权限或者所属
```

### Linux 常用的快捷键

- Ctrl+L : 清屏
- Shift +G：查看文件末尾
- dd：vi 中删除文字
- Tab：命令补齐

## Linux 文件目录

在Linux系统中，一切都是文件，理解文件系统，对于学习Linux来说，是一个非常有必要的前提。

Linux上的文件系统一般来说就是EXT2或EXT3。

标准的Linux文件系统Ext2是使用「基于inode的文件系统」。

###  Linux的文件类型

1. 普通文件
2. 目录文件
3. 符号链接：这种类型的文件类似Windows中的快捷方式
4. 块设备文件和字符设备文件，这些文件一般隐藏在/dev目录下
5. FIFO，管道文件主要用于进程间通讯
6. 套接字，这些文件一般隐藏在/var/run目录下

Linux 的文件是没有所谓的扩展名的，一个 Linux文件能不能被执行与它是否可执行的属性有关，只要你的权限中有 x ，比如[ -rwx-r-xr-x ] 就代表这个文件可以被执行。但是能不能执行成功，当然就得要看该文件的内容了。所以Linux 系统上的文件名真的只是让你了解该文件可能的用途而已。

### Linux的目录树

对Linux系统和用户来说，所有可操作的计算机资源都存在于目录树这个逻辑结构中，对计算机资源的访问都可以认为是目录树的访问。

目录树的逻辑结构也非常简单，就是从根目录（/）开始，不断向下展开各级子目录。

Linux把对不同文件系统的访问交给了VFS（虚拟文件系统）

具体可参考：

[Linux磁盘与文件系统管理](https://blog.csdn.net/shuiyihang0981/article/details/87922207)

### Linux的根

- / - 根目录：每一个文件和目录都从这里开始
- /bin - 用户二进制文件，系统的所有用户使用的命令都设在这里，例如：ps，ls，ping，grep，cp等
- /sbin - 系统二进制文件
- /etc - 配置文件
- /dev - 设备文件
- /proc - 进程信息
- /var - 变量文件，系统日志文件（/var/log）;包和数据库文件（/var/lib）
- /tmp - 临时文件
- /usr - 用户程序，包含二进制文件、库文件、文档和二级程序的源代码。
- /home - 所有用户用home目录来存储他们的个人档案
- /boot - 引导加载程序文件
- /lib - 系统库
- /opt - 可选的附加应用程序，附加应用程序应该安装在/opt/或者/opt/的子目录下。
- /mnt - 挂载目录
- /media - 可移动媒体设备

