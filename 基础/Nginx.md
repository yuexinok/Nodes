# Nginx

1) 静态资源服务

通过本地文件系统提供服务

2）反向代理服务

3）API服务

优点：高并发，高性能，可扩展性好，热部署，BSD许可证

阿里巴巴：Tengine 基于Nginx的基础上，满足大访问量网站的需求。

https://nginx.org/en/docs/

OpenResty® 是一个基于 [Nginx](https://openresty.org/cn/nginx.html) 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第三方模块以及大多数的依赖项。用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。

https://openresty.org/cn/

## 1）常用命令

#### -s 发送信号

nginx -s** reload|stop|quit

Stop：立刻停止服务

quit：优雅的停止服务

reload:重置配置文件

reopen：重新开始急了日志文件

#### -? -h 帮助

#### -c 指定配置文件

#### -p 指定运行目录

#### -t -T 测试配置文件是否有语法错误

#### -v 打印nginx版本信息