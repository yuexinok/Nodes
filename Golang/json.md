# JSON解析

http://www.flysnow.org/2017/11/05/go-auto-choice-json-libs.html

golang官方为我们提供了标准的json解析库–`encoding/json`，大部分情况下，使用它已经够用了。不过这个解析包有个很大的问题–性能。它不够快，如果我们开发高性能、高并发的网络服务就无法满足，这时就需要高性能的json解析库，目前性能比较高的有`json-iterator`和`easyjson`。

go build -tags=jsoniter

自动选择编译库