# Nginx教程 

## 什么是Nginx

### Nginx的核心优点

- 高并发、高性能
- 可扩展性好
- 热部署
- BSD许可证

### Nginx的组成

- 二进制执行文件
- nginx.conf配置文件，控制Nginx的行为
- access.log 记录处理过的请求
- error.log

## Linux安装Nginx

### yum安装

（1）添加源

```bash
[root@fanling /]$ rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

（2）安装

```bash
[root@fanling /]$ yum search nginx
[root@fanling /]$ yum install -y nginx
```

（3）启动

```bash
[root@fanling /]$ systemctl start nginx.service
[root@fanling /]$ systemctl enable nginx.service
```

（3）卸载

```bash
yum remove nginx
```

目录：`/etc/nginx`

默认的配置文件：`/etc/nginx/conf.d`

前端目录：`/usr/share/nginx/html`

日志：`/var/log/nginx`

### 编译安装



## 配置文件

