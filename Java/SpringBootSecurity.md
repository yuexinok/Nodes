# SpringBootSecurity

## 1，简单内存用法

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

```java
@Configuration
//开启
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    public void configure(WebSecurity web) throws Exception {
        //解决静态资源被SpringSecurity拦截的问题
        web.ignoring().antMatchers("/static/**");//这样我的webapp下static里的静态资源就不会被拦截了，也就不会导致我的网页的css全部失效了……
    }
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().mvcMatchers("/api").permitAll().anyRequest().authenticated().and().formLogin().failureUrl("/login?error")
                .defaultSuccessUrl("/user/list").permitAll();
        super.configure(http);
    }
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().
                //不支持
                withUser("xiao").password("123456").roles("USER")
                .and().passwordEncoder(new MyPasswordEncoder()).withUser("admin").password("123456").roles("ADMIN");
        //super.configure(auth);
    }
    
    /**
     * 明文匹配
     */
    class MyPasswordEncoder implements PasswordEncoder{
    
        @Override
        public String encode(CharSequence charSequence) {
            return charSequence.toString();
        }
    
        @Override
        public boolean matches(CharSequence charSequence, String s) {
            return s.equals(charSequence.toString());
        }
    }
}

```

解决不支持明文密码的问题：https://blog.csdn.net/weixin_39220472/article/details/80865411

