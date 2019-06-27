# SpringBootFilter过滤器

> 对静态资源进行拦截，对特殊URL进行权限访问控制，过滤敏感词，压缩响应信息等

## 1,使用注解的方式

```java
public interface Filter {
    //web服务启动的时候创建 只执行一次
    void init(FilterConfig var1) throws ServletException;
		//当对应请求过来的时候调用，可以执行n次
    void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;

  	//WEB容器卸载和停止时调用，只执行一次
    void destroy();
}
```

**在启动类上面**：

```javascript
//开启后，则就可以使用WebFilter和WebSerlet和@WebListener
@ServletComponentScan
```

```java
@WebFilter(filterName = "userFilter",urlPatterns = "/*")
public class UserFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        //ApplicationFilterConfig[name=userFilter, filterClass=com.learnjava.demo.filter.UserFilter]
        System.out.println(filterConfig);
        System.out.println("------->>> init");
    }
    
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println(servletRequest.getLocalName());
        System.out.println("------->>doFilter");
        filterChain.doFilter(servletRequest,servletResponse);
    }
    
    @Override
    public void destroy() {
        System.out.println("-------->>>destroy");
    }
}
```



## 2，使用代码注册方式

即实现FilterRegistationBean

```java
@Bean
    public FilterRegistrationBean druidStatFilter(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(new WebStatFilter());
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.addInitParameter("exclusions","*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
        return filterRegistrationBean;
    }
```

