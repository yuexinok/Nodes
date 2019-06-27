# SpringBootLog4j日志

## 1，Log4j的3大组件

### Loggers记录器

准许定义多个，每个loggers有自己的名字

**7级别：**all->debug->info->warn->error->fatal->off 

### Appenders输出源

即日志输出到哪里，如Consul控制台，Files文件

### Layouts布局

用于控制输出的方式

## 2，使用log4j2

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

