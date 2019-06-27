# SpringBootJMS异步消息

> JMS即Java消息服务（Java Message Service）应用程序接口，是一个Java平台中关于面向消息中间件（MOM）的API，用于在两个应用程序之间，或分布式系统中发送消息，进行异步通信。Java消息服务是一个与具体平台无关的API，绝大多数MOM提供商都对JMS提供支持。

消息中间件利用高效可靠的消息传递机制进行平台无关的数据交流，并基于数据通信来进行分布式系统的集成。通过提供消息传递和消息排队模型，它可以在分布式环境下扩展进程间的通信。对于消息中间件，常见的角色大致也就有Producer（生产者）、Consumer（消费者）
　　常见的消息中间件产品:
　　（1）ActiveMQ
　　ActiveMQ 是Apache出品，最流行的，能力强劲的开源消息总线。ActiveMQ 是一个完全支持JMS1.1和J2EE 1.4规范的 JMS Provider实现。我们在本次课程中介绍 ActiveMQ的使用。
　　（2）RabbitMQ
　　AMQP协议的领导实现，支持多种场景。淘宝的MySQL集群内部有使用它进行通讯，OpenStack开源云平台的通信组件，最先在金融行业得到运用。
　　（3）ZeroMQ
　　史上最快的消息队列系统
　　（4）Kafka
　　Apache下的一个子项目 。特点：高吞吐，在一台普通的服务器上既可以达到10W/s的吞吐速率；完全的分布式系统。适合处理海量数据。

| --           | RabbitMQ                                | ActiveMQ                    | RocketMQ                                          | Kafka                                            | CMQ            |
| ------------ | --------------------------------------- | --------------------------- | ------------------------------------------------- | ------------------------------------------------ | -------------- |
| 社区/公司    | Mozilla Public License                  | Apache                      | 阿里巴巴                                          | Apache                                           | 腾讯           |
| 开发语言     | Erlang                                  | Java                        | Java                                              | Scala & Java                                     | 应该是C++      |
| 客户端语言   | 多语言                                  | 多语言                      | Java，C++                                         | Java为主，语言无关                               | 多语言         |
| 协议支持     | 多协议支持                              | 多协议支持                  | 自定义                                            | Tcp之上自定义                                    | 目前HTTP       |
| 消息批量操作 | 不支持                                  | 支持                        | 支持                                              | 支持                                             | 支持           |
| 消息推拉模式 | Pull，Push                              | Pull，Push                  | Pull，Push                                        | Pull                                             | Pull，Push     |
| 高可用       | Master/Slave，master提供服务，slave备份 | 基于ZooKeeper + LevelDB的MS | 支持多M，多MS，异步复制模式，多M多S模式，同步双写 | replica机制，leader宕机会重新选举                | leader重新选举 |
| 数据可靠性   | 可靠                                    | M/S                         | 可靠                                              | 可靠                                             | 可靠           |
| 单机吞吐量   | 万级                                    | 万级                        | 低十万级                                          | 高十万级                                         | 高万级         |
| 消息延迟     | 微秒级                                  | -                           | -                                                 | 毫秒级                                           | 毫秒级         |
| 持久化能力   | 内存+文件，堆积量影响生产速率           | 内存+文件+数据库            | 磁盘文件                                          | 磁盘文件                                         | 磁盘           |
| 是否有序     | 单Client有序                            | 可以有序                    | 有序                                              | 多Client有序                                     | 单Client有序   |
| 支持事务     | 支持                                    | 支持                        | 支持                                              | 不支持，但是可以通过Low Level API 保证仅消费一次 | 不支持         |
| 集群         | 支持                                    | 支持                        | 支持                                              | 支持                                             | 支持           |
| 负载均衡     | 支持                                    | 支持                        | 支持                                              | 支持                                             | 支持           |
| 管理界面     | 较好                                    | 一般                        | 命令行                                            | 官方只有命令行                                   | 较好           |
| 部署方式     | 独立                                    | 独立                        | 独立                                              | 独立                                             | -              |
| 适合场景     | 非海量高可靠                            | 非海量高可靠                | 海量大规模分布式系统                              | 日志等海量数据流                                 | -              |

## ActiveMQ

> Apache提供的开源消息系统，完全采用Java来实现消息形式： 
> 1、点对点（queue） 
> 2、一对多（topic） 

https://segmentfault.com/a/1190000014958916

消息的 destination 分为 queue 和 topic，而消费者称为 subscriber（订阅者）。queue 中的消息只会发送给一个订阅者，而 topic 的消息，会发送给每一个订阅者。在 broker 中，处理 queue 消息和 topic 消息的逻辑是不同的。queue 先存储消息，然后把消息分发给消费者，topic 收到消息的同时，就会分发。

Queue 中有 doMessageSend 和 iterate 方法，doMessageSend 负责接收生产者的消息，iterate 负责分发消息给消费者。Topic 中也有 doMessageSend 和 iterate 方法，doMessageSend 负责接收生产者的消息，并且分发给消费者。

queue 有持久和临时2种类型（topic相同）：
队列默认为持久队列，一旦创建，一直存在于broker中。而临时队列被创建后，在connection关闭后，broker就会删除它。

topic 订阅有持久和非持久2种类型：
broker 会把消息全部推送给持久订阅，即便该订阅者中途offline了，如果是非持久订阅，一旦它下线，broker 不会为它保留消息，直到它上线后，开始继续发送消息。

默认安装完毕：

http://127.0.0.1:8161/

管理：http://127.0.0.1:8161/admin/  默认密码admin,admin

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
        </dependency>
```

基本配置

```ini
#mq
spring.activemq.broker-url=tcp://127.0.0.1:61616
spring.activemq.in-memory=true
spring.activemq.pool.enabled=false
spring.activemq.packages.trust-all=true
```

#### 生产者

```java
@Service
public class UserProducer {
    
    @Autowired
    private JmsMessagingTemplate jmsMessageTemplate;
    //点对点queue
    private static Destination destination = new ActiveMQQueue("user.queue");
    
    /**
     * 发送消息
     * @param message
     */
    public void sendMessage(final String message) {
        jmsMessageTemplate.convertAndSend(destination,message);
    }
}
```

#### 消费者

```java
@Component
public class UserConsumer {
    
    @JmsListener(destination = "user.queue")
    public void receiveQueue(String text){
        System.out.println("收到消息-->"+text);
    }
}
```

