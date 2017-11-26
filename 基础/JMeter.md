## 简介：

Apache JMeter是Apache组织开发的基于Java的压力测试工具。用于对软件做压力测试，它最初被设计用于Web应用测试，但后来扩展到其他测试领域。 它可以用于测试静态和动态资源，例如静态文件、Java [小服务程序](https://baike.baidu.com/item/%E5%B0%8F%E6%9C%8D%E5%8A%A1%E7%A8%8B%E5%BA%8F)、CGI 脚本、Java 对象、数据库、FTP 服务器， 等等。JMeter 可以用于对服务器、网络或对象模拟巨大的负载，来自不同压力类别下测试它们的强度和分析整体性能。另外，JMeter能够对应用程序做功能/[回归测试](https://baike.baidu.com/item/%E5%9B%9E%E5%BD%92%E6%B5%8B%E8%AF%95)，通过创建带有断言的脚本来验证你的程序返回了你期望的结果。为了最大限度的灵活性，JMeter允许[使用正则表达式](https://baike.baidu.com/item/%E4%BD%BF%E7%94%A8%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)创建断言。

Apache jmeter 可以用于对静态的和动态的资源（文件，Servlet，Perl脚本，java 对象，数据库和查询，[FTP服务器](https://baike.baidu.com/item/FTP%E6%9C%8D%E5%8A%A1%E5%99%A8)等等）的性能进行测试。它可以用于对服务器、网络或对象模拟繁重的负载来测试它们的强度或分析不同压力类型下的整体性能。你可以使用它做性能的图形分析或在大并发[负载测试](https://baike.baidu.com/item/%E8%B4%9F%E8%BD%BD%E6%B5%8B%E8%AF%95)你的服务器/脚本/对象。



源自：慕课网，讲师：大周



## jMeter组成：

取样器：进行脚本逻辑控制

线程组：场景设置

监视器：监视我们的脚本运行，取得性能指标。

类似软件：loadrunner

## 录制工具：

### 1）Badboy录制：

即在浏览器上点击，生成访问，录制成脚本，导出到jmeter. xx.jmx

Bugfree:bug管理系统。

登录的时候用：跟随重定向。

### 2）使用代理方式进行录制：

启动代理，加正则过滤。

## 用户自定义变量

业务流程->录制工具->脚本控制->性能测试

配置元件：用户定义变量，${变量名}

## 函数助手：

选项->函数助手，

CSVRead：${_CSVRead(….,0)}



配置元件：CSV data set comnfig。header: title,age  一一对应

## 关联：

上下文，即使用上一个请求的结果。

后置处理器：正则取样器



