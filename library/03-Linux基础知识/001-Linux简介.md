# Linux简介

## 什么是Linux

 Linux是一个开源的操作系统。由内核、shell、文件系统和应用程序组成，Linux是指linux内核。

## 虚拟机安装Linux

> 最小化安装CentOS 7 

1. 下载官方网站最小化的安装源，启动最小化安装，第一台linux 命名为"fanl01"，密码为 fanling

2. 安装和配置网路模式为NAT，目前笔记本电脑的虚拟机网卡网段为`192.168.177.0`

   | IP地址 IPADDR   | 主机名hostname |
   | --------------- | -------------- |
   | 192.168.177.130 | fanl01         |
   | 192.168.177.131 | fanl02         |
   | 192.168.177.132 | fanl03         |
   | 192.168.177.133 | fanl04         |

> 克隆，命名主机，保存快照，设置网络

1. 查看主机名：执行命令 `hostname`

2. 重命名主机名：(1)命令：`hostnamectl set-hostname fanl01`后重启；（2）修改*/etc/sysconfig/network*  配置文件，修改 HOSTNAME=fanl01；（3）修改本机的域名解析文件 */etc/hosts*，将localhost.xx改为localhost.fanl01，并且添加集群内计算机ip和主机名的映射。

3. 配置网络：`/etc/sysconfig/network-scripts/ifcfg-ens33` 将虚拟机nat地址修改到虚拟机上。添加`IPADDR,NETMASK,GATWAY`以上的数据来源于虚拟的虚拟网卡vmnet8，将dhcp改为static,并且关闭**防火墙 iptables**（查看网络ip的命令为`ip addr`）

   完成集群linux 虚拟机器部署。

   ![1551271233048](assets\1551271233048.png)

## Linux管理

### Linux ssh 免密登录

通过RSA加密算生成了密钥，包括私钥和公钥，我们把公钥追加到用来认证授权的key中去。

每台机器配置本地免密登录，然后将其余每台机器生成的~/.ssh/id_rsa.pub先统一以hostname用scp命令发给Master主机，由Master追加到authorized_keys中，然后再将此文件发给其他Slave主机。

目前有4台虚拟机，fanl01为Master，fanl02为Slave01，fanl02为Slave02，fanl03为Slave03

1. 进入fanl01，使用命令`ssh-keygen`生成rsa密钥。
2. 查看生成的密钥，` cd ~/.ssh/`,公钥为`id_rsa.pub`，私钥为`id_rsa`
3. 进入fanl02-fanl04 生成密钥后传输到fanl01，`scp id_rsa.pub root@fanl01:~/.ssh/fanl02key`。
4. 方案2：使用命令`ssh_copy_id -i ~/.ssh/id_rsa.pub root@fanl01`
5. 将文件追加`cat fanl02key >> authorized_keys`，重复操作，最终fanl01上的授权文件包括了所有服务器，然后将其转发给其他服务器，`scp authorized_keys root@fanl02:~/.ssh`
6. 退出ssh，输入命令： `logout`
7. 新增服务器，需要重复上面的操作，并且修改`/etc/hosts`

### Linux基本操作命令

```shell
#切换目录
cd ..        #切换到上层目录
cd /         #切换到系统跟目录
cd ~         #切换到用户主目录
cd xx        #切换到xx目录
cd /xx/yy/zz #切换到xx/yy/zz目录

#创建和删除文件夹
mkdir xx     #创建xx目录
rmdir xx     #删除空目录xx

#浏览文件命令
cat xx       #查看xx文件

#文件操作命令
rm -rf xx    #删除xx文件或者目录

#复制命令
cp a.txt b.txt #将a.txt复制为b.txt
scp 文件 用户名@IP/hostname:路径 #远程复制

#移动或重命名
mv a.txt ../xx #将a.txt 移动到上层xx目录中
mv a.txt b.txt #将a.txt 重命名为b.txt

#压缩和解压


```

[linux速查手册](https://jaywcjlove.gitee.io/linux-command)

## Linux 文件目录

在Linux系统中，一切都是文件，理解文件系统，对于学习Linux来说，是一个非常有必要的前提。

Linux上的文件系统一般来说就是EXT2或EXT3。

标准的Linux文件系统Ext2是使用「基于inode的文件系统」。

###  Linux中的文件类型

1. 普通文件
2. 目录文件
3. 符号链接：这种类型的文件类似Windows中的快捷方式
4. 块设备文件和字符设备文件，这些文件一般隐藏在/dev目录下
5. FIFO，管道文件主要用于进程间通讯
6. 套接字，这些文件一般隐藏在/var/run目录下

Linux 的文件是没有所谓的扩展名的，一个 Linux文件能不能被执行与它是否可执行的属性有关，只要你的权限中有 x ，比如[ -rwx-r-xr-x ] 就代表这个文件可以被执行。但是能不能执行成功，当然就得要看该文件的内容了。所以Linux 系统上的文件名真的只是让你了解该文件可能的用途而已。

### Linux目录树

对Linux系统和用户来说，所有可操作的计算机资源都存在于目录树这个逻辑结构中，对计算机资源的访问都可以认为是目录树的访问。

目录树的逻辑结构也非常简单，就是从根目录（/）开始，不断向下展开各级子目录。

Linux把对不同文件系统的访问交给了VFS（虚拟文件系统）

具体可参考：

[Linux磁盘与文件系统管理](https://blog.csdn.net/shuiyihang0981/article/details/87922207)

### Linux根

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

