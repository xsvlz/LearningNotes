# Mybatis笔记

> [笔记参考地址](http://mp.weixin.qq.com/mp/homepage?__biz=Mzg2NTAzMTExNg==&hid=3&sn=456dc4d66f0726730757e319ffdaa23e&scene=18#wechat_redirect)

- MyBatis 是一款优秀的持久层框架

- MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集的过程

- MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 实体类 【Plain Old Java Objects,普通的 Java对象】映射成数据库中的记录。

- MyBatis 本是apache的一个开源项目ibatis, 2010年这个项目由apache 迁移到了google code，并且改名为MyBatis 。

- 2013年11月迁移到Github .

- [Mybatis官方中文文档](http://www.mybatis.org/mybatis-3/zh/index.html)

- [GitHub项目地址](https://github.com/mybatis/mybatis-3)

## Mybatis第一个程序

**思路：搭建环境-->导入Mybatis--->编写代码--->测试**

1. 搭建实验数据库
2. 导入Mybatis相关jar包

```xml
<dependency>
   <groupId>org.mybatis</groupId>
   <artifactId>mybatis</artifactId>
   <version>3.5.2</version>
</dependency>
```
3. 编写mybatis核心配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
       PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
   <environments default="development">
       <environment id="development">
           <transactionManager type="JDBC"/>
           <dataSource type="POOLED">
               <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
               <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
               <property name="username" value="root"/>
               <property name="password" value="root"/>
           </dataSource>
       </environment>
   </environments>
   <mappers>
       <mapper resource="com/kuang/dao/userMapper.xml"/>
   </mappers>
</configuration>
```

4. 编写mybatis工具类(固定内容)

- 查看帮助文档

```java
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import java.io.IOException;
import java.io.InputStream;

public class MybatisUtils {

   private static SqlSessionFactory sqlSessionFactory;

   static {
       try {
           String resource = "mybatis-config.xml";
           InputStream inputStream = Resources.getResourceAsStream(resource);
           sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
      } catch (IOException e) {
           e.printStackTrace();
      }
  }

   //获取SqlSession连接
   public static SqlSession getSession(){
       return sqlSessionFactory.openSession();
  }

}
```

5. 创建实体类(和数据库表对应)

```java
public class User {
   
   private int id;  //id
   private String name;   //姓名
   private String pwd;   //密码
   
   //构造,有参,无参
   //set/get
   //toString()
   
}
```

6. 编写Mapper接口类

```java
import top.longzhuang.pojo.User;
import java.util.List;

public interface UserMapper {
   List<User> selectUser();
}
```

7. 编写mapper.xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
       PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
       "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace要写全类名-->
<mapper namespace="top.longzhuang.dao.UserMapper">
 <select id="selectUser" resultType="top.longzhuang.pojo.User">
  select * from user
 </select>
</mapper>
```

8. 编写测试类

```java
@Test
public void test() {
    // 1. 获取SqlSession对象
    SqlSession sqlSession = MybatisUtils.getSqlSession();
    // 2. 执行
    // 方式一：getMapper:
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    List<User> userList = mapper.getUserList();

    // 方式二(不需要掌握):
    List<User> userList1 = sqlSession.selectList("top.longzhuang.dao.UserMapper.getUserList");

    userList1.forEach(System.out::println);
    userList.forEach(System.out::println);

    sqlSession.close();
}
```

## CURD及配置解析

### 编写CURD步骤：

1. 在Mapper接口定义抽象方法

```java
public interface UserMapper {
   //查询全部用户
   List<User> selectUser();
   //根据id查询用户
   User selectUserById(int id);
}
```

2. 在对应的xml中编写SQL语句

- resultType: 返回数据类型
- parameterType: 传入数据类型

```xml
<select id="getUserById" parameterType="int" resultType="top.longzhuang.pojo.User">
    select * from mybatis.user where id = #{id};
</select>
```
- 增删改查的标签不同:
    - select
    - update
    - insert
    - delete

### 配置解析

#### 核心配置文件`mybatis-config.xml`

- 包含以下标签:
- 必须按照顺序配置

```xml
configuration（配置）
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```

> environments - environment

- `transactionManager`: 事务管理器
    - JDBC 
    - MANAGED （这个配置几乎没做什么）
- `dataSource`: 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源(必须配置)
    - 三种类型:
        - unpooled：这个数据源的实现只是每次被请求时打开和关闭连接
        - pooled：这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来 , 这是一种使得并发 Web 应用快速响应请求的流行处理方式
        - jndi：这个数据源的实现是为了能在如 Spring 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。

> mappers - mapper

映射器：告诉Mybatis去哪里找到SQL语句

四种方式:

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
<!-- 使用完全限定资源定位符（URL） 
    (几乎不用)
-->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

Mapper文件:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace绑定接口-->
<mapper namespace="top.longzhuang.dao.UserMapper">
    <!--  查询语句  -->
    <select id="getUserList" resultType="user">
        select * from mybatis.user;
    </select>
    <!--更多语句...-->
</mapper>
```

> 其他配置
- [settings](https://mybatis.org/mybatis-3/zh/configuration.html#settings)

    - 懒加载 lazyLoadingEnabled
    - 日志实现 logImpl:
    ```
    SLF4J | LOG4J | LOG4J2 | JDK_LOGGING | COMMONS_LOGGING | STDOUT_LOGGING | NO_LOGGING
    ```
    - 缓存开启/关闭 cacheEnabled

```xml
<settings>
 <setting name="cacheEnabled" value="true"/>
 <setting name="lazyLoadingEnabled" value="true"/>
 <setting name="multipleResultSetsEnabled" value="true"/>
 <setting name="useColumnLabel" value="true"/>
 <setting name="useGeneratedKeys" value="false"/>
 <setting name="autoMappingBehavior" value="PARTIAL"/>
 <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
 <setting name="defaultExecutorType" value="SIMPLE"/>
 <setting name="defaultStatementTimeout" value="25"/>
 <setting name="defaultFetchSize" value="100"/>
 <setting name="safeRowBoundsEnabled" value="false"/>
 <setting name="mapUnderscoreToCamelCase" value="false"/>
 <setting name="localCacheScope" value="SESSION"/>
 <setting name="jdbcTypeForNull" value="OTHER"/>
 <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

## 作用域和生命周期

[官方文档（最下面）](https://mybatis.org/mybatis-3/zh/getting-started.html)

![](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7JdnS939HH5TayIhQo5s0aJbReBExSQO1U23XeLAXlhTWUeL87mJZL0lDzPstpY3CSIwvW0dN9ccA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

