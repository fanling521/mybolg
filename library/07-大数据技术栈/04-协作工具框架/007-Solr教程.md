# Solr教程

## 什么是Solr

Apache Solr是一个功能强大的搜索服务器，它支持REST风格API。Solr是基于Lucene的，Lucene 支持强大的匹配能力，如短语，通配符，连接，分组和更多不同的数据类型。它使用 Apache Zookeeper特别针对高流量进行优化。Apache Solr提供各式各样的功能，我们列出了部分最主要的功能。

1.  先进的全文搜索功能。
2.  XML，JSON和HTTP - 基于开放接口标准。
3.  高度可扩展和容错。
4.  同时支持模式和无模式配置。
5.  分页搜索和过滤。
6.  支持像英语，德语，中国，日本，法国和许多主要语言
7.  丰富的文档分析。

## 安装部署Solr

（1）zip包安装

解压完成后，cmd进入bin目录，执行

```bash
solr start | restart | stop  # 指定端口 -p 8888
```

打开[http://localhost:8983](http://localhost:8983/)浏览管理端。

## 配置Apache Solr

### 配置Core

当Solr的服务器在独立模式下启动的配置称为核心，当它在SolrCloud模式启动的配置称为集合，每个核心下都有自己的索引库和与之相应的配置文件。

（1）建立核心（core）

Solr的创建命令有以下选项：

1.  **-c <**name**>** -要创建的核心或集合的名称（必需）
2.  **-d <**confdir**>** -配置目录，在SolrCloud模式非常有用
3.  **-n <**configName**>** -配置名称。默认为核心或集合的名称
4.  **-p <**port**>** -本地Solr的实例的端口发送create命令， 默认脚本试图通过寻找运行Solr的实例来检测端口
5.  **-s <**shards**>** -要将集合拆分为的碎片数，默认值是1
6.  **-rf <**replicas**>** -集合中的每个文件的份数，默认值是1

```bash
solr create -c FANL
# 输出
# Created new core 'FANL'
```

### 配置Schema

schema是用来告诉solr如何建立索引的，他的配置围绕着一个schema配置文件，这个配置文件决定着solr如何建立索引，每个字段的数据类型，分词方式等。

新版本不需要手动编辑`schema.xml`，而是使用schemaAPI来配置`managed-schema`，无需重启服务即可完成配置。

schema的成员信息：

- **fieldType**：为field定义类型，最主要作用是定义分词器，分词器决定着如何从文档中检索关键字
- **analyzer**：他是fieldType下的子元素，这就是传说中的分词器，他由一组tokenizer和filter组成
- **field**：他是创建索引用的字段，如果想要这个字段生成索引需要配置他的indexed属性为true，stored属性为true表示存储该索引

路径为`solr-8.1.1\server\solr\FANL\conf\managed-schema`