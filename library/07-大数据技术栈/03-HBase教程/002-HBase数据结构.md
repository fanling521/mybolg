# HBase数据结构

## HBase的存储模型

（1）行键Row Key：行键可以是任意字符，最大长度64KB，存储时按照Row Key的字典序排序存储。

（2）列族Column Family：HBase表中的每个列，都归属于某个列族。列族是表的一部分（而列不是），必须在使用表之前定义。列名都以列族作为前缀，例如 courses:history，courses:math都属于courses 这个列族。

（3）列Column：列都属于某个列族。

（4）单元格Cell：由{rowkey, column Family:columu,version} 唯一确定的单元。cell中的数据是**没有类型**的，全部是**字节码形式**储存。

（5）时间戳TimeStamp：每个 cell都保存 着同一份数据的多个版本，版本通过时间戳来索引，不同版本的数据按照时间倒叙排序。

（6）命名空间Namespace：包含了表和各种权限。

## HBase原理⭐

### region的内部结构

![内部结构](assets/20190422205029.png)

1. 每个region是由1个Hlog及1到多个store组成，每个region中store的数量等于该表的列族的数量，每个store下存储了某一个列簇下的所有的数据
2. 每个store是由1个memstore及零到多个storefile组成
   1. memstore：写缓存，用来加快HBase表的写速度 ，阈值是128M
   2. storefile：由memstore中flush出，storefile文件就是HBase表数据的最终存储文件 
   3. Hlog文件：写数据时默认情况下数据会先写入到对应region的Hlog中进行备份，当因为regionserver节点宕机导致memstore中的数据丢失时可以从Hlog进行恢复

### HBase的读过程

1. 客户端先去访问Zookeeper，从Zookeeper里面获取meta表的region所在的regionserver节点信息
2. 客户端向meta表的region所在regionserver发起读请求，读取meta表数据，获取HBase集群上所有用户表的元数据
3. 根据meta表中用户表的region的位置信息，客户端找到当前需要访问的用户表对应的region及所在的regionserver服务器信息
4. 客户端向对应regionserver服务器发起读请求
5. regionserver收到客户端请求，在当前节点上找到用户表的region，并确定目标store，先扫描memstore（因为数据此时可能存在memstore并未flush成storeFile），再扫描blockcache（读缓存，近期已经读过的数据会加载到该缓冲区中），最后再读取storefile文件，regionserver把数据响应给客户端

### HBase的写过程

1. 客户端先去访问zookeeper,从zookeeper里面获取meta表的region所在的regionserver节点信息
2. 客户端向meta表的region所在regionserver发起读请求，读取meta表数据，获取HBase集群上所有用户表的元数据
3. 根据meta表中用户表的region的位置信息，客户端找到当前需要访问的用户表对应的region及所在的regionserver服务器信息
4. regionserver收到client写请求并响应，默认情况下regionserver先把数据写入对应region的HLog中，防止数据丢失再把数据写入到目标store
5. 当memstore内的数据量达到阈值[128M]，会把memstore里面的数据Flush到HDFS上形成storefile（Hfile）文件
6. 当持续写入数据，导致store下的storefile越来越多，当store文件数量达到3个进行小合并，且每间隔7天左右则会触发大合并操作，HBase会把每个store下的所有的storefile合并成一个大的单一的storefile
7. 当大合并后的单个storefile文件越来越大，达到一定阈值（10G或其他小于10G的动态阈值）时会触发split操作，region被一分为二，并由master将新的region分配给regionserver节点管理 

#### flush机制

1. 当memstore内的数据量达到阀值[128M]，会把memstore里面的数据Flush成storefile
2. 默认值为 `regionserver-jvm-heap`堆内存空间的40% ，当一个regionserver节点上的所有的memstore内缓存的数据量超过`regionserver-jvm-heap`堆内存空间的40%时，将会阻塞此regionserver节点上所有的region的更新操作并进行强制flush操作（优先选择占用空间大的memstore进行flush ）
3. 当一个regionserver节点上的所有的memstore内缓存的数据量超过`regionserver-jvm-heap`堆内存空间的38%时，会进行强制的flush操作。

#### compaction机制 

**原因**：当flush到store下的storefile文件过多过小将影响到HBase的读效率

HBase后启动compaction合并机制，最终将每个store下的所有的storeFile文件合并为一个大的单一的storeFile文件。

compaction的两个合并过程：

- minor compaction ->小合并 ：当store文件数量达到3个时会进行小合并，只是一个归类合并没有用扫描读取文件内的内容也没有进行排序操作
- major compaction ->大合并 ：所有的region每隔3.5天~10.5天之间会进行一次大合并

#### split机制

当大合并后的某个region下的某个store下的storefile文件大小达到一定的阈值，则会引起该region的split分割。

控制策略：

1. 某台regionserver上持有某个表的region个数为1个时，split切分阈值为flush size * 2
2. 其他情况为`hbase.hregion.max.filesize`=10G 

## 相关问题

（1）为什么在建表时表的列簇的数量不宜过多？