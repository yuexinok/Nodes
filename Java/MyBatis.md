# MyBatis学习笔记

[^深入浅出MyBatis技术原理]: 杨开振 2016 电子工业出版社

## 历史：

1，**JDBC**：Java Data Base Connectivity

2，**ORM**：Object Relational Mapping ,POJO(Plain Ordinary Java Object)

3，**Hibernate**：全表映射，无法根据不同的条件组装SQL,夺标和复杂SQL查询支持较差，不能有效支持存储过程等。

4，**iBatis**：DAO(Data Access Objects)

5，**MyBatis**：



## 配置：

### configuration

总配置路径

### properties和property

1，property子元素方式：

```xml
<properties>
  <property name="driver" value="com.mysql.jdbc.Driver"/>
</properties>
```

配置的属性即可在配置文件中**${driver}**使用

2，参照spring模式 配置properties文件：

```xml
<properties resource="jdbc.properties" />
```



3，程序方式导入：

```java
	proStream = Resource.getResourceAsStream("jdbc.properties");
  proReader = new InputStreamReader(proStream);
  //上面可以不用
  properties = new Properties();
  properties.load(proReader);
  //特殊用途，如账号密码是加密的，通过程序重新解密连接，防止泄露密码
  properties.setProperty("username",decode(properties.getProperty("username")))
```

### settings 和 setting

```xml
<settings>
	<setting name="cacheEnabled" value="true" />
</settings>
```

#### 参数表：

| 参数名       | 描叙                             | 有效值               | 默认值 |
| :----------- | -------------------------------- | -------------------- | ------ |
| cacheEnabled | 设置所以映射器中配置缓存全局开关 | true/false           | true   |
| logPrefix    | 日志名称前缀                     |                      |        |
| logImpl      | 日志具体实现                     | SLF4J,LOG4J,LOGFJ2等 |        |

### TypeAliases别名

注系统的自带的_byte —>byte ，byte->Byte ,__byte[] —>byte[]，byte[] —> Byte[]

自定义别名：

```xml
<typeAliases>
	<typeAlias alias="role" type="com.borgxiao.po.Role" />
  <!-- 自动扫描 -->
  <package name="com.borgxiao.po" />
</typeAliases>
```

注默认都会扫描，首字母小写。也可以用`@Alias("role")` 手动指定

### TypeHeaderler类型处理

可以自定义类型转换，也可以用系统自带的，即主要实现DB类型到java的转换，java类型到DB类型的转换。

### Plugins插件

TODO

### enviroments配置环境

可以根据不同的环境设置配置不同的连接，不推荐，就可以通过系统的统一配置搞定，这里没必要再区分。

#### 事务管理机制：

JDBC，MANAGED(JNDI方式，Java Naming and Directory Interface,Java命名和目录接口),自定义

#### db连接池：

UNPOOLED，非连接池数据库；POOLED，连接池数据库；JNDI，JNDI数据源

### Mappers和Mapper

#### 文件引入方式：

```xml
<mappers>
	<mapper resource="com/borgxiao/mapper/roleMapper.xml" />
</mappers>
```

#### 包名引入方式：

```xml
<mappers>
	<packeage name="com.borgxiao.mapper" />
</mappers>
```

#### 类注册引入方式：

```xml
<mappers>
	<mapper class="com.borgxiao.mapper.RoleMapper" />
</mappers>
```

#### 文件引入:

```xml
<mappers>
	<mapper url="file://var/mappers/com/borgxiao/mapper/roleMapper.xml" />
</mappers>
```



## 映射器

