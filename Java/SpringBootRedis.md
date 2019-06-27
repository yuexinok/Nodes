# SpringBootRedis

## 1，基本使用

### 1.1）引入依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

### 1.2）添加配置

```ini
#redis配置
spring.redis.database=0
spring.redis.host=127.0.0.1
spring.redis.port=6379
spring.redis.password=
```

### 1.3）直接使用

```java
@Resource
private RedisTemplate redisTemplate;
@Resource
private StringRedisTemplate stringRedisTemplate;

@Test
    public void testRedis() {
        redisTemplate.opsForValue().set("name","xiaoxiao");
        String name = (String)redisTemplate.opsForValue().get("name");
        System.out.println(name);
        
        stringRedisTemplate.opsForValue().set("name","Borg");
        String name1 = stringRedisTemplate.opsForValue().get("name");
        System.out.println(name1);
    }
```

### 1.4）封装工具类

### 1.5）缓存使用Listener

> 在应用初始化的时候把数据加载到redis中，如一些配置数据

