# canal

![](./images/canal.png)

译意为水道/管道/沟渠，主要用途是**基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费**

也就是等于一个消息队列，且只能是增量,本质上是基于日志来的。

- canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送 dump 协议
- MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )
- canal 解析 binary log 对象(原始为 byte 流)

### 结构

- server 代表一个 canal 运行实例，对应于一个 jvm
- instance 对应于一个数据队列 （1个 canal server 对应 1..n 个 instance )
- instance 下的子模块
  - eventParser: 数据源接入，模拟 slave 协议和 master 进行交互，协议解析
  - eventSink: Parser 和 Store 链接器，进行数据过滤，加工，分发的工作
  - eventStore: 数据存储
  - metaManager: 增量订阅 & 消费信息管理器

### 类图设计

- CanalLifeCycle: 所有 canal 模块的生命周期接口
- CanalInstance: 组合 parser,sink,store 三个子模块，三个子模块的生命周期统一受 CanalInstance 管理
- CanalServer: 聚合了多个 CanalInstance

## 启动deployer

https://github.com/alibaba/canal/releases/tag/canal-1.1.6

### 1）首先需要调整MySQL binlog日志模式

```ini
[mysqld]
log-bin=mysql-bin # 开启 binlog
binlog-format=ROW # 选择 ROW 模式
server_id=1 # 配置 MySQL replaction 需要定义，不要和 canal 的 slaveId 重复
```

1. **STATEMENT模式**只记录了sql语句，但是没有记录上下文信息，在进行数据恢复的时候可能会导致数据的丢失情况
2. **ROW模式**除了记录sql语句之外，还会记录每个字段的变化情况，能够清楚的记录每行数据的变化历史，但是会占用较多的空间，需要使用mysqlbinlog工具进行查看。
3. **MIX模式**比较灵活的记录，例如说当遇到了表结构变更的时候，就会记录为statement模式。当遇到了数据更新或者删除情况下就会变为row模式

还需要准备账号，用于复制监听

```mysql
CREATE USER canal IDENTIFIED BY 'canal';  
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
FLUSH PRIVILEGES;
```

### 2）调整配置启动

这边是直接在源码deployer下面启动运行

**默认起一个实例，名字为example,则更新 conf/example/instance.properties文件**

com.alibaba.otter.canal.deployer.CanalLauncher#main

```ini
## mysql serverId
canal.instance.mysql.slaveId = 1234
#position info，需要改成自己的数据库信息
canal.instance.master.address = 127.0.0.1:3306 
canal.instance.master.journal.name = 
canal.instance.master.position = 
canal.instance.master.timestamp = 
#canal.instance.standby.address = 
#canal.instance.standby.journal.name =
#canal.instance.standby.position = 
#canal.instance.standby.timestamp = 
#username/password，需要改成自己的数据库信息
canal.instance.dbUsername = canal  
canal.instance.dbPassword = canal
canal.instance.defaultDatabaseName =
canal.instance.connectionCharset = UTF-8
#table regex
canal.instance.filter.regex = .\*\\\\..\*
```

```
2023-06-05 14:25:10.592 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## start the canal server.
2023-06-05 14:25:10.803 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[192.168.95.115(192.168.95.115):11111]
```

这里默认把meta信息保留到了数据库中，默认是存放在h2中

```ini
#canal.instance.tsdb.spring.xml = classpath:spring/tsdb/h2-tsdb.xml
canal.instance.tsdb.spring.xml = classpath:spring/tsdb/mysql-tsdb.xml
```

```ini
canal.instance.tsdb.enable=true
canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/canal_tsdb
canal.instance.tsdb.dbUsername=root
canal.instance.tsdb.dbPassword=123456
```

### 3）启动测试客户端测试

com.alibaba.otter.canal.example.SimpleCanalClientTest#main

## 直接同步数据到kafka

https://github.com/alibaba/canal/wiki/Canal-Kafka-RocketMQ-QuickStart

```ini
#conf/example/instance.properties
canal.mq.topic=example
# 针对库名或者表名发送动态topic
#canal.mq.dynamicTopic=mytest,.*,mytest.user,mytest\\..*,.*\\..*
canal.mq.partition=0
# hash partition config
#canal.mq.partitionsNum=3
#库名.表名: 唯一主键，多个表之间用逗号分隔
#canal.mq.partitionHash=mytest.person:id,mytest.role:id
```

注：上面主要配置topic名称和规则，分区情况



```ini
# canal.properties 
# 可选项: tcp(默认), kafka,RocketMQ,rabbitmq,pulsarmq
canal.serverMode = kafka
# ...

# Canal的batch size, 默认50K, 由于kafka最大消息体限制请勿超过1M(900K以下)
canal.mq.canalBatchSize = 50
# Canal get数据的超时时间, 单位: 毫秒, 空为不限超时
canal.mq.canalGetTimeout = 100
# 是否为flat json格式对象
canal.mq.flatMessage = false
```

## client

> 原有sever默认是tcp，所以客户端可以链接使用，通过给定的client-api做任何想做的事情，
>
> 如果不先使用tcp，则可以启动server的时候，使用kafka也是可以的。

```java
// 根据ip，直接创建链接，无HA的功能
        String destination = "example";
        String ip = AddressUtils.getHostIp();
//配置地址
        CanalConnector connector = CanalConnectors.newSingleConnector(new InetSocketAddress(ip, 11111),
            destination,
            "canal",
            "canal");

        final SimpleCanalClientTest clientTest = new SimpleCanalClientTest(destination);
        clientTest.setConnector(connector);
//自己链接
        clientTest.start();
        Runtime.getRuntime().addShutdownHook(new Thread() {

            public void run() {
                try {
                    logger.info("## stop the canal client");
                    clientTest.stop();
                } catch (Throwable e) {
                    logger.warn("##something goes wrong when stopping canal:", e);
                } finally {
                    logger.info("## canal client is down.");
                }
            }

        });
```



## 答疑

1）一个数据库实例可以启动多个deployer,一个deployer可以配置n个instance(实例) 即通道

```ini
canal.destinations = example1,example2
```

2）默认启动的instance可以理解为一个服务端，所以可以有多个客户端，来适配实现

3）deployer可以直接部署发送消息到kafka。

4）服务如何启动或者重新拉取：

- canal.instance.master.journal.name +  canal.instance.master.position :  精确指定一个binlog位点，进行启动
- canal.instance.master.timestamp :  指定一个时间戳，canal会自动遍历mysql binlog，找到对应时间戳的binlog位点后，进行启动
- 不指定任何信息：默认从当前数据库的位点，进行启动。(show master status)

5）https://github.com/alibaba/canal/wiki/AdminGuide 更多使用参考

目前默认支持的instance.xml有以下几种：

1. spring/memory-instance.xml
2. spring/default-instance.xml
3. spring/group-instance.xml

在介绍instance配置之前，先了解一下canal如何维护一份增量订阅&消费的关系信息：

- 解析位点 (parse模块会记录，上一次解析binlog到了什么位置，对应组件为：CanalLogPositionManager)
- 消费位点 (canal server在接收了客户端的ack后，就会记录客户端提交的最后位点，对应的组件为：CanalMetaManager)

对应的两个位点组件，目前都有几种实现：

- memory  (memory-instance.xml中使用)
- zookeeper
- mixed  
- period  (default-instance.xml中使用，集合了zookeeper+memory模式，先写内存，定时刷新数据到zookeeper上)

group-instance.xml介绍：

  主要针对需要进行多库合并时，可以将多个物理instance合并为一个逻辑instance，提供客户端访问。

  场景：分库业务。 比如产品数据拆分了4个库，每个库会有一个instance，如果不用group，业务上要消费数据时，需要启动4个客户端，分别链接4个instance实例。使用group后，可以在canal server上合并为一个逻辑instance，只需要启动1个客户端，链接这个逻辑instance即可. 