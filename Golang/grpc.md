# gRPC

#### 参考：

https://www.jianshu.com/p/774b38306c30

http://doc.oschina.net/grpc

https://grpc.io/docs/quickstart/go.html

### 安装protoc3：

```shell
brew install protobuf
#安装 automake
brew install automake
brew install libtool

➜  ~ protoc --version
libprotoc 3.5.1
```

### 下载gRPC-go包：

```go
go get google.golang.org/grpc
```

是"https://github.com/grpc/grpc-go",但是为什么要用“google.golang.org/grpc”进行安装呢？应该grpc原本是google内部的项目，归属golang，就放在了google.golang.org下面了，后来对外开放，又将其迁移到github上面了，又因为golang比较坑爹的import路径规则，所以就都没有改路径名了。

### 编写proto文件：

hello.proto：

```protobuf
syntax = "proto3";
option objc_class_prefix = "HLW";

package helloworld;

service Greeter {
    rpc SayHello(HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
    string name = 1;
}

message HelloReply {
    string message = 1;
}
```

### 编译proto文件：

```shell
➜  src go get -u github.com/golang/protobuf/proto
➜  src go get -u github.com/golang/protobuf/protoc-gen-go
➜  helloworld protoc --go_out=plugins=grpc:. hello.proto 
➜  helloworld ls
hello.pb.go hello.proto
➜  src ls
client.go         google.golang.org server.go
github.com        helloworld
```



### 编写服务端：





### 理解：

#### 单项 RPC

rpc GetFeature(Point) returns (Feature) {}

首先我们来了解一下最简单的 RPC 形式：客户端发出单个请求，获得单个响应。

- 一旦客户端通过桩调用一个方法，服务端会得到相关通知 ，通知包括客户端的元数据，方法名，允许的响应期限（如果可以的话）
- 服务端既可以在任何响应之前直接发送回初始的元数据，也可以等待客户端的请求信息，到底哪个先发生，取决于具体的应用。
- 一旦服务端获得客户端的请求信息，就会做所需的任何工作来创建或组装对应的响应。如果成功的话，这个响应会和包含状态码以及可选的状态信息等状态明细及可选的追踪信息返回给客户端 。
- 假如状态是 OK 的话，客户端会得到应答，这将结束客户端的调用。

#### 服务端流式 RPC

rpc ListFeatures(Rectangle) returns (**stream** Feature) {}

服务端流式 RPC 除了在得到客户端请求信息后发送回一个应答流之外，与我们的简单例子一样。在发送完所有应答后，服务端的状态详情(状态码和可选的状态信息)和可选的跟踪元数据被发送回客户端，以此来完成服务端的工作。客户端在接收到所有服务端的应答后也完成了工作。

#### 客户端流式 RPC

rpc RecordRoute(**stream** Point) returns (RouteSummary) {}

客户端流式 RPC 也基本与我们的简单例子一样，区别在于客户端通过发送一个请求流给服务端，取代了原先发送的单个请求。服务端通常（但并不必须）会在接收到客户端所有的请求后发送回一个应答，其中附带有它的状态详情和可选的跟踪数据。

#### 双向流式 RPC

rpc RouteChat(**stream** RouteNote) returns (**stream** RouteNote) {}

双向流式 RPC ，调用由客户端调用方法来初始化，而服务端则接收到客户端的元数据，方法名和截止时间。服务端可以选择发送回它的初始元数据或等待客户端发送请求。下一步怎样发展取决于应用，因为客户端和服务端能在任意顺序上读写 - 这些流的操作是完全独立的。例如服务端可以一直等直到它接收到所有客户端的消息才写应答，或者服务端和客户端可以像"乒乓球"一样：服务端后得到一个请求就回送一个应答，接着客户端根据应答来发送另一个请求，以此类推。