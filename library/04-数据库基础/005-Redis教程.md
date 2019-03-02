# Redis教程

REmote DIctionary Server(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统。

它通常被称为数据结构服务器，因为值（value）可以是 字符串(String), 哈希(Hash), 列表(list), 集合(sets) 和 有序集合(sorted sets)等类型。

Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。

Redis 与其他 key - value 缓存产品有以下三个特点：

- Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
- Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
- Redis支持数据的备份，即master-slave模式的数据备份。

Redis 优势：

- 性能极高
- 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作
- Redis的所有操作都是原子性的
- Redis支持 publish/subscribe, 通知, key 过期等特性

## 安装Redis

将必须的文件上传到Linux服务器的root目录中，本例使用的`fanl02`

```bash
# 解压/opt目录下
[root@fanl01 ~]$ tar zxvf redis-5.0.3.tar.gz -C /opt
# 进入/opt/redis-5.0.3
[root@fanl01 ~]$ cd /opt/redis-5.0.3
# 编译依赖gcc，需要安装
[root@fanl01 ~]$ make MALLOC=libc
[root@fanl01 ~]$ make PREFIX=/usr/local/redis  install
```

## 启动和配置Redis

```bash
# 进入redis目录
[root@fanl01 ~]$ cd /usr/local/redis/bin
# Redis服务在启动的时候可以指定配置文件
[root@fanl01 ~]$ cp /opt/redis-5.0.3/redis.conf /usr/local/redis/
# 启动redis服务
[root@fanl01 ~]$ bin/redis-server redis.conf

################################# GENERAL #####################################

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
daemonize no
# 修改 守护进程
daemonize yes

# 然后以普通方式启动即可
[root@fanl02 redis]# bin/redis-server redis.conf
9107:C 01 Mar 2019 19:24:50.962 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
9107:C 01 Mar 2019 19:24:50.963 # Redis version=5.0.3, bits=64, commit=00000000, modified=0, pid=9107, just started
9107:C 01 Mar 2019 19:24:50.963 # Configuration loaded

# 客户端连接测试
[root@fanl01 ~]$ /usr/local/redis/bin/redis-cli

# redis关闭
[root@fanl01 ~]$ redis-cli (-p 端口号，默认6379) shutdown
```

成功后，如图所示：

![成功](assets/20190301191853.png)

以上启动后只在控制台出现，修改配置文件，实现后台执行，修改`redis.conf`中的

## Redis 配置

Redis 的配置文件名为 `redis.conf`

可以通过 **CONFIG** 命令查看或设置配置项。

语法：

```bash
redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME
# 获取全部
redis 127.0.0.1:6379> CONFIG GET *
# 外部网络可以访问
# 编辑配置 CONFIG SET CONFIG_SETTING_NAME NEW_CONFIG_VALUE
redis 127.0.0.1:6379> CONFIG set bind "192.168.157.131"
# 报错 (error) ERR Unsupported CONFIG parameter: bind
# 不能动态设置该参数，打开配置文件修改
# 添加多个ip，不写接受所有ip
bind 127.0.0.1 192.168.157.131
# 需要配合防火墙规则，可以参考001-Linux简介 防火墙配置
```

## Redis 键命令

Redis 键命令用于管理 redis 的键。

命令执行后输出 **(integer) 1**，否则将输出 **(integer) 0**

```bash
DEL key                  # 该命令用于在 key 存在时删除 key。
DUMP key                 # 序列化给定 key ，并返回被序列化的值
EXISTS key               # 检查给定 key 是否存在。
EXPIRE key seconds       # 为给定 key 设置过期时间，以秒计
EXPIREAT key timestamp   # 为给定 key 设置过期时间，时间戳
PEXPIRE key milliseconds # 设置 key 的过期时间以毫秒计。
PEXPIREAT key milliseconds-timestamp # 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计
KEYS pattern             # 查找所有符合给定模式( pattern)的 key
MOVE key db              # 将当前数据库的 key 移动到给定的数据库 db 当中。 （MOVE name 3）
PERSIST key              # 移除 key 的过期时间，key 将持久保持。
PTTL key                 # 以毫秒为单位返回 key 的剩余的过期时间。
TTL key                  # 秒为单位，返回给定 key 的剩余生存时间
RANDOMKEY                # 从当前数据库中随机返回一个 key 。
RENAME key newkey        # 修改 key 的名称
RENAMENX key newkey      # 仅当 newkey 不存在时，将 key 改名为 newkey 。修改成存在的key
TYPE key                 # 返回 key 所储存的值的类型。
```

[更多命令请参考：](https://redis.io/commands)

## Redis 数据备份与恢复

Redis **SAVE** 命令用于创建当前数据库的备份。

如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可

```bash
# 安装目录
fanl02:2>CONFIG get dir
 1)  "dir"
 2)  "/root"
```

## Redis 安全
我们可以通过 redis 的配置文件设置密码参数，这样客户端连接到 redis 服务就需要密码验证，这样可以让你的 redis 服务更安全。

```bash
fanl02:0>config get requirepass
 1)  "requirepass"
 2)  ""
 
 # 设置密码
 fanl02:0>config set requirepass "123456"
 
 # 验证密码
 AUTH 123456
```

## Redis 数据类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

### String（字符串）

string 是 redis 最基本的类型，你可以理解成与 Memcached 一模一样的类型，一个 key 对应一个 value。

string 类型是二进制安全的。意思是 redis 的 string 可以包含任何数据。比如jpg图片或者序列化的对象。

string 类型是 Redis 最基本的数据类型，**string 类型的值最大能存储 512MB**。

语法：SET 、GET

```bash
fanl02:0>set name "fanling"
"OK"
fanl02:0>get  name
"fanling"
fanl02:0>set name jack
"OK"
fanl02:0>get  name
"jack"
```

### Hash（哈希）

Redis hash 是一个键值(key=>value)对集合。

Redis hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象。

每个 hash 可以存储 2^32 -1 键值对（40多亿）。

语法：HMSET、HMGET

```bash
fanl02:0>hmset person name "fanling" age "22"
"OK"
fanl02:0>hmset person name "fanling" age "22"
"OK"
```

### List（列表）

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）

列表最多可存储 2^32 - 1 元素

```bash

```

### Set（集合）

Redis的Set是string类型的无序集合。



### zset(sorted set：有序集合)

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

zset的成员是唯一的,但分数(score)却可以重复。

## Redis面试题目

1. 什么是Redis？
2. Redis相比memcached有哪些优势？
3. Redis支持哪几种数据类型？
4. Redis主要消耗什么物理资源？
5. Redis的全称是什么？
6. Redis有哪几种数据淘汰策略？
7. Redis官方为什么不提供Windows版本？
8. 一个字符串类型的值能存储最大容量是多少？
9. 为什么Redis需要把所有数据放到内存中？
10. Redis集群方案应该怎么做？都有哪些方案？
11. Redis集群方案什么情况下会导致整个集群不可用？
12. MySQL里有2000w数据，redis中只存20w的数据，如何保证redis中的数据都是热点数据？
13. Redis有哪些适合的场景？
14. Redis支持的Java客户端都有哪些？官方推荐用哪个？
15. Redis和Redisson有什么关系？
16. Jedis与Redisson对比有什么优缺点？
17. Redis如何设置密码及验证密码？
18. 说说Redis哈希槽的概念？
19. Redis集群的主从复制模型是怎样的？
20. Redis集群会有写操作丢失吗？为什么？
21. Redis集群之间是如何复制的？
22. Redis集群最大节点个数是多少？
23. Redis集群最大节点个数是多少？
24. 怎么测试Redis的连通性？
25. Redis中的管道有什么用？
26. 怎么理解Redis事务？
27. Redis事务相关的命令有哪几个？
28. Redis key的过期时间和永久有效分别怎么设置？
29. Redis如何做内存优化？
30. Redis回收进程如何工作的？
31. Redis回收使用的是什么算法？ 
32. Redis如何做大量数据插入？
33. 为什么要做Redis分区？
34. 你知道有哪些Redis分区实现方案？
35. Redis分区有什么缺点？
36. Redis持久化数据和缓存怎么做扩容？
37. 分布式Redis是前期做还是后期规模上来了再做好？为什么？
38. Twemproxy是什么？
39. 支持一致性哈希的客户端有哪些？
40. Redis与其他key-value存储有什么不同？
41. Redis的内存占用情况怎么样？
42. 都有哪些办法可以降低Redis的内存使用情况呢？
43. 查看Redis使用情况及状态信息用什么命令？
44. Redis的内存用完了会发生什么？
45. Redis是单线程的，如何提高多核CPU的利用率？
46. 一个Redis实例最多能存放多少的keys？List、Set、Sorted Set他们最多能存放多少元素？
47. Redis常见性能问题和解决方案？
48. Redis提供了哪几种持久化方式？
49. 如何选择合适的持久化方式？
50. 修改配置不重启Redis会实时生效吗？

答案参考：[Redis 面试题](https://blog.csdn.net/u010682330/article/details/81043419)