# SpringBoot异常

## 全局异常

1) 创建自己的业务异常代码

```java
public class BusinessException extends RuntimeException {
    public BusinessException(){
    
    }
    public BusinessException(String message){
        super(message);
    }
}
```

2）创建异常返回实体类

```java
@Data
public class ErrorInfo<T> {
    
    public static final Integer ERROR = 100;
    
    private Integer code;
    
    private String message;
    
    private String url;
    
    private T data;
}
```

3）创建异常捕获配置类

```java
//扫描的包
@ControllerAdvice(basePackages = {"com.learnjava.demo",})
public class GlobalDefaultExceptionHandler {
    
    @ExceptionHandler({BusinessException.class})
    @ResponseBody
    public ErrorInfo defaultErrorHandler(HttpServletRequest request, Exception e) throws Exception {
        ErrorInfo errorInfo = new ErrorInfo();
        errorInfo.setMessage(e.getMessage());
        errorInfo.setUrl(request.getRequestURI());
        errorInfo.setCode(ErrorInfo.ERROR);
        return errorInfo;
    }
}
```

