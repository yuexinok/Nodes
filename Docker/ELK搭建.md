## 准备：

```shell
首先下载
https://github.com/deviantony/docker-elk
➜  elk ls
docker-elk-master docker-elk.zip
➜  elk pwd
/Users/borgxiao/docker/elk

```

## 运行启动：

```shell
➜  docker-elk-master ls
LICENSE            docker-compose.yml extensions         logstash
README.md          elasticsearch      kibana
➜  docker-elk-master docker-compose up -d
Creating network "docker-elk-master_elk" with driver "bridge"
Building elasticsearch
Step 1/1 : FROM docker.elastic.co/elasticsearch/elasticsearch-oss:6.2.3
6.2.3: Pulling from elasticsearch/elasticsearch-oss

访问：
http://127.0.0.1:5601/

```



## 端口说明：

```shell
5000: Logstash TCP input.
9200: Elasticsearch HTTP
9300: Elasticsearch TCP transport
5601: Kibana
```

