# Thrift RPC

参考：

http://blog.csdn.net/liuxinmingcode/article/details/45696237

http://thrift.apache.org/tutorial/go

## 1）理论知识：

### 4层架构：

> Thrift实际上是实现了C/S模式，通过代码生成工具将接口定义文件生成服务器端和客户端代码（可以为不同语言），从而实现服务端和客户端跨语言的支持。用户在Thirft描述文件中声明自己的服务，这些服务经过编译后会生成相应语言的代码文件，然后用户实现服务（客户端调用服务，服务器端提服务）便可以了。其中protocol（协议层, 定义数据传输格式，可以为二进制或者XML等）和transport（传输层，定义数据传输方式，可以为TCP/IP传输，内存共享或者文件共享等）被用作运行时库。

http://thrift.apache.org/docs/concepts

#### Server：

服务器进程调度

| 参数                 | 描述                                       |
| ------------------ | ---------------------------------------- |
| TSimpleServer      | 简单的单线程服务模型，常用于测试                         |
| TThreadPoolServer  | 多线程服务模型，使用标准的阻塞式IO。                      |
| TNonblockingServer | 多线程服务模型，使用非阻塞式IO（需使用TFramedTransport数据传输方式） |

#### Processor：

RPC接口处理函数分发，IDL定义接口的实现将挂接在这

#### Protocol：

协议定义

| 参数                  | 描述                            |
| ------------------- | ----------------------------- |
| TBinaryProtocol     | 二进制                           |
| TCompactProtocol    | 压缩格式                          |
| TJSONProtocol       | JSON格式                        |
| TSimpleJSONProtocol | 提供JSON只写协议, 生成的文件很容易通过脚本语言解析。 |
| TDebugProtocol      | 使用易懂的可读的文本格式，以便于debug         |



#### Transport：

网络传输

| 参数               | 描述                                       |
| ---------------- | ---------------------------------------- |
| TSocket          | 阻塞式socker                                |
| TFramedTransport | 以frame为单位进行传输，非阻塞式服务中使用。                 |
| TFileTransport   | 以文件形式进行传输。                               |
| TMemoryTransport | 将内存用于I/O. java实现时内部实际使用了简单的ByteArrayOutputStream。 |
| TZlibTransport   | 使用zlib进行压缩， 与其他传输方式联合使用。当前无java实现。       |

### Thrift定义

http://thrift.apache.org/docs/types

- bool: **布尔值 (true or false), one byte**
- byte: **有符号字节**
- i16: **16位有符号整型**
- i32: **32位有符号整型**
- i64: **64位有符号整型**
- double: **64位浮点型**
- string: **Encoding agnostic text or binary string** 
- binary: **Blob (byte array) a sequence of unencoded bytes**  主要用在java

> 基本类型中基本都是有符号数，因为有些语言没有无符号数，所以Thrift不支持无符号整型。

#### struct结构体

不能继承，可以嵌套，不能嵌套自己

- 成员都是明确的类型
- 成员都是要通过整数编码，编号不能重复，传输中使用
- 成员分隔符为逗号，或者是分号。建议统一一种
- optional(非必填，需要序列化)和required(必填，且必须序列化) 不指定则可以不填充

#### 容器Containers

**list(t)**：元素类型为t的有序表，准许元素重复 php array

**set(t)**：元素类型为t的无序表，不容许元素重复 php array

**map(t,v)**：键类型t，值类型v的map。php array go map

```idl
//各种语言命名空间定义
namespace go batu.demo
namespace php batu.demo

//结构体定义
struc Article{
  1:i32 id,
  2:string title,
  3,string content,
  4,string author,
}
//常量
const map<string,string> MAPCONSTANT = {'hello':'world', 'goodnight':'moon'}

//定义server
service batuThrift {
  list<string> CallBack(1:i64 callTime,2:string name,3:map<string,string> paramMap),
  void put(1:Article newArticle),
}
```

```idl

include "shared.thrift"

namespace cpp tutorial
namespace d tutorial
namespace dart tutorial
namespace java tutorial
namespace php tutorial
namespace perl tutorial
namespace haxe tutorial

typedef i32 MyInteger

const i32 INT32CONSTANT = 9853
const map<string,string> MAPCONSTANT = {'hello':'world', 'goodnight':'moon'}

enum Operation {
  ADD = 1,
  SUBTRACT = 2,
  MULTIPLY = 3,
  DIVIDE = 4
}

struct Work {
  1: i32 num1 = 0,
  2: i32 num2,
  3: Operation op,
  4: optional string comment,
}

exception InvalidOperation {
  1: i32 whatOp,
  2: string why
}
service Calculator extends shared.SharedService {


   void ping(),

   i32 add(1:i32 num1, 2:i32 num2),

   i32 calculate(1:i32 logid, 2:Work w) throws (1:InvalidOperation ouch),

   oneway void zip()

}

namespace cpp shared
namespace d share // "shared" would collide with the eponymous D keyword.
namespace dart shared
namespace java shared
namespace perl shared
namespace php shared
namespace haxe shared

struct SharedStruct {
  1: i32 key
  2: string value
}

service SharedService {
  SharedStruct getStruct(1: i32 key)
}
```

#### 编译生成IDL文件

```ini
thrift -r --gen go batu.thrift
thrift -r --gen php batu.thrift  
thrift -r --gen php:server batu.thrift #生成PHP服务端接口代码有所不一样
```

## 2）实战

### 2.1）编写thrift文件：

src/crm.thrift

```idl
namespace go crm
namespace php crm

struct Crm {
    1:i32 crmid,
    2:string name,
    3:i32 age,
    4:string address,
}

exception CrmException {
    1:i32 code,
    2:string msg,
}

service CrmServer {
    void ping(),
    i32 add(1:string name,2:i32 age,3:string address) throws (1:CrmException e),
    list<Crm> getlist() throws (1:CrmException e),
    Crm getrowbyid(1:i32 crmid) throws (1:CrmException e),
}
```

### 2.2）生成gen：

```ini
➜  src thrift -r --gen go crm.thrift
➜  src thrift -r --gen php crm.thrift
```

### 2.3）编写Go服务端代码：

src/crm.go  

需要提前：go get git.apache.org/thrift.git/lib/go/thrift 下载会很慢

```go
package main

import (
	"context"
	"fmt"
	"gen-go/crm"
	"git.apache.org/thrift.git/lib/go/thrift"
	"os"
	//"strconv"
	"strconv"
)

const NetAddr = "127.0.0.1:9090"

func main() {
	//定义网络协议：单位块传输
	transportFactory := thrift.NewTFramedTransportFactory(thrift.NewTTransportFactory())
	//定义为传输协议：二进制
	protocolFactory := thrift.NewTBinaryProtocolFactoryDefault()

	//创建服务
	serverTransport, err := thrift.NewTServerSocket(NetAddr)
	if err != nil {
		fmt.Println("服务启动失败！", err)
		os.Exit(1)
	}
	//注册处理服务
	handler := &CrmServerHanlder{}

	processor := crm.NewCrmServerProcessor(handler)
	server := thrift.NewTSimpleServer4(processor, serverTransport, transportFactory, protocolFactory)
	fmt.Println("thrift server 已经启动：", NetAddr)
	server.Serve()
}

//定义handle 这里为了方便方在同一个文件里面

//这里用到type 别名
type crmperson = crm.Crm

//定义自定义错误

type Crmerror struct {
	crm.CrmException
}

func (err *Crmerror) Error() string {
	strFormat := "{'code':%d,'msg':\"%s\"}"
	return fmt.Sprintf(strFormat, err.Code, err.Msg)
}

var CRMS []*crmperson

type CrmServerHanlder struct {
	crm.CrmServer
}

//Ping命令
func (c *CrmServerHanlder) Ping(ctx context.Context) (err error) {
	fmt.Println("接收到客户端Ping命令")
	return
}

//添加crm
func (c *CrmServerHanlder) Add(ctx context.Context, name string, age int32, address string) (r int32, err error) {
	fmt.Println("客户端")
	//TODO 添加客户
	r1 := len(CRMS)
	r = int32(r1)

	crm := &crmperson{Crmid: r, Name: name, Age: age, Address: address}

	//添加进去
	CRMS = append(CRMS, crm)

	fmt.Printf("添加客户：%v\n", crm)
	return int32(r), err
}

func (c *CrmServerHanlder) Getlist(ctx context.Context) (r []*crmperson, err error) {
	fmt.Println("获取所有数据...")
	return CRMS, err
}

func (c *CrmServerHanlder) Getrowbyid(ctx context.Context, crmid int32) (r *crmperson, err error) {

	fmt.Printf("获取客户crmi:%d\n", crmid)

	l := int(crmid)
	length := len(CRMS)

	//超出
	if l < 0 || l > length {
		err = &Crmerror{crm.CrmException{201, "未找到！" + strconv.Itoa(l) + "长度：" + strconv.Itoa(length)}}
		return r, err
	}
	r = CRMS[crmid]
	return r, nil
}

```

### 2.4）编写PHP客户端

```php
<?php

namespace  crm;
error_reporting(E_ALL);


$startTime = getMillisecond();//记录开始时间

$ROOT_DIR = realpath(dirname(__FILE__).'/git.apache.org/thrift.git/lib/php/lib/');
$GEN_DIR = realpath(dirname(__FILE__).'/').'/gen-php';
require_once $ROOT_DIR . '/Thrift/ClassLoader/ThriftClassLoader.php';

use Thrift\ClassLoader\ThriftClassLoader;
use Thrift\Protocol\TBinaryProtocol;
use Thrift\Transport\TSocket;
use Thrift\Transport\TSocketPool;
use Thrift\Transport\TFramedTransport;
use Thrift\Transport\TBufferedTransport;

$loader = new ThriftClassLoader();
$loader->registerNamespace('Thrift',$ROOT_DIR);
$loader->registerDefinition('crm', $GEN_DIR);
$loader->register();

$thriftHost = '127.0.0.1'; //UserServer接口服务器IP
$thriftPort = 9090;            //UserServer端口

$socket = new TSocket($thriftHost,$thriftPort);  
$socket->setSendTimeout(10000);#Sets the send timeout.
$socket->setRecvTimeout(20000);#Sets the receive timeout.
//$transport = new TBufferedTransport($socket); #传输方式：这个要和服务器使用的一致 [go提供后端服务,迭代10000次2.6 ~ 3s完成]
$transport = new TFramedTransport($socket); #传输方式：这个要和服务器使用的一致[go提供后端服务,迭代10000次1.9 ~ 2.1s完成，比TBuffer快了点]
$protocol = new TBinaryProtocol($transport);  #传输格式：二进制格式
$client = new \crm\CrmServerClient($protocol);# 构造客户端

$transport->open();  
$socket->setDebug(true);

//调用ping命令
$client->ping();
//调用添加命令
$crmid = $client->add("名称",12,"这是地址");
var_dump($crmid);
$crmid = $client->add("名称2",21,"这是地址2");
var_dump($crmid);
try{
	$crm = $client->getrowbyid(55);
	print_r($crm->name);
	print_r($crm->crmid);
}catch(\Exception $e){
	print_r($e->getMessage());
}

try{
	$crms = $client->getlist();
	print_r($crms);
}catch(\Exception $e){
	print_r($e->getMessage());
}


$endTime = getMillisecond();

echo "本次调用用时: :".$endTime."-".$startTime."=".($endTime-$startTime)."毫秒\n";

function getMillisecond() {
    list($t1, $t2) = explode(' ', microtime());
    return (float)sprintf('%.0f', (floatval($t1) + floatval($t2)) * 1000);
}

$transport->close();
```

执行结果：

```php
int(2)
int(3)
Internal error processing getrowbyid: {'code':201,'msg':"未找到！"}Array
(
    [0] => crm\Crm Object
        (
            [crmid] => 0
            [name] => 名称
            [age] => 12
            [address] => 这是地址
        )

```

### 2.5）编写GO客户端

```go
package main

import (
	"fmt"
	"gen-go/crm"
	"git.apache.org/thrift.git/lib/go/thrift"
	"net"
	"os"
	"time"
)

const (
	HOST = "127.0.0.1"
	PORT = "9090"
)

func main() {
	startTime := currentTimeMillis()

	transportFactory := thrift.NewTFramedTransportFactory(thrift.NewTTransportFactory())
	protocolFactory := thrift.NewTBinaryProtocolFactoryDefault()

	transport, err := thrift.NewTSocket(net.JoinHostPort(HOST, PORT))
	if err != nil {
		fmt.Fprintln(os.Stderr, "error resolving address:", err)
		os.Exit(1)
	}

	useTransport, err := transportFactory.GetTransport(transport)
	if err != nil {
		fmt.Fprintln(os.Stderr, "error transportFactory", err)
		os.Exit(1)
	}

	client := crm.NewCrmServerClientFactory(useTransport, protocolFactory)
	if err := transport.Open(); err != nil {
		fmt.Fprintln(os.Stderr, "Error opening socket to "+HOST+":"+PORT, " ", err)
		os.Exit(1)
	}
	defer transport.Close()

	//添加
	crmid, err1 := client.Add(nil, "名称", 12, "地址")

	if err1 != nil {
		fmt.Println(err1)
	} else {
		fmt.Println("添加客户：", crmid)
	}

	crm, err2 := client.Getrowbyid(nil, crmid)
	if err2 != nil {
		fmt.Println(err2)
	} else {
		fmt.Printf("获取客户:%v\n", crm)
	}

	crm1, err3 := client.Getrowbyid(nil, 12)
	if err3 != nil {
		fmt.Println(err3)
	} else {
		fmt.Printf("获取客户:%v\n", crm1)
	}

	crms, err4 := client.Getlist(nil)

	if err4 != nil {
		fmt.Println(err4)
	} else {
		fmt.Printf("获取到客户:%v", crms)
	}

	endTime := currentTimeMillis()
	fmt.Printf("本次调用用时:%d-%d=%d毫秒\n", endTime, startTime, (endTime - startTime))

}

func currentTimeMillis() int64 {
	return time.Now().UnixNano() / 1000000
}

```

```go
➜  src go run crmclient.go
添加客户： 1
获取客户:Crm({Crmid:1 Name:名称 Age:12 Address:地址})
Internal error processing getrowbyid: {'code':201,'msg':"未找到！12长度：2"}
获取到客户:[Crm({Crmid:0 Name:名称 Age:12 Address:地址}) Crm({Crmid:1 Name:名称 Age:12 Address:地址})]本次调用用时:1513150613817-1513150613815=2毫秒
```

