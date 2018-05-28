# Spring Boot 视频学习
小马哥 阿里巴巴
服务注册和发现：Registry and Discovery
路由 Routing
可靠性 Reliability
延迟 Latency
热点 Hotspot
短路 Circuit Break
伸缩 Scale
异步 Async
监控 Monitoring
配置 Configuration
数据同步 Data Sync
安全 Security
## 源编程 Meta Programming
如：注解驱动，反射驱动，表达式驱动，Lambda，Scripy On JVM

接口编程：契约编程

> model2
> 
> 
@RestController @Controller
@RequestMapping(path=""),@GetMapping(path="") @PostMapper(path="")
@PathVariable String name 获取路径变量参数（@RequestMapper(path="/get/{name}")）@GetMapping(path="/json/user",produces = MediaType.APPLICATION_JSON_VALUE)
@RequestParam String name 获取url参数 @RequestParam(value = "username",Required = false,defualtValue ="ddd") String name 好处可以类型转换
HttpServletRequest request   request.getParameter("name")

@RequestHeader(value = "Accept") String accept 获取请求头像信息
@CookieValue
@ResponseBody

Swagger 文档生成
// 自带一个 mappings
endpoint.enabled = true
endpoint.

客户端
RestTemplate t = new RestTeamplate();
t.getForObject("https://127.0.0.1:8090/get/users);

Apache的HttpClient

传统Servlet容器
Eclipse Jetty
Apache Tomcat

数据源：DataSource接口
通用型DataSource,分布式XADataSource,嵌入式

事务概念：
自动提交：Auto-commit mode
触发时机：DML执行后，DDL执行后，SELECT查询后关闭后，存储过程执行后

事务隔离级别：isolation levels
脏读：dirty reads
不可重现读，nonrepeatatable reads
幻读：phantom reads

保护点，还原点，Savepoints 如快照
使用场景：部分事务回顾，选择性释放

java.sql.Driver
java.sql.DriverManager
java.sal.DataSource

    


