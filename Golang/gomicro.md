## go-micro

微服务框架：

自带protocbuf（引用的grpc）

```shell
go get -u github.com/micro/protobuf/proto 
go get -u github.com/micro/protobuf/protoc-gen-go 
protoc --go_out=plugins=micro:. ./helloworld/hello.proto
➜  src ls                                                      
github.com helloworld server
➜  src cd helloworld 
➜  helloworld ls
hello.pb.go hello.proto

```

因为之前用的grpc，所以在编译的时候出现了很多问题，因为-u参数