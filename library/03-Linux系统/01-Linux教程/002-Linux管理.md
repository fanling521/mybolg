# Linux管理

整理常见的，常用的命令以及其用法

## Linux文件命令

### 目录切换

```bash
# =====切换到上层目录========
[root@centos01 ~]$ cd ..
# =====切换到系统跟目录========
[root@centos01 ~]$ cd /
# =====切换到用户主目录========
[root@centos01 ~]$ cd ~
# =====切换到xx目录绝对路径========
[root@centos01 ~]$ cd /xx 
# =====切换到xx/yy/zz目录========
[root@centos01 ~]$ cd /xx/yy/zz
```

### 创建和删除目录

```bash
# =====创建xx目录========
[root@centos01 ~]$ mkdir xx
# =====创建多级目录========
[root@centos01 ~]$ mkdir -p /opt/xx1/xx2/xx3
# ====删除目录========
[root@centos01 ~]$ rmdir xx
# ====强制删除目录========
[root@centos01 ~]$ rm -f xx
# =====创建多个目录========
[root@centos01 ~]$ mkdir xx1 xx2
# =====查看文件树，需要安装tree========
[root@centos01 ~]$ tree /opt
```

### 创建和查看文件

```bash
# =====查看xx文件========
[root@centos01 ~]$ cat xx
# =====创建xx文件========
[root@centos01 ~]$ cat >xx
# =====将xx1追加到yy中========
[root@centos01 ~]$ cat xx1 >> yy
# =====创建多个目录========
[root@centos01 ~]$ tail -n logfile
# =====动态查看文件内容========
[root@centos01 ~]$ tail -f logfile
# =====分页查看，回车：一行一行，空格：往下翻页，b：往上翻页========
[root@centos01 ~]$ more xx
# =====支持上下键和翻页键========
[root@centos01 ~]$ less xx
# =====默认前10行，可自定义n========
[root@centos01 ~]$ head -n xx
# =====创建文件========
[root@centos01 ~]$ touch fanl.txt
```

### 文件复制和移动

```bash
# =====将a.txt复制为b.txt========
[root@centos01 ~]$ cp a.txt b.txt
# =====拷贝目录========
[root@centos01 ~]$ cp -r /opt/xx1 /opt/xx2/
# =====远程复制========
[root@centos01 ~]$ scp 文件 用户名@IP/hostname:路径
# =====将a.txt 移动到上层xx目录中========
[root@centos01 ~]$ mv a.txt ../xx
# =====将a.txt 重命名为b.txt========
[root@centos01 ~]$ mv a.txt b.txt
```

### 文件搜索和统计

```shell
# =====按名字：查找 find [范围] [-参数] 模糊查询建议加引号========
[root@centos01 ~]$ find /etc -name init
# =====按大小+大于2M，-小于2M，不写表示等于========
[root@centos01 ~]$ find /etc -size +2M
# =====按类型：f 文件，d 目录========
[root@centos01 ~]$ find /etc -type f
# =====以上命令可以组合使用========
# =====统计字符========
[root@centos01 ~]$ wc -c /xxx/xx
```

### Linux特殊符号

```bash
# =====管道========
[root@centos01 ~]$ cat /etc/profile | grep xx
# =====追加 >>========
[root@centos01 ~]$ echo "22" >> a.txt
# =====覆盖 >========
[root@centos01 ~]$ echo "22" > b.txt
# =====命令换行 \========
[root@centos01 ~]$ cat \
> /etc/profile
```

### 文件的解压和压缩

```shell
# =====tar========
[root@centos01 ~]tar -cvf filename.tar dir # 将目录dir中压缩到filename.tar中，保留原文件
[root@centos01 ~]tar xvf filename.tar  # 将filename.tar解压到当前文件夹，保留原文件
# =====gz========
[root@centos01 ~]gzip filename # 不保留原文件
[root@centos01 ~]gzip -c filename > filename.gz # 保留原文件

[root@centos01 ~]gunzip filename.gz # 不保留原文件
[root@centos01 ~]gunzip -c filename.gz > filename # 保留原文件 
# =====tar.gz========
# =====最常用，解压到指定目录，要看到过程并且保留源文件========
[root@centos01 ~]tar -zxvf filename.tar.gz -C dir
```

### 目录和文件的权限修改

```bash
# =====权限的修改========
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
[root@centos01 ~] chomd 777 test.sh
[root@centos01 ~] chomd u+x test.sh
[root@centos01 ~] chomd +x test.sh
```

## Linux系统命令

若提示缺少`ifconfig`、`netstat`，需要安装`yum install net-tools`

```bash
[hadoop@centos01 ~]$ top           # 系统信息，资源管理器，3秒刷新，按q退出
[hadoop@centos01 ~]$ free          # 查看内存，-m 按M显示，-g按G显示
[hadoop@centos01 ~]$ df -l         # 查看硬盘分区和使用量
[hadoop@centos01 ~]$ ps-ef         # 查看系统进程
[hadoop@centos01 ~]$ kill -9 pid号 # 杀死进程，-9表示强制杀死
[hadoop@centos01 ~]$ ip addr       # 查看网络信息
[hadoop@centos01 ~]$ ifconfig      # 查看网络信息
[hadoop@centos01 ~]$ ping 主机名/ip地址 #检查网络联通
[hadoop@centos01 ~]$ netstat # 查看端口号
							 # -t 监控TCP协议的进程
							 # -l 查看端口是否被监听
							 # -n 显示端口号信息
							 # -p 显示进程PID号
[root@centos01 ~]$ netstat -antp # 查看进程信息
[root@centos01 ~]$ netstat -antp | grep 80    # 查看80端口
[root@centos01 ~]$ netstat -antp | grep hbase # 查看hbase端口信息
[root@centos01 ~]$ vmstat 2 5                 # 每2秒采集5次系统信息
[root@centos01 ~]$ chkconfig --list           # 查看某些服务的开机自启动状态
```

## Linux安装软件

### yum/wget在线安装

```bash
[root@centos01 ~]$ yum list   # 查看yum源仓库中可安装的软件
[root@centos01 ~]$ yum list   # 查看yum源仓库中可安装的软件
[root@centos01 ~]$ yum list installed   # 查看系统中已经安装好的rpm包 rpm -qa 
[root@centos01 ~]$ yum list  installed  | grep tree  
[root@centos01 ~]$ yum list | grep rzsz # 查看安装
[root@centos01 ~]$ yum  -y remove lrzsz.x86_64 #卸载
```

**替换yum源**

由于yum源默认在国外，为了增加访问速度，修改为阿里源。

```bash
[root@centos01 ~]$ cd /etc/yum.repos.d/
[root@centos01 ~]$ mv CentOS-Base.repo CentOS-Base.repo.bak

[root@centos01 ~]$ wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

### 离线安装rpm

```bash
[root@centos01 ~]$ rpm -qa   #所有的
[root@centos01 ~]$ rpm -e --nodeps xx # --nodeps 表示不检查依赖关系，强制卸载
[root@centos01 ~]$ rpm -ivh xx.rpm #-vh 表示显示安装过程，-i  表示忽略大小写
```

### 压缩包离线安装JDK

**第一步**：下载压缩包，并且上传到Linux中的`/opt/software`

**第二步**：解压

```bash
[fanl@centos01 software]$ tar -zxvf jdk-8u201-linux-x64.tar_2.gz -C /opt/module/
# 记住当前路径
[fanl@centos01 jdk1.8.0_201]$ pwd

/opt/module/jdk1.8.0_201
```

**第三步**：设置环境变量

```bash
[fanl@centos01 jdk1.8.0_201]$ sudo vi /etc/profile
# shift+g到最后一行
# 添加以下内容
export JAVA_HOME=/opt/module/jdk1.8.0_201
export PATH=$PATH:$JAVA_HOME/bin
##############################################
# 报存后使其生效
[fanl@centos01 jdk1.8.0_201]$ source /etc/profile
# 检测JAVA环境
[fanl@centos01 jdk1.8.0_201]$ java -version
```