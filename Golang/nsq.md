# NSQ

## 1）NSQ安装和基本使用

http://nsq.io/overview/quick_start.html

http://wiki.jikexueyuan.com/project/nsq-guide/

http://www.infoq.com/cn/news/2015/02/nsq-distributed-message-platform/

https://www.cnblogs.com/zhangboyu/p/7452759.html

https://segmentfault.com/a/1190000009194607

https://godoc.org/github.com/nsqio/go-nsq

brew install nsq

- 具有分布式且无单点故障的拓扑结构 支持水平扩展，在无中断情况下能够无缝地添加集群节点
- 低延迟的消息推送，参见官方提供的[性能说明文档](http://nsq.io/overview/performance.html)
- 具有组合式的负载均衡和多播形式的消息路由
- 既擅长处理面向流（高吞吐量）的工作负载，也擅长处理面向Job的（低吞吐量）工作负载
- 消息数据既可以存储于内存中，也可以存储在磁盘中
- 实现了生产者、消费者自动发现和消费者自动连接生产者，参见nsqlookupd
- 支持安全传输层协议（TLS），从而确保了消息传递的安全性
- 具有与数据格式无关的消息结构，支持JSON、Protocol Buffers、MsgPacek等消息格式
- 非常易于部署（几乎没有依赖）和配置（所有参数都可以通过命令行进行配置）
- 使用了简单的TCP协议且具有多种语言的客户端功能库
- 具有用于信息统计、管理员操作和实现生产者等的HTTP接口
- 为实时检测集成了统计数据收集器[StatsD](https://github.com/etsy/statsd/)
- 具有强大的集群管理界面，参见nsqadmin



- 支持消息内存队列的大小设置，默认完全持久化（值为0），消息即可持久到磁盘也可以保存在内存中
- 保证消息至少传递一次,以确保消息可以最终成功发送
- 收到的消息是无序的, 实现了松散订购
- 发现服务nsqlookupd具有最终一致性,消息最终能够找到所有Topic生产者

![NSQ](./images/nsq.gif)

从上图可以看出，单个nsqd可以有多个Topic，**每个Topic又可以有多个Channel**。Channel能够接收Topic所有消息的副本，从而实现了消息多播分发；而Channel上的每个消息被分发给它的订阅者，从而实现负载均衡，所有这些就组成了一个可以表示各种简单和复杂拓扑结构的强大框架。

### nsqlookupd:

管理拓扑信息并提供最终一致性的发现服务的守护进程

默认：tcp端口4160，http端口4161

```shell
-http-address="0.0.0.0:4161": <addr>:<port> 监听 HTTP 客户端
-inactive-producer-timeout=5m0s: 从上次 ping 之后，生产者驻留在活跃列表中的时长
-tcp-address="0.0.0.0:4160": TCP 客户端监听的 <addr>:<port> 
-broadcast-address: 这个 lookupd 节点的外部地址, (默认是 OS 主机名)
-tombstone-lifetime=45s: 生产者保持 tombstoned  的时长
-verbose=false: 允许输出日志
-version=false: 打印版本信息
```



### nsqd:

一个负责接收、排队、转发消息到客户端的守护进程

默认端口：tcp：4150 http：4151

#### 命令选项值

```shell
-config="": 配置文件路径
-data-path="": 缓存消息的磁盘路径
-deflate=true: 运行协商压缩特性（客户端压缩）
-http-address="0.0.0.0:4151": 为 HTTP 客户端监听 <addr>:<port>
-https-address="": 为 HTTPS 客户端 监听 <addr>:<port>
-lookupd-tcp-address=: 解析 TCP 地址名字 (可能会给多次)
-tcp-address="0.0.0.0:4150": TCP 客户端 监听的 <addr>:<port>
-verbose=false: 打开日志
-version=false: 打印版本
...
```



### nsqadmin:

一套Web用户界面，可实时查看集群的统计数据和执行各种各样的管理任务

默认端口：http:4171

这个时候可以访问：http://127.0.0.1:4171/ 即可看到管理后台

```shell
-graphite-url="": URL to graphite HTTP 地址
-http-address="0.0.0.0:4171": <addr>:<port> HTTP clients 监听的地址和端口
-lookupd-http-address=[]: lookupd HTTP 地址 (可能会提供多次)
-notification-http-endpoint="": HTTP 端点 (完全限定) ，管理动作将会发送到
-nsqd-http-address=[]: nsqd HTTP 地址 (可能会提供多次)
-proxy-graphite=false: Proxy HTTP requests to graphite
-template-dir="": 临时目录路径
-use-statsd-prefixes=true: expect statsd prefixed keys in graphite (ie: 'stats_counts.')
-version=false: 打印版本信息
```



#### 界面关键词解析：

Topic：主题名称 	empty Queue：清空队列	Delete Topic ： 删除主题	Pause Topic ： 暂停主题Memory+Disk ： 内存和磁盘	Messages : 表示消息总数	channels : 消息通道	In-Flight :飞行中，即将消费的消息

Deferred : 延迟消息	Requeued : 已请求的消息	Time Out : 超时	Connections : 连接数

### 集群部署：

```shell
➜  ~ nsqlookupd
[nsqlookupd] 2017/12/21 14:44:59.812652 nsqlookupd v1.0.0-compat (built w/go1.9.1)
[nsqlookupd] 2017/12/21 14:44:59.813130 TCP: listening on [::]:4160
[nsqlookupd] 2017/12/21 14:44:59.813212 HTTP: listening on [::]:4161
➜  ~ mkdir -p test/nsq/nsq1
➜  ~ mkdir -p test/nsq/nsq2

➜  nsq1 nsqd --lookupd-tcp-address=127.0.0.1:4160 --data-path=/Users/borgxiao/test/nsq/nsq1
[nsqd] 2017/12/21 14:50:24.047450 nsqd v1.0.0-compat (built w/go1.9.1)
[nsqd] 2017/12/21 14:50:24.047598 ID: 779
[nsqd] 2017/12/21 14:50:24.047660 NSQ: persisting topic/channel metadata to /Users/borgxiao/test/nsq/nsq1/nsqd.dat
[nsqd] 2017/12/21 14:50:24.050584 HTTP: listening on [::]:4151
[nsqd] 2017/12/21 14:50:24.051099 TCP: listening on [::]:4150
➜  nsq2 nsqd --lookupd-tcp-address=127.0.0.1:4160 --data-path=/Users/borgxiao/test/nsq/nsq2 -http-address="0.0.0.0:4152" -tcp-address="0.0.0.0:4153"
[nsqd] 2017/12/21 14:52:33.483509 nsqd v1.0.0-compat (built w/go1.9.1)
[nsqd] 2017/12/21 14:52:33.483648 ID: 779
[nsqd] 2017/12/21 14:52:33.483854 NSQ: persisting topic/channel metadata to /Users/borgxiao/test/nsq/nsq2/nsqd.dat
[nsqd] 2017/12/21 14:52:33.486354 TCP: listening on [::]:4153
[nsqd] 2017/12/21 14:52:33.486547 HTTP: listening on [::]:4152

➜  ~ nsqadmin --lookupd-http-address=127.0.0.1:4161
[nsqadmin] 2017/12/21 14:57:48.365079 nsqadmin v1.0.0-compat (built w/go1.9.1)
[nsqadmin] 2017/12/21 14:57:48.366252 HTTP: listening on [::]:4171
```

### 常用命令：

```shell
##发送消息
➜  ~ curl -d "msg1" http://127.0.0.1:4151/pub\?topic\=test1
OK%
##批量发送消息
➜  ~ curl -d "msg2\msg3\msg4" http://127.0.0.1:4151/mpub\?topic\=test1
OK%

##创建topic
➜  ~ curl -d "" http://127.0.0.1:4151/topic/create\?topic\=test3
##删除topic
➜  ~ curl  -d "" http://127.0.0.1:4151/topic/delete\?topic\=test3

➜  ~ curl  -d "" http://127.0.0.1:4151/topic/delete\?topic\=test3
{"message":"TOPIC_NOT_FOUND"}%

##创建channel
➜  ~ curl  -d "" http://127.0.0.1:4151/channel/create\?topic\=test2\&channel\=ch3
##删除channel
➜  ~ curl  -d "" http://127.0.0.1:4151/channel/delete\?topic\=test2\&channel\=ch3

/topic/create - 创建一个新的话题（topic)
/topic/delete - 删除一个话题（topic)
/topic/empty - 清空话题（topic)
/topic/pause - 暂停话题（topic)的消息流
/topic/unpause - 恢复话题（topic)的消息流
/channel/create - 创建一个新的通道（channel)
/channel/delete - 删除一个通道（channel)
/channel/empty - 清空一个通道（channel)
/channel/pause - 暂停通道（channel)的消息流
/channel/unpause - 恢复通道（channel)的消息流

##查看信息
➜  ~ curl   http://127.0.0.1:4151/stats\?format\=json
{"version":"1.0.0-compat","health":"OK","start_time":1513839024,"topics":[{"topic_name":"test1","channels":[{"channel_name":"go1","depth":3,"backend_depth":0,"in_flight_count":0,"deferred_count":0,"message_count":3,"requeue_count":0,"timeout_count":0,"clients":[],"paused":false,"e2e_processing_latency":{"count":0,"percentiles":null}}],"depth":0,"backend_depth":0,"message_count":3,"paused":false,"e2e_processing_latency":{"count":0,"percentiles":null}},{"topic_name":"test2","channels":[],"depth":0,"backend_depth":0,"message_count":0,"paused":false,"e2e_processing_latency":{"count":0,"percentiles":null}}]}%

##ping
➜  ~ curl http://127.0.0.1:4151/ping
OK%
##获取版本信息
➜  ~ curl http://127.0.0.1:4151/info
{"version":"1.0.0-compat","broadcast_address":"BorgXiaode.local","hostname":"BorgXiaode.local","http_port":4151,"tcp_port":4150,"start_time":1513839024}%
```

#### nsqlookupd命令:

```shell
##获取对应topic的生产者列表
➜  ~ curl http://127.0.0.1:4161/lookup\?topic\=test1
{"channels":["go1"],"producers":[{"remote_address":"127.0.0.1:58279","hostname":"BorgXiaode.local","broadcast_address":"BorgXiaode.local","tcp_port":4150,"http_port":4151,"version":"1.0.0-compat"}]}%

##获取所有topic列表
➜  ~ curl http://127.0.0.1:4161/topics
{"topics":["test3","test1","test2"]}
##获取对应topic的所有channel
➜  ~ curl http://127.0.0.1:4161/channels\?topic\=test1
{"channels":["go1"]}%

##获取所有节点
➜  ~ curl http://127.0.0.1:4161/nodes
{"producers":[{"remote_address":"127.0.0.1:58797","hostname":"BorgXiaode.local","broadcast_address":"BorgXiaode.local","tcp_port":4153,"http_port":4152,"version":"1.0.0-compat","tombstones":[],"topics":[]},{"remote_address":"127.0.0.1:58800","hostname":"BorgXiaode.local","broadcast_address":"BorgXiaode.local","tcp_port":4150,"http_port":4151,"version":"1.0.0-compat","tombstones":[false,false],"topics":["test1","test2"]}]}%
```

注意：由于开始是在node1节点创建的节点，所有获取topic的生产者的时候只返回了一个。后面再node2也创建了同样的topic后正常。尝试通过admin去创建，发现创建成功，但是没有生产者，疑惑不解，正常不是一个通过lookupd来创建，然后同步到各个节点吗？

**Nsqd** ——nsqd守护进程是NSQ的核心部分，它是一个单独的监听某个端口进来的消息的二进制程序。每个nsqd节点**都独立运行，不共享任何状态**。当一个节点启动时，它向一组nsqlookupd节点进行注册操作，并将保存在此节点上的topic和channel进行广播。

客户端可以发布消息到nsqd守护进程上，或者从nsqd守护进程上读取消息。通常，**消息发布者会向一个单一的local nsqd发布消息**，消费者从连接了的一组nsqd节点的topic上远程读取消息。如果你不关心动态添加节点功能，你可以直接运行standalone模式。

**Nsqlookupd** ——nsqlookupd服务器像consul或etcd那样工作，只是它被设计得没有协调和强一致性能力。每个nsqlookupd都作为nsqd节点注册信息的短暂数据存储区。**消费者连接这些节点去检测需要从哪个nsqd节点上读取消息**。

**Channels** ——channel组与消费者相关，是消费者之间的负载均衡，channel在某种意义上来说是一个“队列”。每当一个发布者发送一条消息到一个topic，消息会被复制到所有消费者连接的channel上，消费者通过这个特殊的channel读取消息，实际上，在消费者第一次订阅时就会创建channel。

```shell
➜  ~ curl http://127.0.0.1:4161/lookup\?topic\=test1
{"channels":["go1"],"producers":[{"remote_address":"127.0.0.1:58800","hostname":"BorgXiaode.local","broadcast_address":"BorgXiaode.local","tcp_port":4150,"http_port":4151,"version":"1.0.0-compat"},{"remote_address":"127.0.0.1:58797","hostname":"BorgXiaode.local","broadcast_address":"BorgXiaode.local","tcp_port":4153,"http_port":4152,"version":"1.0.0-compat"}]}

```

## 2）go-nsq生产者：

```go
package main

import (
	"bufio"
	"fmt"
	"github.com/nsqio/go-nsq"
	"os"
)

var producer *nsq.Producer

func main() {
	strIP1 := "127.0.0.1:4150"
	strIP2 := "127.0.0.1:4153"
	InitProducer(strIP1)

	running := true

	reader := bufio.NewReader(os.Stdin)

	for running {
		data, _, _ := reader.ReadLine()
		command := string(data)
		if command == "stop" {
			running = false
		}

		for err := Publish("test1", command); err != nil; err = Publish("test1", command) {
			strIP1, strIP2 = strIP2, strIP1
			InitProducer(strIP1)
		}
		//关闭
		producer.Stop()
	}
}

func InitProducer(str string) {
	var err error
	fmt.Println("Address:", str)
	//初始化生产者
	producer, err = nsq.NewProducer(str, nsq.NewConfig())
	if err != nil {
		panic(err)
	}
}

func Publish(topic string, message string) error {
	var err error
	if producer != nil {
		if message == "" {
			return nil
		}
		err = producer.Publish(topic, []byte(message))
		return err
	}

	return fmt.Errorf("producer is nil", err)
}

```

### Config：

| 配置                 | 解释     | 默认值                                |
| ------------------ | ------ | ---------------------------------- |
| dial_timeout       | 连接超时时间 | 1s                                 |
| read_timeout       | 读取超时时间 | min:"100ms" max:"5m" default:"60s" |
| write_timeout      | 写超时时间  | min:"100ms" max:"5m" default:"1s"  |
| heartbeat_interval | 心跳时间   | 30s                                |



## 3）go-nsq消费者：

```go
package main

import (
	"fmt"
	"github.com/nsqio/go-nsq"
	"time"
)

//消费者
type ConsumerT struct{}

func main() {
	var ch = make(chan int, 0)
	InitConsumer("test1", "go1", "127.0.0.1:4161")
	/*
		for {
			fmt.Println("sleep:", time.Now())
			time.Sleep(time.Second * 10)
		}
	*/
	<-ch
}

func InitConsumer(topic string, channel string, address string) {
	cfg := nsq.NewConfig()
	cfg.LookupdPollInterval = time.Second
	c, err := nsq.NewConsumer(topic, channel, cfg)

	if err != nil {
		panic(err)
	}
	//c.SetLogger(nil, 0)
	c.AddHandler(&ConsumerT{})
	if err := c.ConnectToNSQLookupd(address); err != nil {
		panic(err)
	}
}

func (*ConsumerT) HandleMessage(msg *nsq.Message) error {
	fmt.Println("Recevie", msg.NSQDAddress, "Message:", string(msg.Body))
	return nil
}

```

## 4）源码解读：

### protocol.go

```go
//名字验证
var validTopicChannelNameRegex = regexp.MustCompile(`^[\.a-zA-Z0-9_-]+(#ephemeral)?$`)
func IsValidChannelName(name string) bool
func IsValidTopicName(name string) bool

func ReadResponse(r io.Reader) ([]byte, error)
func ReadUnpackedResponse(r io.Reader) (int32, []byte, error)
func UnpackResponse(response []byte) (int32, []byte, error)
```

名称为协议，核心功能就是解包

### command.go

```go
type Command struct {
    Name   []byte
    Params [][]byte
    Body   []byte
}
func Auth(secret string) (*Command, error)
func DeferredPublish(topic string, delay time.Duration, body []byte) *Command
func Finish(id MessageID) *Command
func Ping() *Command
func Nop() *Command //心跳包

func Publish(topic string, body []byte) *Command
func MultiPublish(topic string, bodies [][]byte) (*Command, error)

func Subscribe(topic string, channel string) *Command 

```

这里面封装了和实现了所有nsq命令。

参照：http://wiki.jikexueyuan.com/project/nsq-guide/tcp_protocol_spec.html

### config.go

```go
type configHandler interface {
	HandlesOption(c *Config, option string) bool
	Set(c *Config, option string, value interface{}) error
	Validate(c *Config) error
}
func NewConfig() *Config {
	c := &Config{
		configHandlers: []configHandler{&structTagsConfig{}, &tlsConfig{}},
		initialized:    true,
	}
	if err := c.setDefaults(); err != nil {
		panic(err.Error())
	}
	return c
}
//定义了一个Config类型，包含了所有配置

func (c *Config) Set(option string, value interface{}) error
func (c *Config) Validate() error

//核心反射应用
func (h *structTagsConfig) SetDefaults(c *Config) error {
	val := reflect.ValueOf(c).Elem()
	typ := val.Type()
	for i := 0; i < typ.NumField(); i++ {
		field := typ.Field(i)
		opt := field.Tag.Get("opt")
		defaultVal := field.Tag.Get("default")
		if defaultVal == "" || opt == "" {
			continue
		}

		if err := c.Set(opt, defaultVal); err != nil {
			return err
		}
	}

	hostname, err := os.Hostname()
	if err != nil {
		log.Fatalf("ERROR: unable to get hostname %s", err.Error())
	}

	c.ClientID = strings.Split(hostname, ".")[0]
	c.Hostname = hostname
	c.UserAgent = fmt.Sprintf("go-nsq/%s", VERSION)
	return nil
}
```

```go
package main

import (
	"fmt"
	"reflect"
	"time"
)

type Config struct {
	DialTimeout time.Duration `opt:"dial_timeout" default:"1s"`
}

func main() {
	var c = &Config{}
	//初始化值
	val := reflect.ValueOf(c).Elem()
	fmt.Println(val) //{0s}
	//类型
	typ := val.Type()
	fmt.Println(typ) //main.Config
	//获取每个元素
	for i := 0; i < typ.NumField(); i++ {
		field := typ.Field(i)
		fmt.Println(field) //{DialTimeout  time.Duration opt:"dial_timeout" default:"1s" 0 [0] false}
		opt := field.Tag.Get("opt")
		fmt.Println(opt) //dial_timeout
		defaultVal := field.Tag.Get("default")
		fmt.Println(defaultVal) //1s
		fmt.Println(field.Name) //DialTimeout
	}

}

```

反射的用处，发射的TAG。newConfig的时候即初始化了默认配置

### conn.go

非常核心的一个类，类似php的mysql的 PDO

```go
type Conn struct {
    // contains filtered or unexported fields
}

func NewConn(addr string, config *Config, delegate ConnDelegate) *Conn {
	if !config.initialized {
		panic("Config must be created with NewConfig()")
	}
	return &Conn{
		addr: addr,

		config:   config,
		delegate: delegate,

		maxRdyCount:      2500,
		lastMsgTimestamp: time.Now().UnixNano(),

		cmdChan:         make(chan *Command),
		msgResponseChan: make(chan *msgResponse),
		exitChan:        make(chan int),
		drainReady:      make(chan int),
	}
}

// Read performs a deadlined read on the underlying TCP connection
func (c *Conn) Read(p []byte) (int, error) {
	c.conn.SetReadDeadline(time.Now().Add(c.config.ReadTimeout))
	return c.r.Read(p)
}

// Write performs a deadlined write on the underlying TCP connection
func (c *Conn) Write(p []byte) (int, error) {
	c.conn.SetWriteDeadline(time.Now().Add(c.config.WriteTimeout))
	return c.w.Write(p)
}
```

### consumer.go

消费者核心类

```go
//创建一个消费者
func NewConsumer(topic string, channel string, config *Config) (*Consumer, error)
//添加处理方法
func (r *Consumer) AddHandler(handler Handler)
//直连接一个nsqd
func (r *Consumer) ConnectToNSQD(addr string) error

//订阅
cmd := Subscribe(r.topic, r.channel)
	err = conn.WriteCommand(cmd)

//直连接多个nsqd
func (r *Consumer) ConnectToNSQDs(addresses []string) error
//连接nsqlookupd获取
func (r *Consumer) ConnectToNSQLookupd(addr string) error
	// 第一个地址将去轮询获取
	if numLookupd == 1 {
		r.queryLookupd()
		r.wg.Add(1)
		go r.lookupdLoop()
	}

// 轮询定时查询lookupd
func (r *Consumer) lookupdLoop() {
	// add some jitter so that multiple consumers discovering the same topic,
	// when restarted at the same time, dont all connect at once.
	r.rngMtx.Lock()
	jitter := time.Duration(int64(r.rng.Float64() *
		r.config.LookupdPollJitter * float64(r.config.LookupdPollInterval)))
	r.rngMtx.Unlock()
	var ticker *time.Ticker

	select {
	case <-time.After(jitter):
	case <-r.exitChan:
		goto exit
	}
	//定时的设置时间
	ticker = time.NewTicker(r.config.LookupdPollInterval)

	for {
		select {
		case <-ticker.C:
			r.queryLookupd()
		case <-r.lookupdRecheckChan:
			r.queryLookupd()
		case <-r.exitChan://如果收到停止
			goto exit
		}
	}

exit:
	if ticker != nil {
		ticker.Stop()
	}
	r.log(LogLevelInfo, "exiting lookupdLoop")
	r.wg.Done()
}
//连接多个nsqlookupd获取 循环调取的是ConnectToNSQLookupd
func (r *Consumer) ConnectToNSQLookupds(addresses []string) error
//轮询查询地址算法
if r.lookupdQueryIndex >= len(r.lookupdHTTPAddrs) {
		r.lookupdQueryIndex = 0
	}
	addr := r.lookupdHTTPAddrs[r.lookupdQueryIndex]
	num := len(r.lookupdHTTPAddrs)
	r.mtx.RUnlock()
	//一个个来
	r.lookupdQueryIndex = (r.lookupdQueryIndex + 1) % num

//发现服务最终调用的还是ConnectToNSQD
	if discoveryFilter, ok := r.behaviorDelegate.(DiscoveryFilter); ok {
		nsqdAddrs = discoveryFilter.Filter(nsqdAddrs)
	}
	for _, addr := range nsqdAddrs {
		err = r.ConnectToNSQD(addr)
		if err != nil && err != ErrAlreadyConnected {
			r.log(LogLevelError, "(%s) error connecting to nsqd - %s", addr, err)
			continue
		}
	}

//消费handler 定时接收incomingMessages(传入的消息)
func (r *Consumer) handlerLoop(handler Handler) {
	r.log(LogLevelDebug, "starting Handler")

	for {
		message, ok := <-r.incomingMessages
		if !ok {
			goto exit
		}

		if r.shouldFailMessage(message, handler) {
			message.Finish()
			continue
		}
      	//处理消息
		err := handler.HandleMessage(message)
		if err != nil {
			r.log(LogLevelError, "Handler returned error (%s) for msg %s", err, message.ID)
			if !message.IsAutoResponseDisabled() {
				message.Requeue(-1)
			}
			continue
		}
      	//消息处理结束
		if !message.IsAutoResponseDisabled() {
			message.Finish()
		}
	}

exit:
	r.log(LogLevelDebug, "stopping Handler")
	if atomic.AddInt32(&r.runningHandlers, -1) == 0 {
		r.exit()
	}
}

```

### message.go

消息处理类，即对nsq消息的封装

```go
type Message struct {
	ID        MessageID
	Body      []byte
	Timestamp int64
	Attempts  uint16

	NSQDAddress string

	Delegate MessageDelegate

   //是否关闭自动回复，即告诉程序我已经收到且消费 默认是0
	autoResponseDisabled int32
	responded            int32
}
//解码nsq消息
func DecodeMessage(b []byte) (*Message, error) {
	var msg Message

	if len(b) < 10+MsgIDLength {
		return nil, errors.New("not enough data to decode valid message")
	}

	msg.Timestamp = int64(binary.BigEndian.Uint64(b[:8]))
	msg.Attempts = binary.BigEndian.Uint16(b[8:10])
	copy(msg.ID[:], b[10:10+MsgIDLength])
	msg.Body = b[10+MsgIDLength:]

	return &msg, nil
}

//重新将消息队列（表示处理失败）这里没有成功后响应
//这个消息放在队尾，表示已经发布过，但是因为很多实现细节问题，不要严格信赖这个，将来会改进。
//简单来说，消息在传播途中，并且超时就表示 REQ。对应REQ命令
func (m *Message) Requeue(delay time.Duration)

//完成一个消息 (表示成功处理) 这里没有成功后响应  对应FIN命令
func (m *Message) Finish()

//重置传播途中的消息超时时间 即重新激活 对应TOUCH命令
func (m *Message) Touch()
```

### delegates.go

代理行为类，类似事件注册，但是没有开发对应的注册方法，有点遗憾

```go
//消息处理行为接口
type MessageDelegate interface {
	// OnFinish is called when the Finish() method
	// is triggered on the Message
	OnFinish(*Message)

	// OnRequeue is called when the Requeue() method
	// is triggered on the Message
	OnRequeue(m *Message, delay time.Duration, backoff bool)

	// OnTouch is called when the Touch() method
	// is triggered on the Message
	OnTouch(*Message)
}

//消费者处理行为接口
type consumerConnDelegate struct {
	r *Consumer
}

func (d *consumerConnDelegate) OnResponse(c *Conn, data []byte)       { d.r.onConnResponse(c, data) }
func (d *consumerConnDelegate) OnError(c *Conn, data []byte)          { d.r.onConnError(c, data) }
func (d *consumerConnDelegate) OnMessage(c *Conn, m *Message)         { d.r.onConnMessage(c, m) }
func (d *consumerConnDelegate) OnMessageFinished(c *Conn, m *Message) { d.r.onConnMessageFinished(c, m) }
func (d *consumerConnDelegate) OnMessageRequeued(c *Conn, m *Message) { d.r.onConnMessageRequeued(c, m) }
func (d *consumerConnDelegate) OnBackoff(c *Conn)                     { d.r.onConnBackoff(c) }
func (d *consumerConnDelegate) OnContinue(c *Conn)                    { d.r.onConnContinue(c) }
func (d *consumerConnDelegate) OnResume(c *Conn)                      { d.r.onConnResume(c) }
func (d *consumerConnDelegate) OnIOError(c *Conn, err error)          { d.r.onConnIOError(c, err) }
func (d *consumerConnDelegate) OnHeartbeat(c *Conn)                   { d.r.onConnHeartbeat(c) }
func (d *consumerConnDelegate) OnClose(c *Conn)                       { d.r.onConnClose(c) }


//生成者处理行为接口

type producerConnDelegate struct {
	w *Producer
}

func (d *producerConnDelegate) OnResponse(c *Conn, data []byte)       { d.w.onConnResponse(c, data) }
func (d *producerConnDelegate) OnError(c *Conn, data []byte)          { d.w.onConnError(c, data) }
func (d *producerConnDelegate) OnMessage(c *Conn, m *Message)         {}
func (d *producerConnDelegate) OnMessageFinished(c *Conn, m *Message) {}
func (d *producerConnDelegate) OnMessageRequeued(c *Conn, m *Message) {}
func (d *producerConnDelegate) OnBackoff(c *Conn)                     {}
func (d *producerConnDelegate) OnContinue(c *Conn)                    {}
func (d *producerConnDelegate) OnResume(c *Conn)                      {}
func (d *producerConnDelegate) OnIOError(c *Conn, err error)          { d.w.onConnIOError(c, err) }
func (d *producerConnDelegate) OnHeartbeat(c *Conn)                   { d.w.onConnHeartbeat(c) }
func (d *producerConnDelegate) OnClose(c *Conn)                       { d.w.onConnClose(c) }

```

### producer.go

生产者类

```go
type Producer struct {
	id     int64
	addr   string
	conn   producerConn
	config Config

	logger   logger
	logLvl   LogLevel
	logGuard sync.RWMutex

	responseChan chan []byte
	errorChan    chan []byte
	closeChan    chan int

	transactionChan chan *ProducerTransaction
	transactions    []*ProducerTransaction
	state           int32

	concurrentProducers int32
	stopFlag            int32
	exitChan            chan int
	wg                  sync.WaitGroup
	guard               sync.Mutex
}
//创建一个生产者
func NewProducer(addr string, config *Config) (*Producer, error) {
	config.assertInitialized()
	err := config.Validate()
	if err != nil {
		return nil, err
	}

	p := &Producer{
		id: atomic.AddInt64(&instCount, 1),

		addr:   addr,
		config: *config,

		logger: log.New(os.Stderr, "", log.Flags()),
		logLvl: LogLevelInfo,

		transactionChan: make(chan *ProducerTransaction),
		exitChan:        make(chan int),
		responseChan:    make(chan []byte),
		errorChan:       make(chan []byte),
	}
	return p, nil
}

func (w *Producer) Ping() error
func (w *Producer) SetLogger(l logger, lvl LogLevel) 
//停止生产者，优雅的
func (w *Producer) Stop() 

//异步发送消息 -->Publish
func (w *Producer) PublishAsync(topic string, body []byte, doneChan chan *ProducerTransaction,
	args ...interface{}) error
//异步多条信息 -->MultiPublish
func (w *Producer) MultiPublishAsync(topic string, body [][]byte, doneChan chan *ProducerTransaction,
	args ...interface{}) error
//发布消息
func (w *Producer) Publish(topic string, body []byte) error
func (w *Producer) MultiPublish(topic string, body [][]byte) error
//延迟发布消息
func (w *Producer) DeferredPublish(topic string, delay time.Duration, body []byte) error

func (w *Producer) sendCommand(cmd *Command) error {
	doneChan := make(chan *ProducerTransaction)
	err := w.sendCommandAsync(cmd, doneChan, nil)
	if err != nil {
		close(doneChan)
		return err
	}
  	//等待读取返回
	t := <-doneChan
	return t.Error
}

func (w *Producer) sendCommandAsync(cmd *Command, doneChan chan *ProducerTransaction,
	args []interface{}) error {
	// keep track of how many outstanding producers we're dealing with
	// in order to later ensure that we clean them all up...
	atomic.AddInt32(&w.concurrentProducers, 1)
	defer atomic.AddInt32(&w.concurrentProducers, -1)

	if atomic.LoadInt32(&w.state) != StateConnected {
		err := w.connect()
		if err != nil {
			return err
		}
	}

   //把命令传入通道
	t := &ProducerTransaction{
		cmd:      cmd,
		doneChan: doneChan,
		Args:     args,
	}

	select {
	case w.transactionChan <- t:
	case <-w.exitChan:
		return ErrStopped
	}

	return nil
}
//连接的时候已经开了个线程等待执行命令如下命令
func (w *Producer) router() {
	for {
		select {
		case t := <-w.transactionChan://收到事物命令执行
			w.transactions = append(w.transactions, t)
			err := w.conn.WriteCommand(t.cmd)
			if err != nil {
				w.log(LogLevelError, "(%s) sending command - %s", w.conn.String(), err)
				w.close()
			}
		case data := <-w.responseChan://收到执行结果
			w.popTransaction(FrameTypeResponse, data)
		case data := <-w.errorChan://错误
			w.popTransaction(FrameTypeError, data)
		case <-w.closeChan://关闭
			goto exit
		case <-w.exitChan://停止
			goto exit
		}
	}

exit:
	w.transactionCleanup()
	w.wg.Done()
	w.log(LogLevelInfo, "exiting router")
}
```



