---
layout: post
title: "Hello, Spring Framework"
date: 2022-08-10
author: ingerchao
category: blog
tag:
- Spring
---



### Introduction

#### Spring 生态系统的复杂度

Spring Framework 是 Spring 整个体系的基础，如果缺少 Spring Framwork 的相关知识，就无法完全理解 Srping Boot、Spring Data 等 Spring 的上层框架。

### Spring Framework

什么是 Spring Framework？简而言之，Spring 框架的核心是一个依赖注入容器，并且在上层添加了一些工具（例如：数据访问、代理、面向切片编程、RPC、web mvc 框架），以助于更加便捷的开发 Java 应用。

### 依赖注入基础

什么是依赖？例如，编写一个 Java 类用来访问数据库中的用户表 `users`，这些类统称为 DAOs （data assess object，数据访问对象）或者 Repositories（存储库）。

```java
public class UserDao {
    public User findById(Integer id) {
        // 执行一个 sql 查询语句来查找 user
    }
}
```

为了执行 sql 查询，UserDao 需要建立数据库连接。通常，Java 通过 DataSource 来建立数据库连接，代码看起来可能是下面这样子的：

```java
import javax.sql.DataSource;

public class UserDao {

    public User findById(Integer id) throws SQLException {
        try (Connection connection = dataSource.getConnection()) { // (1)
               PreparedStatement selectStatement = connection.prepareStatement("select * from users where id =  ?");
               // use the connection etc.
        }
    }

}
```

1. 问题来了，如何获取 dataSource 依赖呢？DAO 显然依赖于有效的 DataSource 来触发这些 SQL 查询

为了让 UserDao 能够通过 dataSource 访问数据库，可以通过 `new()` 实例化一个 DataSource 对象。以 Mysql JDBC 为例，代码如下：

```JAVA
import com.mysql.cj.jdbc.MysqlDataSource;

public class UserDao {
    public User findById(Integer id) {
        MysqlDataSource dataSource = new MysqlDataSource(); // (1)
        dataSource.setURL("jdbc:mysql://anyHost:port/dbName");
        dataSource.setUser("username");
        dataSource.setPassword("pwd");

        try (Connection connection = dataSource.getConnection()) { // (2)
            PreparedStatement selectStatement = connection.prepareStatement("select * from users where id =  ?");
            // use the connection etc.
            return user;
        }
    }
}
```

1. 通过 MysqlDataSource + hardcode 的url、username、password 来建立数据源；
2. 基于 dataSource 建立数据库连接进行查询；

看起来没什么问题，但是每新增一个方法，都要重写一遍：new 一个 MysqlDataSource 并配置 url、username、pwd。这样做不仅造成了代码冗余，并且，每个 DataSource 都可以创建物理上的 java 程序到 mysql socket 连接，代价高昂。

为了实现单一数据源配置，并在项目中复用，可以**通过一个全局的 Application 类来管理依赖**，代码如下：

```JAVA
import com.mysql.cj.jdbc.MysqlDataSource;
import javax.sql.DataSource;

public enum Application {
    INSTANCE;

    private DataSource dataSource;

    public DataSource dataSource() {
        if (dataSource == null) {
            MysqlDataSource dataSource = new MysqlDataSource();
            dataSource.setUser("root");
            dataSource.setPassword("s3cr3t");
            dataSource.setURL("jdbc:mysql://localhost:3306/myDatabase");
            this.dataSource = dataSource;
        }
        return dataSource;
    }
}
```

在 UserDao 中建立连接通过下面的语句：

```JAVA
Connection connection = Application.INSTANCE.dataSource().getConnection()
```

这样做的好处有两个：

1. 数据访问对象无需重新构建 DataSource 依赖，只需要通过 `Application` 类访问；
2. 整个应用程序类是一个单例（只会创建一个实例），并且该单例包含对 DataSource 单例的引用。

但是依然存在缺点：

1. UserDAO 必须主动知道从哪里获取其依赖项，它必须调用 `Application` 类；
2. 如果程序越来越大，并且可能会拥有越来越多的依赖项，Application.java 类用于处理所有依赖项，将会非常臃肿。基于此，自然而然会想到将 Application 拆分成多个类或工厂。

### **控制反转（Inversion of Control ）**

有没有一种可能，UserDAO并不关心从哪里查找依赖，也不需要主动调用 `Application.INSTANCE.dataSource()`，也就是**不再控制从何时/何处/如何获取** DataSource，而仅仅在使用时直接调用 DataSource datasource。这种方式就叫控制反转。

基于控制反转的 UserDAO 代码可能如下：

```JAVA
import javax.sql.DataSource;

public class UserDao {

    private DataSource dataSource;

    public UserDao(DataSource dataSource) { // (1)
        this.dataSource = dataSource;
    }

    public User findById(Integer id) {
        try (Connection connection = dataSource.getConnection()) { // (2)
               PreparedStatement selectStatement = connection.prepareStatement("select * from users where id =  ?");
               // TODO execute the select etc.
        }
    }

    public User findByFirstName(String firstName) {
        try (Connection connection = dataSource.getConnection()) { // (2)
               PreparedStatement selectStatement = connection.prepareStatement("select * from users where first_name =  ?");
               // TODO execute the select etc.
        }
    }
}
```

1. 实例化 UserDao 时必须传入一个有效的 DataSource 对象；
2. UserDao 的方法可以直接调用私有的 dataSource 对象来访问数据库。

从 UserDAO 的角度来看，易读性更好，它并不关心 DataSource 从哪来，如何创建，而是将 DataSource 的创建交给了调用者。因此，调用 UserDAO 时，必须传入 DataSource，be like：

```JAVA
public class MyApplication {
    public static void main(String[] args) {
        UserDao userDao = new UserDao(Application.INSTANCE.dataSource());
        User user1 = userDao.findById(1);
        User user2 = userDao.findById(2);
        // etc ...
    }
}
```

对开发人员而言，问题只是在哪里调用 `Application.INSTANCE.dataSource()` 而已，依然需要主动创建 DataSource。

如果有个 *something* 知道 UserDao 有一个 DataSource 构造函数，并且知道如何构造 DataSouce，从而神奇的构造了两个对象：一个 DataSource 和一个有效的 UserDao。这个 *something* 就是**依赖注入容器**，也是Spring 框架的核心。

### Spring 的依赖注入容器原理

#### ApplicationContext

可以控制所有类并可以适当地管理它们（使用必要的依赖项创建它们）的人在 Spring 世界中称为 ApplicationContext。

基于 ApplicationContext ，为了实现如下的调用代码：

```JAVA
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import javax.sql.DataSource;

public class MyApplication {

    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(someConfigClass); // (1)

        UserDao userDao = ctx.getBean(UserDao.class); // (2)
        User user1 = userDao.findById(1);
        User user2 = userDao.findById(2);

        DataSource dataSource = ctx.getBean(DataSource.class); // (3)
        // etc ...
    }
}
```

1. 构建一个 Spring ApplicationContext；
2. ApplicationContext 提供的配置了 DataSource 的 UserDao;
3. ApplicationContext 也可以直接提供 DataSource 的内容；

基于上述实现，调用者只需要调用 ApplicationContext 就可以完成相应工作。

由 1. 可知，构建 AnnotationConfigApplicationContext 需要一个配置类，代码如下：

```JAVA
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MyApplicationContextConfiguration {  // (1)

    @Bean
    public DataSource dataSource() {  // (2)
        MysqlDataSource dataSource = new MysqlDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("pwd");
        dataSource.setURL("jdbc:mysql://host:port/dbname");
        return dataSource;
    }

    @Bean
    public UserDao userDao() { // (3)
        return new UserDao(dataSource());
    }

}
```

1. 通过 `@Configuration` 注解定义一个配置类；
2. 通过 Spring 特有的 `@Bean` 注解，定义一个返回 DataSource  的方法；
3. 定义一个返回用 dataSource 初始化的 UserDao 对象方法；

通过 Spring 配置类，就可以创建第一个 Spring 应用程序了：

```JAVA
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class MyApplication {

    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(MyApplicationContextConfiguration.class);
        UserDao userDao = ctx.getBean(UserDao.class);
        // User user1 = userDao.findById(1);
        // User user2 = userDao.findById(1);
        DataSource dataSource = ctx.getBean(DataSource.class);
    }
}
```

其中，Spring 中的 ApplicationContext 的构建方式有多种，例如：xml 文件、带 @Configuration 注解的 Java 类、或者其他编程方式实现。如果是通过 xml 文件创建 ApplicationContext，需要通过 `ClassPathXmlApplicationContext` 注解来实现。

#### Bean 

Bean 在 Spring 宇宙里是一个神奇的单词，神奇之处在于，Spring 声明 Bean 就像在说 “我（Spring 容器）创建了你们，你们在我的控制之下。”

`@Bean` 是方法级别的注解，是 xml 文件里的 `<bean/>` 标签的直接模拟（analog）。`@Bean` 可以用在被`@Configuration`和`@Component`标注的类内部。

#### Bean Scope

通过 `@Scope` 注解明确 Bean 的作用域。如果不特殊声明，默认 Bean 的Scope 为 singleton，等同于`@Scope("singleton")` 。Bean Scope 参数和说明见下表：

| Scope              | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| singleton（默认）  | 将单个 bean 定义为每个 Spring IoC 容器的单个对象实例。即，全局有且仅有一个实例。 |
| prototype          | 将单个 bean 定义为任意数量的对象实例。即，每次获取 Bean 时都会创建一个新的实例。 |
| request（web）     | 将单个 bean 定义为单个 http 请求的生命周期，每个 http 请求拥有自己的 bean 实例。 |
| session（web）     | 将单个 bean 定义为单个 http session 的生命周期。             |
| application（web） | 将单个 bean 定义为 ServletContext 的生命周期。               |
| websocket（we b）  | 将单个 bean 定义为 WebSocket 的生命周期。                    |

> 注：(web) 表示该作用域仅在 web 感知的 Spring ApplicationContext 的上下文中有效。

#### Java Config

通过注解的方式，在 ApplicationContext 中显式配置 bean，就是 Spring 中说的 Java Config。相比 Spring 先前基于 xml 配置的方式，方便了许多。

####`@ComponentScan` 

有一个新的问题是，可不可以隐式实例化带有 dataSource 的 UserDao() ，而不是 `UserDao userDao = ctx.getBean(UserDao.class);`？通过 `@ComponentScan` 注解可以解决该问题。

`@ComponentScan` 注解的作用是告诉 Spring：如果它们看起来像 Spring Bean，请查看与上下文配置相同的包中的所有 Java 类！

这意味着如果您的 MyApplicationContextConfiguration 位于 com.example 包中，Spring 将扫描以 com.example 开头的每个包，包括子包，以查找潜在的 Spring bean。

Spring如何知道某物是否是Spring bean？简单：目标类需要使用标记注释进行注释，称为`@Component`。

#### `@Component` 和 `@Autowired`

用 Component 注解修饰 UserDao 以便于它可以被 Spring 视作 Bean；

```java
import javax.sql.DataSource;
import org.springframework.stereotype.Component;

@Component
public class UserDao {
    private DataSource dataSource;

    public UserDao(DataSource dataSource) { // (1)
        this.dataSource = dataSource;
    }
}
```

1. 被 Component 注解修饰的方法也可以视作为 Bean；

> 如果查看 @Controller、@Service 或 @Repository 等注解的源代码时，你会发现它们都由多个注解组成，这些注解中总是包括 @Component！

基于 `Autowired` 注解，Spring 自动注入由`@Bean`修饰的 DataSource 的相关配置，然后用该特定 DataSource 创建新的 UserDao，代码如下：

```java
import javax.sql.DataSource;
import org.springframework.stereotype.Component;
import org.springframework.beans.factory.annotation.Autowired;

@Component
public class UserDao {

    private DataSource dataSource;
  	// 如果 DataSource 没有被 @Bean 修饰，那么运行时会抛出 NoSuchBeanDefinition 异常
    public UserDao(@Autowired DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

在 Spring 4.2 版本以前，构造函数注入必须显式指定 `@Autowired` 注解；4.2 版本以后的 Spring 已经聪明到可以自动注入构造函数，无需显式声明。代码如下

```java
@Component
public class UserDao {

    private DataSource dataSource;

    public UserDao(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

构造函数注入只是 Spring 依赖注入的其中一种方式，其他还有字段注入（Field Injection）、 Setter 注入、Method 注入。

Field 注入：

```java
@Component
public class UserDao {

    @Autowired
    private DataSource dataSource;

}
```

Setter 注入：

```java
@Component
public class UserDao {

    private DataSource dataSource;

    @Autowired
    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

这几种注入方式都可以获得一个有效的 Spring Bean。显然，不同的注入方式，也意味着在项目中应该使用哪种注入方式，存在一些争论。

#### 构造函数注入 VS 字段注入

网上已经有很多争论，构造函数注入还是字段注入更好，甚至有一些强烈的声音声称字段注入是有害的。

为了不给这些论点增加噪音，本文的观点是：

- 近年来，我在各种项目中使用了两种风格，构造函数注入和字段注入。仅基于个人经验，我并不真正偏爱一种风格。
- 一致性为王：不要对 80% 的 bean 使用构造函数注入，对 10% 使用字段注入，对剩余的 10% 使用方法注入。
- 官方文档中 Spring 的方法是：对强制依赖项使用构造函数注入，对可选依赖项使用 setter/field 注入。注意：要真正保持一致。

最重要的是：软件项目的整体成功并不取决于所选择的依赖注入方法。

### 总结

Spring 是一个基于依赖注入实现控制反转的轻量级容器。

----

References：

- [What is Spring Frameworks](https://www.marcobehler.com/guides/spring-framework)
- [Bean Overview](https://docs.spring.io/spring-framework/docs/5.2.x/spring-framework-reference/core.html#beans-definition)
- [Martin Fowler](https://martinfowler.com/), [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html), 2004.

