# Linux配置

本教程仅仅适用于有一定基础的Linux使用人员。

## 什么是Linux

 Linux是一个开源的操作系统。由内核、shell、文件系统和应用程序组成，Linux是指Linux内核，有很多发行版，常见有Ubantu、CentOS、RedHat等。

## 虚拟机中安装Linux

下载**CentOS7**的官方网站最小化的安装源，启动最小化安装。

安装和配置网络模式为`NAT`，当前的电脑安装虚拟机的虚拟网卡网段为`192.168.157.0`，查看虚拟机的网络设置，查看NAT网卡的网段和网关。

规定：

| 名称    | 值              |
| ------- | --------------- |
| IPADDR  | 192.168.157.160 |
| GATEWAY | 192.168.157.2   |
| NETMASK | 255.255.255.0   |
| DNS1    | 192.168.157.2   |

### 配置主机名

可以在安装过程中指定，但是如果是已经安装好的或者克隆的主机，需要使用命令模式来更改。

```shell
# =====查看主机名========
[root@centos01 ~]$ hostname
# =====设置主机名========
[root@centos01 ~]$ hostnamectl set-hostname [yourname]
# =====配置文件中的主机名========
[root@centos01 ~]$ cat /etc/hostname
```

### 配置网络

可以在安装过程中指定，但是如果是已经安装好的或者克隆的主机，需要使用命令模式来更改。

```shell
# =====配置文件路径========
[root@centos01 ~]$ vi /etc/sysconfig/network-scripts/ifcfg-ens33
# =====修改配置文件========
# =====添加========
IPADDR=192.168.157.131
NETMASK=255.255.255.0
GATEWAY=192.168.157.2
#  =====注意是DNS1 不是 DNS========
DNS1=192.168.157.2
# =====修改========
ONBOOT=yes
BOOTPROTO=static
# =====重启网络服务========
[root@centos01 ~]$ service network [restart|stop|start]
# =====查看网络========
[root@centos01 ~]$ ip addr
# =====或者，但是centos7需要安装net-tools========
[root@centos01 ~]$ ifconfig
```

### 配置映射

和windows一样，配置主机名和IP地址的映射，可以用主机名访问主机。

```shell
# =====配置文件路径========
[root@centos01 ~]$ vi /etc/hosts
# =====配置信息的格式========
192.168.157.160 centos7
```

### 配置防火墙和SELINUX

为了方便内网通讯，减少不必要的麻烦，建议关闭防火墙和SELINUX

注意：CentOS7内置的防火墙已经从`iptables`变成了`firewalld`

```shell
# =====查看防火墙的状态========
[root@centos01 ~]$ systemctl status firewalld
# =====开机禁用防火墙，启用enable========
[root@centos01 ~]$ systemctl disable firewalld
# =====临时关闭防火墙========
[root@centos01 ~]$ systemctl [start|stop|restart] firewalld
# =====增加端口规则========
[root@centos01 ~]$ firewall-cmd --zone=public --add-port=6379/tcp --permanent
```

SELINUX是安全子系统

```shell
# =====查看SELINUX的状态========
[root@centos01 ~]$ sestatus
# =====配置文件的路径========
[root@centos01 ~]$ vi /etc/sysconfig/selinux
# =====修改========
SELINUX=disabled
# =====临时关闭========
[root@centos01 ~]$ setenforce 0
```

## 用户管理

由于`root`用户权限过大，实际操作的时候不能使用该用户，需要重新创建一个用户。

规定：创建组`hadoop`，创建用户`fanl`

### Linux用户分类

用户信息存放在配置文件，路径为 `/etc/passwd`。

组的信息存放在配置文件，路径为`/etc/group`。

- UID为0的用户都是**超级用户**，只要把`/etc/passwd`相应的用户的UID改为0，该用户就变成超级用户了
- uid 500-60000的为**普通用户**
- uid 1-499是与系统和程序服务相关的**伪（系统）用户**

### 用户和组管理

```bash
# =====新增用户fanl========
[root@centos01 ~]$ adduser fanl
# =====删除用户fanl========
[root@centos01 ~]$ userdel fanl 
# =====修改密码，修改为【1q2w3e】========
[root@centos01 ~]$ passwd fanl
# =====新增用户组========
[root@centos01 ~]$ groupadd hadoop
# =====删除用户组========
[root@centos01 ~]$ groupdel hadoop
# =====查看组员========
[root@centos01 ~]$ cat /etc/group | grep hadoop
# =====添加组员========
[root@centos01 ~]$ gpasswd -a hadoop zhongguo
# =====删除组员========
[root@centos01 ~]$ gpasswd -d hadoop fanl

```

其他用户和组的常用命令

```shell
# =====查看用户所属组========
[root@centos01 ~]$ groups fanl
# =====查看用户信息======== 
[root@centos01 ~]$ id fanl
[root@centos01 ~]$ who
# =====查看某一个用户======== 
[root@centos01 ~]$ W fanl
```

### 赋予用户sudo权限

```shell
# =====配置文件地址，/etc/sudoers 快速进入========
[root@centos01 ~]$ visudo
# =====修改内容========
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
# =====添加内容========
fanl    ALL=(ALL)       ALL
fanl    ALL=(ALL)    NOPASSWD: ALL 
# =====普通用户执行root的命令的时候需要输入密码========
```

### SSH无密码登录

通过RSA加密算生成了密钥，包括私钥和公钥，我们把公钥追加到用来认证授权的key中去。

确保以下配置文件已经修改 `/etc/ssh/sshd_conf`

```bash
AuthorizedKeysFile      .ssh/authorized_keys
PubkeyAuthentication yes
```

1. 使用命令`ssh-keygen`生成rsa密钥
2. `ssh-copy-id hostname`，先统一追加到主机上，然后再分发。
3. 进入.ssh目录`scp authorized_keys username@hostname:/home/username/.ssh/`

**非root用户发现无效，检查权限问题**

```bash
[fanl@hadoop1 ~]$ chmod 700  ~/.ssh
[fanl@hadoop1 ~]$ chmod 600  ~/.ssh/authorized_keys
```

其他用户也按照以上方法配置。