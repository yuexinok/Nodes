# SpringBootListener监听器

> 监听web应用中某些对象，信息对象的创建，销毁，增加，修改，删除等动作发生，然后做出相应的响应处理，常用于在线统计人数，系统加载进行信息初始化，统计访问量等。

1）监听对象的创建和销毁，ServletContextListener.

2）监听对象的域中属性的增加和删除，HttpSessionListener.

3）监听绑定到session上的某个对象的状态，如ServletContextAttributeListener,HttpSessionAttributeListener,HttpRequestAttributeListener

即监听器分3大类，**ServletContext**对应(Application)，**HttpSession**对应(session)，**ServletRequest**对应(Request)

```java
//1）-->在启动类添加@ServletComponentScan

//2) -->编写监听器
@WebListener
public class UserListener implements ServletContextListener {
    
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println(servletContextEvent);
        System.out.println("------>Application初始化");
    }
    
    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println(servletContextEvent);
        System.out.println("------>Application销毁");
    }
}

```

#### HttpSessionListener

```java
@WebListener
public class UserListener implements HttpSessionListener {
    
    @Override
    public void sessionCreated(HttpSessionEvent httpSessionEvent) {
    
    }
    
    @Override
    public void sessionDestroyed(HttpSessionEvent httpSessionEvent) {
    
    }
}
```

#### ServletRequestListener

```java
@WebListener
public class UserListener implements ServletRequestListener {
    
    @Override
    public void requestDestroyed(ServletRequestEvent servletRequestEvent) {
    
    }
    
    @Override
    public void requestInitialized(ServletRequestEvent servletRequestEvent) {
    
    }
}
```