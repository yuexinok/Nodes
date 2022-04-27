# JAVA调试工具

## jps查看java进程

Java Virtual Machine Process Status Tool 查看当前（用户）系统的java进程情况

类似 pgrep java  或者 ps -ef grep java

```shell
➜  ~ jps
29424
31851 KotlinCompileDaemon
31852 Launcher
31853 CrmInitDataApplication
32174 Jps

//只展示进程名称
➜  ~ jps -q 
29424
31851
31852
32188
31853
➜  ~ jps -m
31851 KotlinCompileDaemon --daemon-runFilesPath /Users/borgxiao/Library/Application Support/kotlin/daemon --daemon-autoshutdownIdleSeconds=7200 --daemon-compilerClasspath /Applications/IntelliJ IDEA.app/Contents/plugins/Kotlin/kotlinc/lib/kotlin-compiler.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_141.jdk/Contents/Home/lib/tools.jar:/Applications/IntelliJ IDEA.app/Contents/plugins/Kotlin/kotlinc/lib/kotlin-daemon.jar

➜  ~ jps -l
29424
32209 sun.tools.jps.Jps
31851 org.jetbrains.kotlin.daemon.KotlinCompileDaemon
31852 org.jetbrains.jps.cmdline.Launcher
31853 com.ec.crm.init.data.CrmInitDataApplication

//更加详细 
➜  ~ jps -v
29424  -Xms128m -Xmx2048m -XX:ReservedCodeCacheSize=240m -XX:+UseG1GC -XX:+UserNUMA -XX:SoftRefLRUPolicyMSPerMB=50 -ea -XX:CICompilerCount=2 -Dsun.io.useCanonPrefixCache=false -Djava.net.preferIPv4Stack=true -Djdk.http.auth.tunneling.disabled Schemes="" -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -Djdk.attach.allowAttachSelf=true -Dkotlinx.coroutines.debug=off -Djdk.module.illegalAccess.silent=true -XX:+UseCompressedOops -Dfile.encoding=UTF-8 -XX:ErrorFile=/Users/borgxiao/java_error_in_idea_%p.log -XX:HeapDumpPath=/Users/borgxiao/java_error_in_idea.hprof -javaagent:/Users/borgxiao/Downloads/jetbrains-agent/jetbrains-agent.jar -Djb.vmOptionsFile=/Users/borgxiao/Library/Preferences/IntelliJIdea2019.3/idea.vmoptions -Didea.home.path=/Applications/IntelliJ IDEA.app/Contents -Didea.executable=idea -Didea.paths.selector=IntelliJIdea2019.3
```



## jstack打印线程堆栈

java线程堆栈跟踪工具，可以用来分析线程死锁问题等

```shell
NEW,未启动的。不会出现在Dump中。

RUNNABLE,在虚拟机内执行的。

BLOCKED,受阻塞并等待监视器锁。

WATING,无限期等待另一个线程执行特定操作。

TIMED_WATING,有时限的等待另一个线程的特定操作。

TERMINATED,已退出的。
```

```shell
-F当’jstack [-l] pid’没有相应的时候强制打印栈信息 -l长列表. 打印关于锁的附加信息,例如属于java.util.concurrent的ownable synchronizers列表. -m打印java和native c/c++框架的所有栈信息. -h | -help打印帮助信息 pid 需要被打印配置信息的java进程id,可以用jps查询.
jstack -l 29424  

```

```java
"Reference Handler" #2 daemon prio=10 os_prio=31 cpu=231.36ms elapsed=23621.78s tid=0x00007fd5e402f800 nid=0x3d03 waiting on condition  [0x0000700006a4e000]
   java.lang.Thread.State: RUNNABLE
        at java.lang.ref.Reference.waitForReferencePendingList(java.base@11.0.5/Native Method)
        at java.lang.ref.Reference.processPendingReferences(java.base@11.0.5/Reference.java:241)
        at java.lang.ref.Reference$ReferenceHandler.run(java.base@11.0.5/Reference.java:213)

   Locked ownable synchronizers:
        - None

"Finalizer" #3 daemon prio=8 os_prio=31 cpu=159.30ms elapsed=23621.78s tid=0x00007fd5e4807000 nid=0x4403 in Object.wait()  [0x0000700006b51000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(java.base@11.0.5/Native Method)
        - waiting on <no object reference available>
        at java.lang.ref.ReferenceQueue.remove(java.base@11.0.5/ReferenceQueue.java:155)
        - waiting to re-lock in wait() <0x0000000780105198> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(java.base@11.0.5/ReferenceQueue.java:176)
        at java.lang.ref.Finalizer$FinalizerThread.run(java.base@11.0.5/Finalizer.java:170)

   Locked ownable synchronizers:
        - None

"VM Thread" os_prio=31 cpu=3575.00ms elapsed=23779.61s tid=0x00007fd5e402e800 nid=0x4603 runnable  

"GC Thread#0" os_prio=31 cpu=13021.33ms elapsed=23779.65s tid=0x00007fd5e4010800 nid=0x4b03 runnable  

"GC Thread#1" os_prio=31 cpu=12887.00ms elapsed=23777.84s tid=0x00007fd5e48c1800 nid=0x9f03 runnable  

"GC Thread#2" os_prio=31 cpu=12999.29ms elapsed=23777.84s tid=0x00007fd5e48bd800 nid=0x9e03 runnable  
```

### 死锁：

```
Found one Java-level deadlock
Java stack information for the threads listed above:
```



## jmap打印堆内存

outOfMemoryError **年老代内存不足。**
outOfMemoryError:PermGen Space **永久代内存不足。**
outOfMemoryError:GC overhead limit exceed **垃圾回收时间占用系统运行时间的98%或以上。**

```shell
➜  ~ jmap 31852
Attaching to process ID 31852, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.141-b15

  #查看堆内存
  ➜  ~ jmap -heap 31852
Attaching to process ID 31852, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.141-b15

using thread-local object allocation.
Parallel GC with 4 thread(s)

Heap Configuration:
   MinHeapFreeRatio         = 0
   MaxHeapFreeRatio         = 100
   MaxHeapSize              = 734003200 (700.0MB)
   NewSize                  = 89128960 (85.0MB)
   MaxNewSize               = 244318208 (233.0MB)
   OldSize                  = 179306496 (171.0MB)
   NewRatio                 = 2
   SurvivorRatio            = 8
   MetaspaceSize            = 21807104 (20.796875MB)
   CompressedClassSpaceSize = 1073741824 (1024.0MB)
   MaxMetaspaceSize         = 17592186044415 MB
   G1HeapRegionSize         = 0 (0.0MB)

Heap Usage:
PS Young Generation
Eden Space:
   capacity = 67108864 (64.0MB)
   used     = 62300256 (59.414154052734375MB)
   free     = 4808608 (4.585845947265625MB)
   92.83461570739746% used
From Space:
   capacity = 11010048 (10.5MB)
   used     = 8900040 (8.487739562988281MB)
   free     = 2110008 (2.0122604370117188MB)
   80.83561488560268% used
To Space:
   capacity = 11010048 (10.5MB)
   used     = 0 (0.0MB)
   free     = 11010048 (10.5MB)
   0.0% used
PS Old Generation
   capacity = 179306496 (171.0MB)
   used     = 81936 (0.0781402587890625MB)
   free     = 179224560 (170.92185974121094MB)
   0.04569605777138158% used

4604 interned Strings occupying 369560 bytes.

#查看对象和大小
  ~ jmap -histo 31853
  num     #instances         #bytes  class name
  编号     个数                字节     类名
  ----------------------------------------------
  1:             7        1322080  [I
  2:          5603         722368  <methodKlass>
  3:          5603         641944  <constMethodKlass>
  4:         34022         544352  java.lang.Integer
```



### jmap -dump:format=b,file=heapDump 6900

dump文件

1.如果程序内存不足或者频繁GC，很有可能存在内存泄露情况，这时候就要借助Java堆Dump查看对象的情况。
2.要制作堆Dump可以直接使用jvm自带的jmap命令
3.可以先使用`jmap -heap`命令查看堆的使用情况，看一下各个堆空间的占用情况。
4.使用`jmap -histo:[live]`查看堆内存中的对象的情况。如果有大量对象在持续被引用，并没有被释放掉，那就产生了内存泄露，就要结合代码，把不用的对象释放掉。
5.也可以使用 `jmap -dump:format=b,file=`命令将堆信息保存到一个文件中，再借助jhat命令查看详细内容
6.在内存出现泄露、溢出或者其它前提条件下，建议多dump几次内存，把内存文件进行编号归档，便于后续内存整理分析。

```shell
jmap -dump:format=b,file=test.bin 1018
```



## jstat 统计

jstat(JVM Statistics Monitoring Tool)是用于监控虚拟机各种运行状态信息的命令行工具。他可以显示本地或远程虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据，在没有GUI图形的服务器上，它是运行期定位虚拟机性能问题的首选工具。

```shell
#查看类文件数量
➜  ~ jstat -class 31851
Loaded  Bytes  Unloaded  Bytes     Time
  1721  3256.1        0     0.0       0.53

#显示VM实时编译的数量信息
➜  ~ jstat -compiler 31851
Compiled Failed Invalid   Time   FailedType FailedMethod
     633      0       0     0.78          0
  
 #显示 gc信息，查看gc次数和时间
➜  ~ jstat -gc 31851
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
10752.0 10752.0  0.0    0.0   65536.0  15730.2   175104.0    2761.6   9600.0 9043.9 1152.0 980.1       1    0.005   1      0.014    0.019
```

`S0C` 年轻代中第一个survivor（幸存区）的容量 (字节)

 `S1C` 年轻代中第二个survivor（幸存区）的容量 (字节) 

`S0U` 年轻代中第一个survivor（幸存区）目前已使用空间 (字节) 

`S1U` 年轻代中第二个survivor（幸存区）目前已使用空间 (字节)

 `EC` 年轻代中Eden（伊甸园）的容量 (字节)

 `EU` 年轻代中Eden（伊甸园）目前已使用空间 (字节) 

`OC` Old代的容量 (字节)

 `OU` Old代目前已使用空间 (字节)

 `PC` Perm(持久代)的容量 (字节)

 `PU` Perm(持久代)目前已使用空间 (字节)

 `YGC` 从应用程序启动到采样时年轻代中gc次数

 `YGCT` 从应用程序启动到采样时年轻代中gc所用时间(s)

 `FGC` 从应用程序启动到采样时old代(全gc)gc次数

 `FGCT` 从应用程序启动到采样时old代(全gc)gc所用时间(s) 

`GCT` 从应用程序启动到采样时gc用的总时间(s)



```shell
#显示vm三代中对象的使用和占用大小
➜  ~ jstat -gccapacity 31851
 NGCMN    NGCMX     NGC     S0C   S1C       EC      OGCMN      OGCMX       OGC         OC       MCMN     MCMX      MC     CCSMN    CCSMX     CCSC    YGC    FGC
 87040.0 238592.0  87040.0 10752.0 10752.0  65536.0   175104.0   478208.0   175104.0   175104.0      0.0 1058816.0   9600.0      0.0 1048576.0   1152.0      1     1
```

`NGCMN` 年轻代(young)中初始化(最小)的大小(字节)

 `NGCMX` 年轻代(young)的最大容量 (字节)

 `NGC` 年轻代(young)中当前的容量 (字节)

 `S0C` 年轻代中第一个survivor（幸存区）的容量 (字节)

 `S1C` 年轻代中第二个survivor（幸存区）的容量 (字节) `EC` 年轻代中Eden（伊甸园）的容量 (字节) `OGCMN` old代中初始化(最小)的大小 (字节) `OGCMX` old代的最大容量(字节) `OGC` old代当前新生成的容量 (字节) `OC` Old代的容量 (字节) `PGCMN` perm代中初始化(最小)的大小 (字节) `PGCMX` perm代的最大容量 (字节)
`PGC` perm代当前新生成的容量 (字节) `PC` Perm(持久代)的容量 (字节) `YGC` 从应用程序启动到采样时年轻代中gc次数 `FGC` 从应用程序启动到采样时old代(全gc)gc次数



```shell
#gc统计信息
➜  ~ jstat -gcutil 31851
  S0     S1     E      O      M     CCS    YGC     YGCT    FGC    FGCT     GCT
  0.00   0.00  24.00   1.58  94.21  85.08      1    0.005     1    0.014    0.019

#年轻代对象统计信息 gcnewcapacity
  ➜  ~ jstat -gcnew 31851
 S0C    S1C    S0U    S1U   TT MTT  DSS      EC       EU     YGC     YGCT
10752.0 10752.0    0.0    0.0  7  15 10752.0  65536.0  15730.2      1    0.005

#old代对象统计信息 gcoldcapacity
➜  ~ jstat -gcold 31851
   MC       MU      CCSC     CCSU       OC          OU       YGC    FGC    FGCT     GCT
  9600.0   9043.9   1152.0    980.1    175104.0      2761.6      1     1    0.014    0.019
  
```



```shell
#当前vm执行的信息
➜  ~ jstat -printcompilation 31851
Compiled  Size  Type Method
     634    586    1 java/io/File exists
```

`Compiled` 编译任务的数目 `Size` 方法生成的字节码的大小 `Type` 编译类型 `Method` 类名和方法名用来标识编译的方法。类名使用/做为一个命名空间分隔符。方法名是给定类中的方法。上述格式是由-XX:+PrintComplation选项进行设置的

## jhat查看dump文件

分析java堆栈情况的命令

启动一个web浏览器，查看dump文件。

支持 **QQL 对象查询语言**。



## jinfo查看配置信息

```shell
➜  ~ jinfo 31852
Attaching to process ID 31852, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.141-b15
Java System Properties:

java.vendor = Oracle Corporation
preload.project.path = /Users/borgxiao/Documents/work/webjava/crm-services
sun.java.launcher = SUN_STANDARD
idea.config.path = /Users/borgxiao/Library/Preferences/IntelliJIdea2019.3
sun.management.compiler = HotSpot 64-Bit Tiered Compilers
sun.nio.ch.bugLevel =
idea.paths.selector = IntelliJIdea2019.3
```

## javap反编译

javap对class文件进行反编译，因为有很多成熟的反编译工具可以使用，比如jad。但是，javap还可以查看java编译器为我们生成的字节码。通过它，可以对照源代码和字节码，从而了解很多编译器内部的工作

```java
➜  test javap Color.class
Compiled from "Color.java"
public final class me.bigbig.test.Color extends java.lang.Enum<me.bigbig.test.Color> {
  public static final me.bigbig.test.Color RED;
  public static final me.bigbig.test.Color YELLOW;
  public static me.bigbig.test.Color[] values();
  public static me.bigbig.test.Color valueOf(java.lang.String);
  public java.lang.String getName();
  static {};
}

```

上面可以看出java编译后的Enum类是什么样子的。

```java
➜  test javap -c Color.class
Compiled from "Color.java"
public final class me.bigbig.test.Color extends java.lang.Enum<me.bigbig.test.Color> {
  public static final me.bigbig.test.Color RED;

  public static final me.bigbig.test.Color YELLOW;

  public static me.bigbig.test.Color[] values();
    Code:
       0: getstatic     #1                  // Field $VALUES:[Lme/bigbig/test/Color;
       3: invokevirtual #2                  // Method "[Lme/bigbig/test/Color;".clone:()Ljava/lang/Object;
       6: checkcast     #3                  // class "[Lme/bigbig/test/Color;"
       9: areturn

  public static me.bigbig.test.Color valueOf(java.lang.String);

```

注：添加-c参数 jvm指令

```shell
-help 帮助
-l 输出行和变量的表
-public 只输出public方法和域
-protected 只输出public和protected类和成员
-package 只输出包，public和protected类和成员，这是默认的
-p -private 输出所有类和成员
-s 输出内部类型签名
-c 输出分解后的代码，例如，类中每一个方法内，包含java字节码的指令，
-verbose 输出栈大小，方法参数的个数
-constants 输出静态final常量
```

### jad反编译

非java常规工具，已经很久不更新。

### CFR反编译

```shell
java -jar cfr_0_125.jar switchDemoString.class --decodestringswitch false
```

## 常用工具：

### jconsole:

JConsole工具是JDK自带的可视化监控工具。查看java应用程序的运行概况、监控堆信息、永久区使用 情况、类加载情况等。

### jvisualvm：

**监控本地Java进程：**

可以监控本地的java进程的CPU，类，线程等 

**监控远端Java进程：**

比如监控远端tomcat，演示部署在阿里云服务器上的tomcat (1)在visualvm中选中“远程”，右击“添加” (2)主机名上写服务器的ip地址，比如31.100.39.63，然后点击“确定” (3)右击该主机“31.100.39.63”，添加“JMX”[也就是通过JMX技术具体监控远端服务器哪个Java进程] (4)要想让服务器上的tomcat被连接，需要改一下 bin/catalina.sh 这个文件

### Arthas：

Arthas 是Alibaba开源的Java诊断工具，采用**命令行交互模式**，是排查jvm相关问题的利器。

### MAT：

Java堆分析器，用于查找内存泄漏 Heap Dump，称为堆转储文件，是Java进程在某个时间内的快照



## 线上问题排查：

https://www.hollischuang.com/archives/1561

### 频繁GC问题或内存溢出问题

一、使用`jps`查看线程ID

二、使用`jstat -gc 3331 250 20` 查看gc情况，一般比较关注PERM区的情况，查看GC的增长情况。

三、使用`jstat -gccause`：额外输出上次GC原因

四、使用`jmap -dump:format=b,file=heapDump 3331`生成堆转储文件

五、使用jhat或者可视化工具（Eclipse Memory Analyzer 、IBM HeapAnalyzer）分析堆情况。

六、结合代码解决内存溢出或泄露问题。

### 死锁问题

一、使用`jps`查看线程ID

二、使用`jstack 3331`：查看线程情况

