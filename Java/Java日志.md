# Java日志

## 日志注意事项：

1）**应用中的扩展日志命名方式应该有统－的约定**，通过命名能直观明了地表明当前日志文件是什么功能，如监控、访问日志等。推荐的日志文件命名方式为appName_logType logName.log 。其中，**log Type 为**
**日志类型，推荐分类有stats 、monitor 、visit 等**，logNam e 为日志描述。这种命名的好处是通过文件名就可以知道曰志文件属于什么应用，什么类型，什么目的，也有利于归类查找。例如，mppserv er 应用中单独监控时区转换异常的日志文件名定义为mppserver monitor timeZoneConvert.log 。

2）代码规约推荐曰志文件至少保存1 5 天，可以根据日志文件的重要程度、文件大小及磁盘空间再自行延长保存时间。

3）预先判断曰志级别，避免无效日志打印，区别对待错误日志，保证记录内容完整

## 日志框架分类：

### 日志门面

门面设计模式是面向对象设计模式中的一种，日志框架采用的就是这种模式，类
似JDB C 的设计理念。它只提供一套接口规范，自身不负责日志功能的实现，目的是
让使用者不需要关注底层具体是哪个日志库来负责日志打印及具体的使用细节等。目
前用得最为广泛的曰志门面有两种**slf4j 和commons -logging** 。

### 日志库

它具体实现了日志的相关功能，主流的日志库有三个，分别是log4j 、log -jdk 、
logback 。最早Java 要想记录曰志只能通过System.out 或System.err 来完成，非常不方便。
log4j 就是为了解决这一问题而提出的，它是最早诞生的曰志库。接着JD K 也在1 .4 版
本引入了一个日志库java. util.logging. Logger.，简称log-dk。这样市面上就出现两种日志
功能的实现，开发者在使用时需要关注所使用的日志库的具体细节。logback 是最晚出
现的，它与log4j 出自同一个作者，是log4j的升级版且本身就实现了slf4j的接口。

### 日志适配器

曰志适配器分两种场景
( I ）日志门面适配器，因为slf4j规范是后来提出的此之前的日志库是没有
实现slf4j的接口的，例如log4j ；所以在工程里要想使用slf4j +log4j 的模式，就额
外需要个适配器（slf4j-log4j12 来解决接口不兼容的问题。
( 2 ）日志库适配器，在一些老的工程里，一开始为了开发简单而直接使用了日志库API来完成曰志打印，随着时间的推移想将原来直接调用日志库的模式改为业界标准的门面模式（例如slf4j +logback 组合），但老工程代码里打印曰志的地方太多，难以改动，所以需要个运队器来完成从旧日志库的API 到slf4j 的路由，这样在不改动原有代码的情况l、也能使用slf4j叫来统一管理曰志，而日后续自由替换具体日志库也不成可题。

![](/Users/borgxiao/Documents/work/notes/Java/images/log.png)

## 日志框架

### JDK Logger

jdk自带的一套日志框架，可以直接使用，使用简单

日志分为9个级别：all，finest(优美的)，finer(精细)，fine(好的)，config，info，warning，serve，off

IDK Logger 默认在控制台输出，并且输出 info 级别和高于 info 级别的信息，我们也可以通过调用 Logger setLevel （）方法或者通过配置文件来设置，当然，我们也可以设置多个输出，对每个输出设置不同的级别，然后把输出的日志添加到同 个或者多个日志文件中来集中管理

 JDK 自带的llogger 日志框架可谓鸡肋，在易用性、功能及扩展性方面都要稍逊一筹，所以很少在线上系统中被使用。

### Apache Commons Logging

最早得到广泛使用的是 Log4j ，许多应用程序的日志记录功能都由 Log4j 实现。不过，作为应用开发者，我们希望自己的组件不依赖于某个日志工具，毕竟还有很多其他日志工具可用 ，如果存在性能或者其他问题，需要在日志实现框架切换，为了解决切换带来的问题，所以commons Logging到来

提供了日志接口，具体的实现根据运行时的配置来动态查找日志的实现框架，有了它，开发人员可以针对commons logging api进行编程，运行时候可以选择不同的日志实现框架

所以基本上是 commons Logging（日志门面）+Log4j(日志实现框架) 来记录日志

配置：

1） org.apache cornrnons logging.LogFactory

2）META－INF/se ices/org.apache.commons. logging. LogFactory 

3）commons-logging. properties

4）默认自己的SimpleLog

### Apache Log4j

Apache Log4j （简称 Log4j ）是一款由 Java 编写的可靠、灵活的日志框架，是 Apache 旗下个开源项目，如今， log4j 经被移植到了多种语言中 ，服务于更多的开发者

#### Log4j的3大组件

##### Loggers记录器

准许定义多个，每个loggers有自己的名字，负责捕捉日志记录的信息

**7级别：**all->debug->info->warn->error->fatal(致命)->off 

##### Appenders输出源

即日志输出到哪里，如ConsoleAppender（Consul控制台），FileAppender（Files文件）

**DailyRollingFileAppender** ：每天产生一个日志磁盘文件，日志文件按天滚动生成。

**RollingFileAppender** ：日志磁盘文件的大小达到指定尺寸时会产生一个新的文件，日志文件按照日志大小滚动生成

##### Layouts布局

用于控制输出的方式，对日志进行格式化，负责生成不同格式的日志信息

#### 配置

log4j.properties，log4j.xml

#### 性能问题：

性能低下

### Slf4j：

Slf4j ( Simple Logging Facade for Java ）与 Apache Commons Logging 样，都是使用门面模式对外提供统 的日志接口，应用程序可以只依赖于slf4j来实现日志打印，具体的日志实现由配置来决定使用 Log4j 还是 Logback 等，在不改变应用代码的前提下切换底层的日志实现

### Logback:

Logback 相对于 Log4j 的最大提升是效率 Logback Log4j 的内 进行了重写和优化，一些关键执行路径上性能提升了至少 10 ，初始化内存加载也变得更小了

logback.xml

### Apache log4j2

是log4j的升级版本，且提供了logback所有高级特性，性能也提供了，

Log4j 实现了 API 模块和实现模块的分离，如图 4-6 示，它包含两个 Jar 包，个是 log4j-api ，另 个是 log4j-core.jar ，前者提供 Log4j 对外提供的 API 主要包含 Logger类和 LogManager ，后者包含实现日志记录功能的核心基础类

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-log4j2</artifactId>
        </dependency>

<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>rg.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-logging</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

```ini
#日志
logging.config=classpath:log4j2.xml
```



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<Configuration status="WARN" >
    <Appenders>
        <Console name="allLog" target="SYSTEM_OUT">
            <PatternLayout pattern="[%d{HH:mm:ss:SSS}] [%p] - %1 - %m%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Root level="all">
            <AppenderRef ref="allLog" />
        </Root>
    </Loggers>
</Configuration>
```

## 日志格式配置：

• %p ：输出日志信息的优先级，即 debug info warn eηor fatal

• %d ：输出日志时间点的日期或时间，默认格式为 IS08601 ，也可以在其后指定格式，比如%d{yyy MMM dd HH:mm:ss,SSS ｝，输出类似于“2017 06 18 12: 01: 12, 058 ”。

• %r ：输出自应用启动到输出该 Log 信息所用的毫秒数。

• %c ：输出日志信息所属的类目，通常就是所在类的全名。

• %t ：输出产生该日志事件的钱程名

• %M：输出产生该日志的方法名

• %1 ：输出日志事件的发生位直，相当于%C.%M(%F:%L）的组合，包括类名、发生的线程，以及在代码中的行数，例如 Log4jDemo.main(Log4jDemo.java:22）。

• %x ：输出和当前线程相关的 NDC 嵌套诊断环境），主要用于 Servlet 这样的多客户、多线程的 We 应用中

• %%：输出 个’%’字符

 • %F ：输出日志消息产生时所在的文件名称。

 •%L ：输出代码中的行号

• %m ：输出代码中指定的消息。

• %n ：输出 个回车换行符， Windows 平台为’’\r\n”， UNIX Linux 平台为"\n”日志信息换行。

• %30c ：指定输出 category 的名称，最小的宽度是 30 个字符，如果 category 的名称少于20 个字符，则在默认情况下右对齐。

## 日志采集系统

### 日志采集器

Logstash

Fluentd

Flume

Scribe

Rsyslog

### 日志缓存队列

kafka,Redis,RabbitMQ

### 日志解析器

Logstash,Fluentd

### 日志存储和检索

Elasticsearch

Solr

## ELK

ELK 项目是开源项目 Elasticsearch Logstash Kibana 集合，集合中每个项目的职责如

下。

• Elasticsearch 是基于 Lucene 搜索引擎的 NoSQL 数据库

• Logstash 个基于管道的处理工具，它从不同的数据源接收数据，执行不同的转换，然后发送数据到不同的目标系统。

• Kibana 工作在 elasticsearch 上，是数据的展示层系统。