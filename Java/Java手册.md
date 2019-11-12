# Java手册

## 1）POJO

Plain(清楚的) Ordinary(普通) Java Object

| DO(Domain Object)         |      |
| ------------------------- | ---- |
| BO(Business Object)       |      |
| DTO(Data Transfer Object) |      |
| VO(View Object)           |      |
| AO(Application Object)    |      |
|                           |      |

注：POJO的类属性必须用包装数据类型，即Integer，RPC方法的返回值必须使用包装类型。所以的局部变量，推荐是基本数据类型。

注：包装类型缓存区间，-128-127，其中Character为0-127


## 2）覆写和重载

### **向上转型：**

1）无法调用到子类存在，父类不存在的方法。

2）可以调用到子类中覆写的父类方法。这是一种多态。

### **覆写要点：**

1）访问权限不能变小。

2）返回类型能够向上转型为父类的返回类型。

3）异常也能向上转型为父类异常。

4）方法名，参数类型及个数必须严格一致。

### 重载：

即方法名称相同，参数不一样。

优先级：

1）精确匹配。

2）如果是基本数据类型，自动转换成更大表示范围的基本类型。

3）通过自动拆箱与装箱。

4）通过子类向上转型继承路线依次匹配。

5）通过可变参数匹配

```java
class OverLoadMethod{
    public void methodForOverload(){
        System.out.println("无参数方法");
    }
    public void methodForOverload(int param){
        System.out.println("int");
    }
    public void methodForOverload(Integer param){
        System.out.println("Integer");
    }
    public void methodForOverload(Integer... params){
        System.out.println("Integer...");
    }
    public void methodForOverload(Object param){
        System.out.println("object");
    }
    
    
}
public class OverloadTest {
    
    public static void main(String[] args) {
        OverLoadMethod overLoadMethod = new OverLoadMethod();
        //int
        overLoadMethod.methodForOverload(7);
        //object
        overLoadMethod.methodForOverload(7L);
        //无参数方法
        overLoadMethod.methodForOverload();
        Integer[] ss = {7};
        //Integer...
        overLoadMethod.methodForOverload(ss);
    }
}
```

注：尽量少用覆写。

## UML图：

![](./images/object.png)

![](./images/base.png)

## 4）异常：

![](./images/exception.png)

### 非受检异常：

运行时的异常，继承RuntimeException,不需要程序进行显示的补捉。

1）可预测异常，IndexOutOfBoundException,NullPointerExption可以提前做好边界检查。

2）需捕获的异常：如Dubbo的DubboTimeOutExption.

3）可透出异常：指框架自己会处理的异常，而程序无需关心，如Spring的NoSuchRequestHandlingExption

### 受检异常：

即需要代码捕获和处理的异常，否则编译报错。如SQLExption,ClassNotFoundExption.

### Try代码块：

try-catch-finally 是处理程序异常的三部曲。当存在try 时，可以只有catch 代码块，
也可以只有finally 代码块，就是不能单独只有try 这个光杆司令。

契约式编程理念就完全处于防御式编程理念的下风，
所以我们推荐方法的返回值可以为null ，不强制返回空集合或者空对象等，但是必须
添加注释充分说明什么情况下会返回null 值。防止NP E 定是调用方的责任，需要
调用方进行事先判断。

注：避免在finally里面return.

## 5）日志：

1）应用中的扩展日志命名方式应该有统－的约定，通过命名能直观明了地表明当前日志文件是什么功能，如监控、访问日志等。推荐的日志文件命名方式为appName_logType logName.log 。其中，log Type 为
日志类型，推荐分类有stats 、monitor 、visit 等，logNam e 为日志描述。这种命名的好处是通过文件名就可以知道曰志文件属于什么应用，什么类型，什么目的，也有利于归类查找。例如，mppserv er 应用中单独监控时区转换异常的日志文件名定义为mppserver monitor timeZoneConvert.log 。

2）代码规约推荐曰志文件至少保存1 5 天，可以根据日志文件的重要程度、文件大小及磁盘空间再自行延长保存时间。

3）预先判断曰志级别，避免无效日志打印，区别对待错误日志，保证记录内容完整

### 日志框架：

日志门面
门面设计模式是面向对象设计模式中的一种，日志框架采用的就是这种模式，类
似JDB C 的设计理念。它只提供一套接口规范，自身不负责日志功能的实现，目的是
让使用者不需要关注底层具体是哪个日志库来负责日志打印及具体的使用细节等。目
前用得最为广泛的曰志门面有两种slf4j 和commons -logging 。
日志库
它具体实现了日志的相关功能，主流的日志库有三个，分别是log4j 、log -jdk 、
logback 。最早Java 要想记录曰志只能通过System.out 或System.err 来完成，非常不方便。
log4j 就是为了解决这一问题而提出的，它是最早诞生的曰志库。接着JD K 也在1 .4 版
本引入了一个日志库java. util.logging. Logger.，简称log-dk。这样市面上就出现两种日志
功能的实现，开发者在使用时需要关注所使用的日志库的具体细节。logback 是最晚出
现的，它与log4j 出自同一个作者，是log4j的升级版且本身就实现了slf4j的接口。
日志适配器
曰志适配器分两种场景
( I ）日志门面适配器，因为slf4j规范是后来提出的此之前的日志库是没有
实现slf4j的接口的，例如log4j ；所以在工程里要想使用slf4j +log4j 的模式，就额
外需要个适配器（slf4j-log4j12 来解决接口不兼容的问题。
( 2 ）日志库适配器，在一些老的工程里，一开始为了开发简单而直接使用了日志库API来完成曰志打印，随着时间的推移想将原来直接调用日志库的模式改为业界标准的门面模式（例如slf4j +logback 组合），但老工程代码里打印曰志的地方太多，难以改动，所以需要个运队器来完成从旧日志库的API 到slf4j 的路由，这样在不改动原有代码的情况l、也能使用slf4j叫来统一管理曰志，而日后续自由替换具体日志库也不成可题。

![](./images/log.png)

## 5）数据结构与集合

集合与泛型的联合使用，可以把泛型的功能发挥到极致，很多程序员不清楚List、List<Object>、List<?>三者的区别，更加不能区分<? extends T>与<? super T>的使用场景。

List<?>是一个泛型，在没有赋值之前，表示它可以接受任何类型的集合赋值，赋值之后就不能再随便往里添加元素了
package Test;

import java.util.List;

public class ListNoGeneric {
    public static void main(String[] args){
        
```java
    //第一段：泛型出现之前的集合定义方式
    List a1 = new ArrayList();
    a1.add(new Object());
    a1.add(new Integer(111));
    a1.add(new String("hello a1a1"));
    
    //第二段：把a1引用赋值给a2，注意a2与a1的区别是增加了泛型限制<Object>
    List<Object> a2 = a1;
    a2.add(new Object());
    a2.add(new Integer(222));
    a2.add(new String("hello a2a2"));
    
    //第三段：把a1引用赋值给a3，注意a3与a1的区别是增加了泛型<Integer>
    List<Integer> a3 = a1;
    a3.add(new Integer(333));
    //以下两行代码编译错误，不允许增加非Integer的元素进入集合
    a3.add(new Object());
    a3.add(new Object());
    
    //第四段：把a1引用赋值给a4,a1与a4的区别是增加了通配符
    List<?> a4 = a1;
    //允许删除和消除元素
    a1.remove(0);
    a4.clear();
    //编译出错，不允许增加任何元素
    a4.add(new Object());
}
```

第一段说明：在定义List之后，毫不犹豫地往集合里装入三种不同的对象，即Object、Integer、String，遍历没有问题。
第二段说明：把a1赋值给a2，a2是List<Object>类型的，因为Object是所有对象的始祖，所以也可以再往里面装入三种不同的对象。我们经常认为List和List<Object>是完全相同的，至少从前两段来看是这样的。
第三段说明：泛型是在JDK5之后才出现的，考虑到向前兼容，因此历史代码有时需要赋值给新泛型代码，从编译器角度来说是允许的。
第四段说明：问号在正则表达式中可以匹配任何字符，List<?>成为通配符集合。它可以接收任何类型的集合引用赋值，不能添加任何元素，但是可以remove和clear，并非immuntable集合。List<?>一般作为参数来接收外部的集合，或者返回一个不知道具体元素类型的集合。

