---
layout: post
title: "Spring 配置参数最佳实践"
date: 2022-08-20 00:00:00
author: ingerchao
toc: true
category: blog
tag:
- Spring
- Java


---

## 背景

在使用 Spring 开发时，经常需要配置一些外部服务的参数，例如云存储、数据库连接等。如果直接在代码中硬编码这些参数，会导致：

1. 测试环境和生产环境使用相同的配置
2. 需要通过修改代码来动态调整配置
3. 敏感信息（如密钥）暴露在代码中

本文将介绍如何通过 Spring 的 `@Configuration` 和 `@PropertySource` 注解，实现配置参数的动态管理。

## 问题示例：硬编码配置

一开始的写法：

1. 私有静态常量字符串在程序中存储 secretId, secretKey, bucketName
2. 通过静态代码块，在初始化类时初始化 CosClient

```java
private static final String SECRET_ID = "AKIDa5TEsIUhz88zh0TK9lMK9thO9kLm6JH4";
private static final String SECRET_KEY = "tfP2VmsnZFnDpIcOBpJdptnV3YvJuVu8";
private static final String BUCKET_KEY = "6365-ceshihuanj-0gc41ib2612d4621-1308307270";

private static COSClient cosClient;

static {
    COSCredentials cred = new BasicCOSCredentials(SECRET_ID, SECRET_KEY);
    Region region = new Region("ap-shanghai");
    ClientConfig clientConfig = new ClientConfig(region);
    clientConfig.setHttpProtocol(HttpProtocol.https);
    cosClient = new COSClient(cred, clientConfig);
}
```

**缺点：**

1. 生产环境用了测试环境的存储环境
2. 需要通过修改代码来动态配置小程序的存储环境

## 解决方案：基于 properties 的动态配置

### 步骤一：定义环境配置文件

针对生产环境和测试环境，分别定义 COS 客户端的参数：

```properties
# 开发环境、测试环境 development/cos.properties
cos.appId=1400581424
cos.region=ap-shanghai
cos.bucketName=6365-ceshihuanj-0gc41ib2612d4621-1308307270
cos.secretKey=tfP2VmsnZFnDpIcOBpJdptnV3YvJuVu8
cos.secretId=AKIDa5TEsIUhz88zh0TK9lMK9thO9kLm6JH4
cos.hosts=https://6365-ceshihuanj-0gc41ib2612d4621-1308307270.tcb.qcloud.la

# 生产环境 production/cos.properties
cos.appId=1400581424
cos.region=ap-shanghai
cos.bucketName=7072-prod-1gskrgmsdae0e163-1308307270
cos.secretKey=tfP2VmsnZFnDpIcOBpJdptnV3YvJuVu8
cos.secretId=AKIDa5TEsIUhz88zh0TK9lMK9thO9kLm6JH4
cos.hosts=https://7072-prod-1gskrgmsdae0e163-1308307270.cos.ap-shanghai.myqcloud.com
```

### 步骤二：使用 @Configuration 加载配置

通过 `@Configuration` 注解定义一个配置类，通过 `@PropertySource` 注解将 properties 配置文件中的值存储到 Spring 的 Environment 中，并指定配置文件地址。

**注意：** 由于 `@Value` 不能直接注入值给静态属性，Spring 不允许（或不支持）把值注入到静态变量中。因此，通过非静态的赋值方法将值注入到静态变量中。

```java
@Configuration
@PropertySource(value = "classpath:cos.properties", encoding = "utf-8")
public class COSUtil {
    private static String SECRET_ID = "cos.secretId";
    private static String SECRET_KEY = "cos.secretKey";
    private static String BUCKET_KEY = "cos.bucketName";
    private static String REGION = "cos.region";

    @Value("${cos.secretId}")
    public void setSecretId(String secretId) {
        SECRET_ID = secretId;
    }

    @Value("${cos.secretKey}")
    public void setSecretKey(String secretKey) {
        SECRET_KEY = secretKey;
    }

    @Value("${cos.bucketName}")
    public void setBucketKey(String bucketKey) {
        BUCKET_KEY = bucketKey;
    }

    @Value("${cos.region}")
    public void setRegion(String region) {
        REGION = region;
    }
}
```

### 步骤三：使用 @PostConstruct 初始化客户端

若此时还在静态区中对 COSClient 进行初始化，会导致读取到的参数是代码定义的值。因此，通过 `@PostConstruct` 注解，让 Spring 在初始化时执行 COSClient 的初始化逻辑。

```java
@PostConstruct
public void initClient() {
    COSCredentials cred = new BasicCOSCredentials(SECRET_ID, SECRET_KEY);
    Region region = new Region(REGION);
    ClientConfig clientConfig = new ClientConfig(region);
    clientConfig.setHttpProtocol(HttpProtocol.https);
    cosClient = new COSClient(cred, clientConfig);
}
```

## 总结

通过上述方式，实现了 COSClient 的参数根据环境，由 properties 来动态配置参数。这样做的好处：

1. **环境隔离**：不同环境使用不同的配置文件
2. **易于维护**：配置与代码分离，修改配置无需改动代码
3. **安全性**：敏感信息可以放在配置文件中，避免硬编码在代码里
4. **灵活性**：可以通过不同的部署策略加载不同的配置文件

## 参考资料

- [Spring @Configuration 注解](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html)
- [Spring @PropertySource 注解](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/PropertySource.html)
- [Spring @PostConstruct 注解](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/PostConstruct.html)
