# Linux简介

## 什么是Linux

 Linux是一个开源的操作系统。

## 虚拟机安装Linux

> 最小化安装CentOS 7 

1. 下载安装源，启动最小化安装，第一台linux 命名为"fanl01"，密码为 fanling，依次类推，克隆3个

2. 安装和配置网路模式为NAT，当前虚拟机网卡网段为`192.168.177.0`

   | 192.168.177.130 | fanl01 |
   | --------------- | ------ |
   | 192.168.177.131 | fanl02 |
   | 192.168.177.132 | fanl03 |
   | 192.168.177.133 | fanl04 |

> 克隆，命名主机，保存快照，设置网络

1. 查看主机名： `hostname`

2. 重命名主机名：(1)命令：`hostnamectl set-hostname fanl01`后重启；（2）修改*/etc/sysconfig/network*  配置文件，修改 HOSTNAME=fanl01；（3）修改本机的域名解析文件 */etc/hosts*，将localhost.xx改为localhost.fanl01

3. 配置网络：*/etc/sysconfig/network-scripts/ifcfg-eth0* 将虚拟机nat地址修改到虚拟机上。添加`IPADDR,NETMASK,GATWAY`以上的数据来源于虚拟的虚拟网卡v8，并且关闭**防火墙**（查看网络ip的命令为`ip addr`）

4. 添加主机名和ip映射关系，将集群内的ip和主机名一一映射在hosts中，如`192.168.177.130 fanl01`

   完成集群linux 虚拟机器部署。

   ![1551271233048](assets\1551271233048.png)

##  Linux管理

### Linux ssh 免密登录

通过RSA加密算生成了密钥，包括私钥和公钥，我们把公钥追加到用来认证授权的key中去。

每台机器配置本地免密登录，然后将其余每台机器生成的~/.ssh/id_dsa.pub公钥内容追加到其中一台主机的authorized_keys中，然后将这台机器中包括每台机器公钥的authorized_keys文件发送到集群中所有的服务器。这样集群中每台服务器都拥有所有服务器的公钥，这样集群间任意两台机器都可以实现免密登录了。

目前有3台虚拟，fanl01为Master，fanl02为Slave01，fanl02为Slave02

1. 进入fanl01，使用命令`ssh-keygen`
2. 查看生成的密钥，` cd ~/.ssh/`,公钥为`id_rsa.pub`，私钥为`id_rsa`
3. 进入fanl02 和 fanl03 生成密钥后传输到fanl01，`scp id_rsa.pub root@fanl01:~/.ssh/fanl02key`。
4. 将文件追加`cat fanl02key >> authorized_keys`，重复操作，最终fanl01上的授权文件包括了所有服务器，然后将其转发给其他服务器，`scp authorized_keys root@fanl02:~/.ssh`
5. 退出ssh，输入命令： `logout`
6. 新增服务器，需要重复上面的操作，并且修改`/etc/hosts`