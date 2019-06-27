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

### select

```xml
<select id="getListCommentByTypeId" resultType="me.bigbig.domain.Comment">
        select * from comment
          WHERE type_id = #{typeId}  AND type = #{type}
        <if test="isReplay">
           AND replay_id > 0
        </if>
        <if test="!isReplay">
         AND replay_id = 0
        </if>
          order by create_time desc
        <if test="limit>0">
            limit #{start},#{limit}
        </if>
    </select>

    <!-- 获取评论列表根据-->
    <select id="getListCommentNumByTypeId" resultType="int">
        select count(*) from comment
        WHERE replay_id = 0 AND type_id = #{typeId}  AND type = #{type}
    </select>
```

```java

int upNextChapterNumberByCourseId(@Param("courseId") long courseId,@Param("number") int number,@Param("isDel") int isDel);
```

*参数传递：直接定义在接口里面，超过5个也可以定义一个POJO直接传递

*返回结果：直接映射到POJO，特殊情况直接返回单个参数int或者其他

### insert：

没有什么特殊的，注意主键的获取即可：可以映射到对应的POJO对象的自定对象，keyColumn不是必要的

```xml
<insert id="insert" useGeneratedKeys="true" keyProperty="group.id" keyColumn="f_group_id">
   
</insert>
```



### update：

```xml
<update id="batchUpdateSort">
        UPDATE xxx
        SET f_user_id=#{userId}, f_sort =
        <foreach collection="groups" item="group" separator=" " open="CASE f_id" close="END">
            WHEN #{group.id} THEN #{group.sort}
        </foreach>
        WHERE f_corp_id = #{corpId} AND f_id IN
        <foreach collection="groups" item="group" separator="," open="(" close=")">
            #{group.id}
        </foreach>
    </update>
```

### delete：



### 参数：

$和#的区别，一般情况安全情况使用#。



### Sql:

```xml
<sql id ="columsAs" >
  f_id AS id,f_name AS name
</sql>
<select>
	select <include refid="columsAs" /> from xxx
</select>
```

即一些公共的语句可以统一定义一个段。

## 动态SQL

### if test判断

```xml
<if test="roleName != null and roleName != ''"></if>
```

### Choose,when,otherwise 选择

```xml
<choose>
	<when test="">
  </when>
  <when test="">
  </when>
  <otherwise>
  </otherwise>
</choose>
```

### where语句

假如用普通的if活着choose多条件下，为了正常通过需要在where条件添加1=1这样的附加条件，但是用where可以避免,where可以理解为 select的一个条件，并会自动添加where关键字

```xml
<select>
  select * from tab
  <where>
  		<if test="">
    		and role_name="xxx"
    	</if>
  </where>
</select>
```



### trim语句

过滤和添加前缀等

```xml
<trim prefix="set" suffixOverrides="," prefixOverrides="and">..</trim>
```

prefix添加前缀，suffixOverrides去掉后面默认, prefixOverrides去掉前面的and

### foreach语句

```xml
sex in
<foreach item="sex" index="index" collection="sexList" open="(" sparator="," close=")">
 #{sex}
</foreach>

```

open和close配合是

## 运行原理

### 构建SqlSessionFactory

1，构建Configuration：全局参数，设置，别名，类型处理器，对象，插件，环境，数据库标识，映射器

2，映射器：MappedStatement(保存一个映射器的一个节点select,insert,delete,update),SqlSource(提供BoudSql对象的地方，是MappedStatement的一个属性)，BoudSql(SQL,parameterObject,parameterMappings)

3，构建SqlSessionFactory

### SqlSession运行原理

> SqlSession是一个接口，构建SqlSessionFactory的时候获取，SqlSession提供查询，插入，更新，删除等方法。Executor执行器，调度StatementHanlder，ParameterHanlder，ResultHandler，其中执行器defaultExecutorType=SIMPLE(简易执行器)，REUSE(重用预处理语句) BATCH(重用语句+批量更新)

StatementHanlder->RoutingStatementHanlder(路由到3个执行器)->(SimpleStatementHanlder,PrepareStatementHanlder,CallableStatementHanlder)分别对应3个执行器

1，Executor—> Executor调用StatementHanlder(数据库会话器)的方法

2， prepare方法  **StatementHanlder**执行prepare进行预编译

3，paramerize **StatementHanlder**执行paramerize 方法设置参数

​      3.1 setParameters **ParameterHanlder**执行setParameters设置参数

​      3.2 typeHandler **typeHandler**提供参数设置和结果类型转换规则

4，update/query **StatementHanlder**执行update或query

5，handleResult **ResultHandler**的handleResult方法封装结果 —> 需要结果参数解析 typeHandler->setParameters

6，objectFactory **ObjectFactoruy**提供结果对象的生成规则

## 插件

```java
@ImportResource({"classpath:mybatis.xml"})
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <plugins>
        <plugin interceptor="com.learnjava.demo.intercepts.MyBatisPlugin" />
    </plugins>
</configuration>
```

```java
@Intercepts({@Signature(
        type = Executor.class,//需要拦截的对象
        method = "update",//需要拦截的方法
        args = {MappedStatement.class, Object.class} //拦截的方法的参数 这里需要的是可以确定唯一性
)})
public class MyBatisPlugin implements Interceptor {
    
    Properties properties = null;
    
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        System.out.println("开始拦截.....");
        System.out.println(invocation.getMethod());
        System.out.println(invocation.getArgs());
        Object obj = invocation.proceed();
        System.out.println("拦截结束");
        return obj;
    }
    
    @Override
    public Object plugin(Object target) {
        System.out.println("调用生成代理对象.....");
        if (target instanceof Executor) {
            return Plugin.wrap(target, this);
        }
        return target;
    }
    
    @Override
    public void setProperties(Properties properties) {
        System.out.println("初始化获取参数........");
        System.out.println(properties.getProperty("dbType"));
        this.properties = properties;
    }
}
```

注：并未生效