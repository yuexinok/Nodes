# Arthas调试实战

https://arthas.aliyun.com/doc/

1. 这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？
2. 我改的代码为什么没有执行到？难道是我没 commit？分支搞错了？
3. 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？
4. 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
5. 是否有一个全局视角来查看系统的运行状况？
6. 有什么办法可以监控到JVM的实时运行状态？
7. 怎么快速定位应用的热点，生成火焰图？
8. 怎样直接从JVM内查找某个类的实例？

## 环境准备

本机下载：

```shell
➜  soft pwd
/Users/borgxiao/Documents/work/soft
➜  soft curl -O https://arthas.aliyun.com/arthas-boot.jar
```

公司应用：

```shell
[root@crm-core-k8s crm-core]# cd /ec/apps/arthas-3.1.4/
[root@crm-core-k8s arthas-3.1.4]# pwd
/ec/apps/arthas-3.1.4
[root@crm-core-k8s arthas-3.1.4]# ls
arthas-agent.jar  arthas-client.jar  arthas-demo.jar  as.bat          as.sh
arthas-boot.jar   arthas-core.jar    arthas-spy.jar   as-service.bat  install-local.sh
```

### 启动应用：

```java
➜  soft java -jar arthas-boot.jar
[INFO] arthas-boot version: 3.5.4
[INFO] Process 94510 already using port 3658
[INFO] Process 94510 already using port 8563
[INFO] Found existing java process, please choose one and input the serial number of the process, eg : 1. Then hit ENTER.
* [1]: 94510 com.ec.crm.core.CrmCoreApplication
  [2]: 40034
  [3]: 7596 /usr/local/Cellar/activemq/5.15.5/libexec//bin/activemq.jar
1
[INFO] arthas home: /Users/borgxiao/.arthas/lib/3.5.4/arthas
[INFO] The target process already listen port 3658, skip attach.
[INFO] arthas-client connect 127.0.0.1 3658
```

## 常用命令

### [dashboard] 查看全盘大盘

![arthas_dashboard](./images/arthas_dashboard.png)

- ID: Java级别的线程ID，注意这个ID不能跟jstack中的nativeID一一对应。
- NAME: 线程名
- GROUP: 线程组名
- PRIORITY: 线程优先级, 1~10之间的数字，越大表示优先级越高
- STATE: 线程的状态
- CPU%: 线程的cpu使用率。比如采样间隔1000ms，某个线程的增量cpu时间为100ms，则cpu使用率=100/1000=10%
- DELTA_TIME: 上次采样之后线程运行增量CPU时间，数据格式为`秒`
- TIME: 线程运行总CPU时间，数据格式为`分:秒`
- INTERRUPTED: 线程当前的中断位状态
- DAEMON: 是否是daemon线程

### [thread] 查看线程情况

#### 查看所有线程：

直接thread命令即可。

#### 查看单个线程：

thread  线程ID

```shell
[arthas@95428]$ thread 761
"Hashed wheel timer #1" Id=761 TIMED_WAITING
    at java.lang.Thread.sleep(Native Method)
    at org.jboss.netty.util.HashedWheelTimer$Worker.waitForNextTick(HashedWheelTimer.java:445)
    at org.jboss.netty.util.HashedWheelTimer$Worker.run(HashedWheelTimer.java:364)
    at org.jboss.netty.util.ThreadRenamingRunnable.run(ThreadRenamingRunnable.java:108)
    at java.lang.Thread.run(Thread.java:748)
```

#### 查看最忙前10线程：

thread -n 10

#### 查看所有线程：

thread -all

#### 查看阻塞其他线程的线程：

thread -b

#### 查看指定状态的线程：

thread --state WATING/RUNABLE

```shell
[arthas@95428]$ thread --state watting
Illegal argument, state should be one of [RUNNABLE, BLOCKED, WAITING, TIMED_WAITING, NEW, TERMINATED]
```

### [jvm]查看jvm信息

命令 jvm

```shell
----------------------------------------------------------------------------------------------
 MEMORY
----------------------------------------------------------------------------------------------
 HEAP-MEMORY-USAGE          init : 268435456(256.0 MiB)
 [memory in bytes]          used : 980847576(935.4 MiB)
                            committed : 1571291136(1.5 GiB)
                            max : 3817865216(3.6 GiB)

 NO-HEAP-MEMORY-USAGE       init : 2555904(2.4 MiB)
 [memory in bytes]          used : 149469328(142.5 MiB)
                            committed : 165490688(157.8 MiB)
                            max : -1(-1 B)

 PENDING-FINALIZE-COUNT     0
```

### [sysprop]获取系统属性

获取系统属性全部值

sysprop 属性：获取单个属性值

```shell
[arthas@95428]$ sysprop java.version
 KEY                VALUE
----------------------------------------------------------------------------------------------
 java.version       1.8.0_141
```

### [sysenv]获取系统环境变量

### [vmoption]查看或修改vm参数

```shell
[arthas@95428]$ vmoption
 KEY                     VALUE                  ORIGIN                  WRITEABLE
----------------------------------------------------------------------------------------------
 HeapDumpBeforeFullGC    false                  DEFAULT                 true
 HeapDumpAfterFullGC     false                  DEFAULT                 true
 HeapDumpOnOutOfMemoryE  false                  DEFAULT                 true
```

#### 修改gc参数

```shell
[arthas@95428]$ vmoption PrintGC
 KEY                     VALUE                  ORIGIN                  WRITEABLE
----------------------------------------------------------------------------------------------
 PrintGC                 false                  DEFAULT                 true
[arthas@95428]$ vmoption PrintGC true
Successfully updated the vm option.
 NAME     BEFORE-VALUE  AFTER-VALUE
------------------------------------
 PrintGC  false         true
[arthas@95428]$ vmoption PrintGC
 KEY                     VALUE                  ORIGIN                  WRITEABLE
----------------------------------------------------------------------------------------------
 PrintGC                 true                   MANAGEMENT              true
```

### [logger]查看日志或修改日志级别

查看日志类信息

```shell
[arthas@95428]$ logger
 name            root
 class           org.apache.logging.log4j.core.config.LoggerConfig
 classLoader     sun.misc.Launcher$AppClassLoader@764c12b6
 classLoaderHas  764c12b6
 h
 level           INFO
 config          XmlConfiguration[location=/Users/borgxiao/Documents/work/webjava/crm-applica
                 tion-services/crm-core/target/classes/log4j2.xml]
 additivity      true
 codeSource      file:/Users/borgxiao/.m2/repository/org/apache/logging/log4j/log4j-core/2.13
                 .2/log4j-core-2.13.2.jar
 appenders       name            CONSOLE
                 class           org.apache.logging.log4j.core.appender.ConsoleAppender
                 classLoader     sun.misc.Launcher$AppClassLoader@764c12b6
                 classLoaderHash 764c12b6
                 target          SYSTEM_OUT
```

#### 查看指定类的日志类：

```shell
[arthas@95428]$ logger -n com.ec.crm.core.mapper.dao
 name            com.ec.crm.core.mapper
 class           org.apache.logging.log4j.core.config.LoggerConfig
 classLoader     sun.misc.Launcher$AppClassLoader@764c12b6
 classLoaderHas  764c12b6
 h
 level           INFO
 additivity      true
 codeSource      file:/Users/borgxiao/.m2/repository/org/apache/logging/log4j/log4j-core/2.13
                 .2/log4j-core-2.13.2.jar
```

#### 更新日志级别

```shell
[arthas@95428]$ logger --name com.ec.crm.core.mapper.dao --level debug
Update logger level success.
```

### [ognl]动态执行代码

### [sc]获取类型信息

sc -d 类名称 获取类信息

```shell
[arthas@24922]$ sc -d com.ec.crm.core.CrmCoreApplication
 class-info        com.ec.crm.core.CrmCoreApplication
 code-source       /Users/borgxiao/Documents/work/webjava/crm-application-services/crm-core/t
                   arget/classes/
 name              com.ec.crm.core.CrmCoreApplication
 isInterface       false
 isAnnotation      false
 isEnum            false
 isAnonymousClass  false
 isArray           false
```

### [sm]获取类的方法信息

sm -d 类名称

### [dump]导出类的 bytecode

dump 类名称

```shell
[arthas@24922]$ dump com.ec.crm.core.CrmCoreApplication
 HASHCODE  CLASSLOADER                                    LOCATION
 764c12b6  +-sun.misc.Launcher$AppClassLoader@764c12b6    /Users/borgxiao/logs/arthas/classdu
             +-sun.misc.Launcher$ExtClassLoader@5d5eef3d  mp/sun.misc.Launcher$AppClassLoader
                                                          -764c12b6/com/ec/crm/core/CrmCoreAp
                                                          plication$$EnhancerBySpringCGLIB$$b
                                                          46918f7.class
```

注：对应目录 logs/arthas/*.. 

### [heapdump] 导出应用堆=jmap

heapdump 文件

```shell
#导出动指定文件
[arthas@58205]$ heapdump /tmp/dump.hprof
Dumping heap to /tmp/dump.hprof...
Heap dump file created

#只导出存活的
[arthas@58205]$ heapdump --live /tmp/dump.hprof
Dumping heap to /tmp/dump.hprof...
Heap dump file created

#导出到临时文件
[arthas@58205]$ heapdump
Dumping heap to /var/folders/my/wy7c9w9j5732xbkcyt1mb4g40000gp/T/heapdump2019-09-03-16-385121018449645518991.hprof...
Heap dump file created
```

### [vmtool]

强制gc

```shell
vmtool --action forceGc
```



### [jad] 反编译类文件查看

jad 类完整路径

```java
[arthas@95428]$ jad com.ec.crm.core.CrmCoreApplication

ClassLoader:
+-sun.misc.Launcher$AppClassLoader@764c12b6
  +-sun.misc.Launcher$ExtClassLoader@5d5eef3d

Location:
/Users/borgxiao/Documents/work/webjava/crm-application-services/crm-core/target/classes/

       /*
        * Decompiled with CFR.
        */
       package com.ec.crm.core;

       import com.ec.crm.boot.application.annotation.CrmSpringBootApplication;
```

### [classloader]查看应用classloader信息

   classloader
   classloader -t
   classloader -l

### [mc]编译java生成class

编译`.java`文件生成`.class`

### [monitor] 接口统计接口方法调用情况

```shell
[arthas@95554]$ monitor com.ec.crm.core.service.impl.MoveCrmServiceImpl run -c 1
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 2) cost in 45 ms, listenerId: 3
 timestamp     class                 method                total  succe  fail    avg-r  fail-
                                                                  ss             t(ms)  rate
----------------------------------------------------------------------------------------------
 2021-11-01 1  com.ec.crm.core.serv  run                   2      2      0       4874.  0.00%
 1:20:12       ice.impl.MoveCrmServ                                              37
               iceImpl
```



### [watch]查看函数的返回值

watch 类完整路径 方法名称

能观察到的范围为：`返回值`、`抛出异常`、`入参`，通过编写 OGNL 表达式进行对应变量的查看。

```shell
[arthas@95428]$ watch com.ec.crm.core.service.impl.CheckRepeatServiceImpl getExistCrm
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 170 ms, listenerId: 1
method=com.ec.crm.core.service.impl.CheckRepeatServiceImpl.getExistCrm location=AtExit
ts=2021-10-28 14:51:16; [cost=1.146818ms] result=@ArrayList[
    @Object[][isEmpty=false;size=9],
    @CheckRepeatServiceImpl[com.ec.crm.core.service.impl.CheckRepeatServiceImpl@17fa25f1],
    null,
]
```

```shell
[arthas@41617]$ watch com.ec.crm.core.service.ReceivePubCrmService run "params,target,returnObj" "params[1] == 64634"
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 138 ms, listenerId: 8
method=com.ec.crm.core.service.impl.ReceivePubCrmServiceImpl.run location=AtExit
ts=2021-11-01 14:20:07; [cost=823.175579ms] result=@HashMap[
    @Long[3027171071]:@CrmOperateResultDto[CrmOperateResultDto(crmId=3027171071, crmName=小写, company=, isOk=true, userId=0, businessId=166, receiveBusinessId=null, error=null)],
]
method=com.ec.crm.core.service.impl.ReceivePubCrmServiceImpl.run location=AtExit
ts=2021-11-01 14:20:07; [cost=830.852894ms] result=@HashMap[
    @Long[3027171071]:@CrmOperateResultDto[CrmOperateResultDto(crmId=3027171071, crmName=小写, company=, isOk=true, userId=0, businessId=166, receiveBusinessId=null, error=null)],
]
```

注：观察表达式，默认值：`{params, target, returnObj}`

```shell
#耗时统计
$ watch demo.MathGame primeFactors '{params, returnObj}' '#cost>200' -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 66 ms.
ts=2018-12-03 19:40:28; [cost=2112.168897ms] result=@ArrayList[
    @Object[][
        @Integer[1],
    ],
    @ArrayList[
        @Integer[5],
        @Integer[428379493],
    ],
]
```

### [trace]分布式跟踪类方法



### [stack]堆栈信息



### [tt] 统计方法耗时或者回放

方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测

### [profiler] 查看火焰图

```shell
[arthas@51619]$ profiler start
Started [cpu] profiling
[arthas@51619]$ profiler getSamples
38
[arthas@51619]$ profiler status
[cpu] profiling is running for 50 seconds
[arthas@51619]$ profiler stop
OK
profiler output file: /Users/borgxiao/Documents/work/webjava/crm-application-services/arthas-output/20211105-155006.svg
```

