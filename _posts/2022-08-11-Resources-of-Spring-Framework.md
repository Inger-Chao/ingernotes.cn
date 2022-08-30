---
layout: post
title: "Resources of Spring Framework"
date: 2022-08-11
author: ingerchao
category: blog
tag:
- Spring
---



Java 语言中如果要通过 HTTP 或 FTP 协议访问文件，可以通过 UML 类编写一些管道代码。‘

同样，如果要在应用程序的类路径下（或者在 servlet 上下文中）读文件，必须要知道该应用程序的跟路径在哪里。因此，需要编写大量模版代码来访问路径，而且，每个用例（URL、类路径、servlet 上下文）的代码都会有所不同。

Spring 的抽象资源（resource abstraction）解决了上述问题。以如下的代码为例：

```java
import org.springframework.core.io.Resource;

public class MyApplication {

    public static void main(String[] args) {
            ApplicationContext ctx = new AnnotationConfigApplicationContext(someConfigClass); // (1)

            Resource aClasspathTemplate = ctx.getResource("classpath:somePackage/application.properties"); // (2)

            Resource aFileTemplate = ctx.getResource("file:///someDirectory/application.properties"); // (3)

            Resource anHttpTemplate = ctx.getResource("https://marcobehler.com/application.properties"); // (4)

            Resource depends = ctx.getResource("myhost.com/resource/path/myTemplate.txt"); // (5)

            Resource s3Resources = ctx.getResource("s3://myBucket/myFile.txt"); // (6)
    }
}
```

