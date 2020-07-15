# JVM学习随笔

### 双亲委派模型的破坏(以JDBC为例)：

在使用JDBC时，如果我们要加载mysql驱动，在最开始我们要写上：
```java
Class.forName("com.mysql.jdbc.Driver");
DriverManager.getConnection("..."); // 数据库连接url
```

这行代码的作用是触发`com.mysql.jdbc.Driver`类的加载，在该类的静态代码块中将Driver注册到了`DriverManager`中。
但实际上即使我们不使用该行代码，当我们获取链接的时候发现也可以获取到。

`DriverManager`位于`java.sql`包下，应该由启动类加载器进行加载，那么如果按照双亲委派模型来看，是不能使用下级的应用程序类加载器去加载`com.mysql.jdbc.Driver`类的。但是在`DriverManager`类中，可以获取线程上下文类加载器（`ThreadContextClassLoader`），并通过SPI的方式加载到mysql驱动类（在驱动包的META-INF/services/java.sql.Driver文件中指定驱动类就可以了）。

而线程上下文类加载器是线程启动时获得的，它默认为应用程序类加载器。因此通过DriverManager获得了应用程序类加载器，就起到了上级类加载器去请求下级类加载器加载驱动的作用。这就达到了破坏双亲委派模型的效果。