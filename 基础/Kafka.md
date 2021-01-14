# Kafka实战：

Kafka是一款分布式消息发布和订阅系统，它的特点是高性能、高吞吐量。 最早设计的目的是作为LinkedIn的活动流和运营数据的处理管道。

一个典型的kafka集群包含若干Producer(可以是应用节点产生的消息，也可以是通过Flume收集日志 产生的事件)，若干个Broker(kafka支持水平扩展)、若干个Consumer Group，以及一个 zookeeper集群。kafka通过zookeeper管理集群配置及服务协同。Producer使用push模式将消息发布 到broker，consumer通过监听使用pull模式从broker订阅并消费消息。

多个broker协同工作，producer和consumer部署在各个业务逻辑中。三者通过zookeeper管理协调请 求和转发。这样就组成了一个高性能的分布式消息发布和订阅系统。

### 启动：

```shell
sh kafka-server-start.sh -damoen config/server.properties
sh kafka-server-stop.sh -daemon config/server.properties
```



## 名词解释：

### Broker：

Kafka集群包含一个或多个服务器，这种服务器被称为broker。broker端不维护数据的消费状态，提升 了性能。直接使用磁盘进行存储，线性读写，速度快:避免了数据在JVM内存和系统内存之间的复制， 减少耗性能的创建对象和垃圾回收。

### Producer：

负责发布消息到Kafka broker

```shell
 sh kafka-console-producer.sh --broker-list 192.168.244.128:9092 --topic first_topic
```



### Consumer：

消息消费者，向Kafka broker读取消息的客户端，consumer从broker拉取(pull)数据并进行处理

```shell
 sh kafka-console-consumer.sh --bootstrap-server 192.168.13.106:9092 --topic test --from-beginning
```

1. 如果consumer比partition多，是浪费，因为kafka的设计是在一个partition上是不允许并发的， 所以consumer数不要大于partition数

2. 如果consumer比partition少，一个consumer会对应于多个partitions，这里主要合理分配 consumer数和partition数，否则会导致partition里面的数据被取的不均匀。最好partiton数目是 consumer数目的整数倍，所以partition数目很重要，比如取24，就很容易设定consumer数目

3. 如果consumer从多个partition读到数据，不保证数据间的顺序性，kafka只保证在一个partition 上数据是有序的，但多个partition，根据你读的顺序会有不同

4. 增减consumer，broker，partition会导致rebalance，所以rebalance后consumer对应的 partition会发生变化

   当出现以下几种情况时，kafka会进行一次分区分配操作，也就是kafka consumer的rebalance

   1. 同一个consumer group内新增了消费者

   2. 消费者离开当前所属的consumer group，比如主动停机或者宕机
   3.  topic新增了分区(也就是分区数量发生了变化)

#### offset:

每个消息在被添加到分区时，都会被分配一个 offset(称之为偏移量)，它是消息在此分区中的唯一编号，kafka通过offset保证消息在分区内的顺 序，offset的顺序不跨分区，即kafka只保证在同一个分区内的消息是有序的; 对于应用层的消费来 说，每次消费一个消息并且提交以后，会保存当前消费到的最近的一个offset

在kafka中，提供了一个consumer_offsets_* 的一个topic，把offset信息写入到这个topic中。 consumer_offsets——按保存了每个consumer group某一时刻提交的offset信息。 __consumer_offsets 默认有50个分区。

### Consumer Group：

每个Consumer属于一个特定的Consumer Group(可为每个Consumer指定group name，若不指定 group name则属于默认的group)

#### 【group.id】

consumer group是kafka提供的可扩展且具有容错性的消费者机制。既然是一个组，那么组内必然可以 有多个消费者或消费者实例(consumer instance)，它们共享一个公共的ID，即group ID。组内的所有 消费者协调在一起来消费订阅主题(subscribed topics)的所有分区(partition)。当然，每个分区只能由 同一个消费组内的一个consumer来消费

#### coordinator(协调员)

Kafka提供了一个角色:coordinator来执行对于consumer group的管理，Kafka提供了一个角色: coordinator来执行对于consumer group的管理，当consumer group的第一个consumer启动的时 候，它会去和kafka server确定谁是它们组的coordinator。之后该group内的所有成员都会和该 coordinator进行协调通信。

consumer group如何确定自己的coordinator是谁呢, 消费者向kafka集群中的任意一个broker发送一个 GroupCoordinatorRequest请求，服务端会返回一个负载最小的broker节点的id，并将该broker设置 为coordinator

#### JoinGroup的过程

 在rebalance之前，需要保证coordinator是已经确定好了的，整个rebalance的过程分为两个步骤，Join和Sync

**join**: 表示加入到consumer group中，在这一步中，所有的成员都会向coordinator发送joinGroup的请 求。一旦所有成员都发送了joinGroup请求，那么coordinator会选择一个consumer担任leader角色， 并把组成员信息和订阅信息发送消费者

leader选举算法比较简单，如果消费组内没有leader，那么第一个加入消费组的消费者就是消费者 leader，如果这个时候leader消费者退出了消费组，那么重新选举一个leader，这个选举很随意，类似 于随机算法。

1）在joingroup阶段，每个consumer都会把自己支持的分区分配策略发送到coordinator

2）coordinator收集到所有消费者的分配策略，组成一个候选集 

3）每个消费者需要从候选集里找出一个自己支持的策略，并且为这个策略投票 

4）最终计算候选集中各个策略的选票数，票数最多的就是当前消费组的分配策略

#### consumer group rebalance的过程：

Ø 对于每个consumer group子集，都会在服务端对应一个GroupCoordinator进行管理， GroupCoordinator会在**zookeeper上添加watcher**，当消费者加入或者退出consumer group时，会修 改zookeeper上保存的数据，从而触发GroupCoordinator开始Rebalance操作。

Ø 当消费者准备加入某个Consumer group或者GroupCoordinator发生故障转移时，消费者并不知道 GroupCoordinator的在网络中的位置，这个时候就需要确定GroupCoordinator，消费者会向集群中的 任意一个Broker节点发送ConsumerMetadataRequest请求，收到请求的broker会返回一个response 作为响应，其中包含管理当前ConsumerGroup的GroupCoordinator，

Ø 消费者会根据broker的返回信息，连接到groupCoordinator，并且发送HeartbeatRequest，发送心 跳的目的是要GroupCoordinator这个消费者是正常在线的。当消费者在指定时间内没有发送 心跳请求，则GroupCoordinator会触发Rebalance操作。

### Topic：

每条发布到Kafka集群的消息都有一个类别，这个类别被称为Topic。(物理上不同Topic的消息分开存 储，逻辑上一个Topic的消息虽然保存于一个或多个broker上但用户只需指定消息的Topic即可生产或消 费数据而不必关心数据存于何处)

```shell
 #创建topic
 sh kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 -- partitions 1 --topic test
 
 #查看
 sh kafka-topics.sh --list --zookeeper localhost:2181
 #查看属性
 sh kafka-topics.sh --describe --zookeeper localhost:2181 --topic first_topic
 
```

### Parttion：

Parition（分区）是物理上的概念，每个Topic包含一个或多个Partition.

Topic在逻辑上可以被认为是一个queue，每条消费都必须指定它的Topic，可以简单理解为必须指明把 这条消息放进哪个queue里。为了使得Kafka的吞吐率可以线性提高，物理上把Topic分成一个或多个 Partition，每个**Partition在物理上对应一个文件夹**，该文件夹下存储这个**Partition的所有消息和索引文 件**。若创建topic1和topic2两个topic，且分别有13个和19个分区，则整个集群上会相应会生成共32个 文件夹。

每个topic可以划分多个分区(每个Topic至少有一个分区)，同一topic下的不同分区包含的消息是不同 的。每个消息在被添加到分区时，都会被分配一个offset(称之为偏移量)，它是消息在此分区中的唯 一编号，kafka通过offset保证消息在分区内的顺序，offset的顺序不跨分区，即**kafka只保证在同一个 分区内的消息是有序的**。

kafka采用的是hash取模的分区算法。如果Key为null，则会随机分配一个分区。这个随机 是在这个参数”metadata.max.age.ms”的时间范围内随机选择一个。对于这个时间段内，如果key为 null，则只会发送到唯一的分区。这个**值默认情况下是10分钟更新一次**。

#### 分区分配策略：

在kafka中，存在三种分区分配策略，一种是Range(默认)、 另一种是RoundRobin(轮询)、 StickyAssignor(粘性)。 在消费端中的ConsumerConfig中，通过这个属性来指定分区分配策略

##### **RangeAssignor(范围分区)**

假设n = 分区数/消费者数量
 m= 分区数%消费者数量 那么前m个消费者每个分配n+l个分区，后面的(消费者数量-m)个消费者每个分配n个分区

11个分区，那么最后分区分配的结果看起来是这样的:

 C1-0 将消费 0, 1, 2, 3 分区

C2-0 将消费 4, 5, 6, 7 分区

C3-0 将消费 8, 9, 10 分区

##### **RoundRobinAssignor(轮询分区)**

轮询分区策略是把所有partition和所有consumer线程都列出来，然后按照hashcode进行排序。最后通过轮询算法分配partition给消费线程。如果所有consumer实例的订阅是相同的，那么partition会均匀分布。

##### StrickyAssignor(粘滞策略)

分区的分配尽可能的均匀
分区的分配尽可能和上次分配保持相同

当两者发生冲突时， 第 一 个目标优先于第二个目标。 

#### 分区副本：

每个分区可以有多个副本，并且在副本集合中会存在一个leader的副本，所有的读写请求都是由leader 副本来进行处理。剩余的其他副本都做为follower副本，follower副本会从leader副本同步消息日志。

一般情况下，同一个分区的多个副本会被均匀分配到集群中的不同broker上，当leader副本所在的 broker出现故障后，可以重新选举新的leader副本继续对外提供服务。通过这样的副本机制来提高 kafka集群的可用性。

**LEO**:即日志末端位移(log end offset)，记录了该副本底层日志(log)中下一条消息的位移值。注意是下 一条消息!也就是说，如果LEO=10，那么表示该副本保存了10条消息，位移值范围是[0, 9]。另外， leader LEO和follower LEO的更新是有区别的。我们后面会详细说

**HW**:即上面提到的水位值。对于同一个副本对象而言，其HW值不会大于LEO值。小于等于HW值的所 有消息都被认为是“已备份”的(replicated)。同理，leader副本和follower副本的HW更新是有区别的

## JAVA库：

```xml
 <dependency>
   <groupId>org.apache.kafka</groupId>
   <artifactId>kafka-clients</artifactId> 
   <version>2.0.0</version>
</dependency>
```

### 同步/异步发送：

kafka对于消息的发送，可以支持同步和异步，同步会 需要阻塞，而异步不需要等待阻塞的过程。

从本质上来说，**kafka都是采用异步的方式来发送消息到broker**，但是kafka并不是每次发送消息都会直 接发送到broker上，而是**把消息放到了一个发送队列中，然后通过一个后台线程不断从队列取出消息进 行发送，发送成功后会触发callback**。kafka客户端会积累一定量的消息统一组装成一个批量消息发送出 去，触发条件是前面提到的batch.size和linger.ms

而**同步发送的方法，无非就是通过future.get()来等待消息的发送返回结果**，但是这种方法会严重影响消 息发送的性能。

### 常用参数：

#### 【batch.szie】

生产者发送多个消息到broker上的同一个分区时，为了减少网络请求带来的性能开销，通过批量的方式 来提交消息，可以通过这个参数来控制批量提交的字节数大小，默认大小是16384byte,也就是16kb， 意味着当一批消息大小达到指定的batch.size的时候会统一发送

#### 【linger.ms】

linger(徘徊)

Producer默认**会把两次发送时间间隔内收集到的所有Requests进行一次聚合然后再发送**，以此提高吞 吐量，而linger.ms就是为**每次发送到broker的请求增加一些delay，以此来聚合更多的Message请求**。 这个有点想TCP里面的Nagle算法，在TCP协议的传输中，为了减少大量小数据包的发送，采用了Nagle 算法，也就是**基于小包的等-停协议**。

#### 【**enable.auto.commit**】

消费者消费消息以后自动提交，只有当消息提交以后，该消息才不会被再次接收到，还可以配合 auto.commit.interval.ms控制自动提交的频率。

当然，我们也可以通过consumer.commitSync()的方式实现手动提交

#### 【**auto.offset.reset**】

这个参数是针对新的groupid中的消费者而言的，当有新groupid的消费者来消费指定的topic时，对于 该参数的配置，会有不同的语义

auto.offset.reset=latest情况下，新的消费者将会从其他消费者最后消费的offset处开始消费Topic下的 消息

auto.offset.reset= earliest情况下，新的消费者会从该topic最早的消息开始消费 

auto.offset.reset=none情况下，新的消费者加入以后，由于之前不存在offset，则会直接抛出异常。

#### 【**max.poll.records**】

此设置限制每次调用poll返回的消息数，这样可以更容易的预测每次poll间隔要处理的最大值。通过调 整此值，可以减少poll间隔

### Spring Java：

```xml
 <dependency> 
   <groupId>org.springframework.kafka</groupId> 
   <artifactId>spring-kafka</artifactId>
   <version>2.2.0.RELEASE</version>
</dependency>
```

```java
 @Component
public class KafkaProducer {
    @Autowired
    private KafkaTemplate<String,String> kafkaTemplate;
		public void send(){ 
      kafkaTemplate.send("test","msgKey","msgData");
		} 
}
 @Component
public class KafkaConsumer {
    @KafkaListener(topics = {"test"})
   public void listener(ConsumerRecord record){
	 } 
}
```

```ini
spring.kafka.bootstrap- servers=192.168.13.102:9092,192.168.13.103:9092,192.168.13.104:9092 
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer 
spring.kafka.producer.valueserializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.consumer.group-id=test-consumer-group
spring.kafka.consumer.auto-offset-reset=earliest 
spring.kafka.consumer.enable-auto-commit=true
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer 
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

## 消息存储：

kafka是使用日志文件的方式来保存生产者和发送者的消息，每条消息都有一 个offset值来表示它在分区中的偏移量。Kafka中存储的一般都是海量的消息数据，为了避免日志文件过 大，Log并不是直接对应在一个磁盘上的日志文件，而是对应磁盘上的一个目录，这个目录的命名规则 是<topic_name>_<partition_id>

一个topic的多个partition在物理磁盘上的保存路径，路径保存在 /tmp/kafka-logs/topic_partition，包 含日志文件、索引文件和时间索引文件

kafka是通过分段的方式将Log分为多个LogSegment，LogSegment是一个逻辑上的概念，一个 LogSegment对应磁盘上的一个日志文件和一个索引文件，其中日志文件是用来记录消息的。索引文件 是用来保存消息的索引

假设kafka以partition为最小存储单位，那么我们可以想象当kafka producer不断发送消息，必然会引 起partition文件的无线扩张，这样对于消息文件的维护以及被消费的消息的清理带来非常大的挑战，所 以kafka 以segment为单位又把partition进行细分。每个partition相当于一个巨型文件被平均分配到多 个大小相等的segment数据文件中(每个segment文件中的消息不一定相等)，这种特性方便已经被消 费的消息的清理，提高磁盘的利用率。

1. 根据offset的值，查找segment段中的index索引文件。由于索引文件命名是以上一个文件的最后 一个offset进行命名的，所以，使用二分查找算法能够根据offset快速定位到指定的索引文件。
2. 找到索引文件后，根据offset进行定位，找到索引文件中的符合范围的索引。(kafka采用稀疏索 引的方式来提高查找性能)
3. 得到position以后，再到对应的log文件中，从position出开始查找offset对应的消息，将每条消息 的offset与目标offset进行比较，直到找到消息

比如说，我们要查找offset=2490这条消息，那么先找到00000000000000000000.index, 然后找到 [2487,49111]这个索引，再到log文件中，根据49111这个position开始查找，比较每条消息的offset是 否大于等于2490。最后查找到对应的消息以后返回

### 日志文件内容：

offset和position这两个前面已经讲过了、 createTime表示创建时间、keysize和valuesize表示key和 value的大小、 compresscodec表示压缩编码、payload:表示消息的具体内容



### 日志清除：

日志的分段存储，一方面能够减少单个文件内容的大小，另一方面，方便kafka进行日志 清理。日志的清理策略有两个

​	1. 根据消息的保留时间，当消息在kafka中保存的时间超过了指定的时间，就会触发清理过程

2. 根据topic存储的数据大小，当topic所占的日志文件大小大于一定的阀值，则可以开始删除最旧的消息。

   kafka会启动一个后台线程，定期检查是否存在可以删除的消息 通过log.retention.bytes和log.retention.hours这两个参数来设置，当其中任意一个达到要求，都会执行删除。 默认的保留时间是:7天

### 日志压缩：

消息的key和value的值之间的对应关系是不断变化的，就像 数据库中的数据会不断被修改一样，消费者只关心key对应的最新的value。因此，我们可以开启kafka 的日志压缩功能，服务端会在后台启动启动Cleaner线程池，定期将相同的key进行合并，只保留最新的 value值。

### 磁盘存储优化：

#### 1）顺序写

大部分企业仍然用的是机械结构的磁盘，如果把消息以随机的方式写入到磁盘，那么磁盘首先 要做的就是寻址，也就是定位到数据所在的物理地址，在磁盘上就要找到对应的柱面、磁头以及对应的 扇区;这个过程相对内存来说会消耗大量时间，为了规避随机读写带来的时间消耗，kafka采用顺序写 的方式存储数据。即使是这样，但是频繁的I/O操作仍然会造成磁盘的性能瓶颈。

#### 2）零拷贝

用于将数据从页缓存传输到socket;在Linux中，是通过sendfile系 统调用来完成的。Java提供了访问这个系统调用的方法:FileChannel.transferTo API

使用sendfile，只需要一次拷贝就行，允许操作系统将数据直接从页缓存发送到网络上。所以在这个优 化的路径中，只有最后一步将数据拷贝到网卡缓存中是需要的

#### 3）页缓存

页缓存是操作系统实现的一种主要的磁盘缓存，但凡设计到缓存的，基本都是为了提升i/o性能，所以页 缓存是用来**减少磁盘I/O操作的**。

第一，访问磁盘的速度要远低于访问内存的速度，若从处理器L1和L2高速缓存访问则速度更快。

 第二，数据一旦被访问，就很有可能短时间内再次访问。正是由于基于访问内存比磁盘快的多，所以磁盘的内存缓存将给系统存储性能带来质的飞越。

