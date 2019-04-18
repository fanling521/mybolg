# HBase数据结构

## HBase的存储模型

### RowKey

与nosql数据库们一样,RowKey是用来检索记录的主键。访问HBase table中的行，只有三种方式：

1. 通过单个RowKey访问

2. 通过RowKey的range（正则）

3. 全表扫描

(RowKey)可以是任意字符串(最大长度是64KB，实际应用中长度一般为10-100bytes)，在HBase内部，RowKey保存为字节数组。存储时，数据按照RowKey的字典序(byte order)排序存储。设计RowKey时，要充分排序存储这个特性，将经常一起读取的行存储放到一起。(位置相关性)

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

### 写过程

### Flush过程

### 数据合并过程