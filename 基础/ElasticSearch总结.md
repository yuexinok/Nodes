## 简介：

慕课网：ElasticSearch，讲师：瓦力

ElasticSearch是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。Elasticsearch是用Java开发的，并作为Apache许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于[云计算](https://baike.baidu.com/item/%E4%BA%91%E8%AE%A1%E7%AE%97)中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。

轻松的横向扩展，可支持PB级的结构化和非结构化数据处理。

源于：为老婆做一个搜索菜谱搜索。

### 应用场景：

英国卫报：实时分析公众对文章的回应。

维基百科，和GitHub站内实时搜索。

百度，实时日志分析。

### 版本问题：

1.x->2.x->**5.x**,因为：Lucene的版本问题，为了统一。

https://www.elastic.co/

head插件

集群分布式部署

## 基本概念：

### 索引：

含有相同属性的文档集合，数据库

### 类型：

可以定义一个或多个类型，文档必须属于一个类型  表

### 文档：

文档是可以被索引的基本数据单位  一行记录

### 分片：

每个索引都有多个分片，每个分片是一个Lucene索引。

### 备份：

拷贝一份分片就完成了分片的备份。

head插件里面的数字表示分片，其中标粗的表示主分片。

postman

==> Downloading https://artifacts.elastic.co/downloads/elasticsearch/elasticsear

######################################################################## 100.0%
==> Caveats
Data:    /usr/local/var/lib/elasticsearch/elasticsearch_borgxiao/
Logs:    /usr/local/var/log/elasticsearch/elasticsearch_borgxiao.log
Plugins: /usr/local/var/elasticsearch/plugins/
Config:  /usr/local/etc/elasticsearch/

➜  bin pwd
/usr/local/Cellar/elasticsearch/6.0.0_1/bin