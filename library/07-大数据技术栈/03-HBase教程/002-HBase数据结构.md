# HBase数据结构

## HBase的存储模型

### RowKey

与nosql数据库们一样,RowKey是用来检索记录的主键。访问HBase table中的行，只有三种方式：

1. 通过单个RowKey访问

2. 通过RowKey的range（正则）

3. 全表扫描

可以是任意字符串（最大长度是64KB，实际应用中长度一般为10-100bytes），在HBase内部，RowKey保存为字节数组。存储时，数据按照RowKey的字典序(byte order)排序存储。设计RowKey时，要充分排序存储这个特性，将经常一起读取的行存储放到一起。(位置相关性)

### Column Family

**列族**：HBase表中的每个列，都归属于某个列族。列族是表的schema的一部 分（而列不是），必须在使用表之前定义。

列名都以列族作为前缀。例如 courses:history，courses:math都属于courses 这个列族。

### Column

列都属于某个列族。

### Cell

由{rowkey, column Family:columu,version} 唯一确定的单元。cell中的数据是**没有类型**的，全部是**字节码形式**从储存。

### Time Stamp

每个 cell都保存 着同一份数据的多个版本。版本通过时间戳来索引。

### Namespace

包含了表和各种权限。

## HBase原理⭐

### 读过程

1. Client先访问Zookeeper，从meta表读取region的位置，然后读取meta表中的数据，meta中又存储了用户表的region信息
2. 根据namespace、表名和rowkey在meta表中找到对应的region信息
3. 找到这个region对应的regionserver，查找对应的region
4. 先从MemStore找数据，如果没有，再到BlockCache里面读，BlockCache还没有，再到StoreFile上读
5. 如果是从StoreFile里面读取的数据，不是直接返回给客户端，而是先写入BlockCache，再返回给客户端

### 写过程

1. Client向HregionServer发送写请求
2. HregionServer将数据写到HLog
3. HregionServer将数据写到MemStore
4. 反馈Client写成功

### Flush过程

1. 当MemStore数据达到阈值（默认是128M），将数据刷到硬盘，将内存中的数据删除，同时删除HLog中的历史数据
2. 并将数据存储到HDFS中
3. 在HLog中做标记点

### 数据合并过程

1. 当数据块达到4块，Hmaster触发合并操作，Region将数据块加载到本地，进行合并
2. 当合并的数据超过256M，进行拆分，将拆分后的Region分配给不同的HregionServer管理
3. 当HregionServer宕机后，将HregionServer上的Hlog拆分，然后分配给不同的HregionServer加载，修改.META.