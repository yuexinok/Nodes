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

### 安装和启动：

```shell
https://www.elastic.co/guide/en/elasticsearch/reference/7.14/brew.html
brew install elastic/tap/elasticsearch-full

```

注：https://127.0.0.1:9200 

```json
{
  "name" : "77a082ffcbdc",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "zu9QN9zBTAmsbpfUQZdpDA",
  "version" : {
    "number" : "7.14.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "dd5a0a2acaa2045ff9624f3729fc8a6f40835aa1",
    "build_date" : "2021-07-29T20:49:32.864135063Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

```shell
docker network create elastic
docker pull docker.elastic.co/elasticsearch/elasticsearch:7.14.0
docker run --name es01-test --net elastic -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.14.0

docker pull docker.elastic.co/kibana/kibana:7.14.0
docker run --name kib01-test --net elastic -p 5601:5601 -e "ELASTICSEARCH_HOSTS=http://es01-test:9200" docker.elastic.co/kibana/kibana:7.14.0
```

注：http://localhost:5601/  访问kibana



