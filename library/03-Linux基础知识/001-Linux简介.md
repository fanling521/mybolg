# Linux简介

## 什么是Linux

## 虚拟机安装Linux

> 最小化安装CentOS 7 

1. 下载安装源，启动最小化安装，第一台linux 命名为"fanl01"，密码为 fanling，依次类推，克隆3个

2. 安装和配置网路模式为NAT，当前虚拟机网卡网段为`192.168.177.0`

   | 192.168.177.130 | fanl01 |
   | --------------- | ------ |
   | 192.168.177.131 | fanl02 |
   | 192.168.177.132 | fanl03 |
   | 192.168.177.133 | fanl04 |

   

3. 保存快照，克隆虚拟机，重命名主机名等，保证工具可连接linux

   1. 查看主机名： `hostname`

   2. 重命名主机名：(1)命令：`hostname fanl01`；（2）修改*/etc/sysconfig/network*  配置文件，修改 HOSTNAME=fanl01；（3）修改本机的域名解析文件 */etc/hosts*，将localhost.xx改为localhost.fanl01

   3. 配置网络：*/etc/sysconfig/network-scripts/ifcfg-eth0* 将虚拟机nat地址修改到虚拟机上。添加`IPADDR,NETMASK,GATWAY`以上的数据来源于虚拟的虚拟网卡v8，并且关闭**防火墙**（查看网络ip的命令为`ip addr`）

   4. 添加主机名和ip映射关系，将集群内的ip和主机名一一映射在hosts中，如`192.168.177.130 fanl01`

4. 完成集群linux 虚拟机器部署。