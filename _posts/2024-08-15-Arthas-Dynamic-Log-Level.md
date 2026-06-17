---
layout: post
title: "Arthas 实战：动态修改日志级别"
date: 2024-08-15 00:00:00
author: ingerchao
toc: true
category: blog
tag:
- Arthas
- Java


---

## 什么是 Arthas

Arthas 是 Alibaba 开源的 Java 诊断工具，能够帮助开发人员快速定位和诊断 Java 应用的问题。它可以在不重启应用的情况下，动态查看和修改应用的行为。

## 应用场景

在生产环境中，经常遇到以下场景：

1. **日志级别过低** - 关键业务逻辑的日志太多，影响性能
2. **日志级别过高** - 需要排查问题时，日志信息不足
3. **临时调试** - 需要临时开启某个类的详细日志
4. **紧急修复** - 需要快速调整日志级别，但不能重启应用

传统方式需要修改配置文件并重启应用，而使用 Arthas 可以在线动态修改。

## 实战操作

### 步骤一：连接到目标进程

首先，找到需要诊断的 Java 进程：

```bash
# 列出所有 Java 进程
jps -l

# 或者使用 Arthas 的 dashboard 命令
java -jar arthas-boot.jar
```

### 步骤二：查找 ClassLoader

在修改日志级别之前，需要先找到目标类使用的 ClassLoader：

```bash
# 查找 Logger 相关的 ClassLoader
[arthas@317]$ sc -d org.slf4j.LoggerFactory | grep classLoaderHash
 classLoaderHash   ed17bee
 classLoaderHash   1efdcd5
```

**说明：**
- `sc` 命令用于搜索已加载的类
- `-d` 参数显示详细信息
- `grep classLoaderHash` 过滤出 ClassLoader 的 hash 值

### 步骤三：确认正确的 ClassLoader

多个 ClassLoader 可能对应不同的日志框架，需要确认正确的 ClassLoader：

```bash
# 查看第一个 ClassLoader
[arthas@317]$ logger -c ed17bee
[arthas@317]$

# 查看第二个 ClassLoader
[arthas@317]$ logger -c 1efdcd5
 name                      root
 class                     org.apache.logging.log4j.core.config.LoggerConfig
 classLoader               org.springframework.boot.loader.LaunchedURLClassLoader@1efdcd5
 classLoaderHash           1efdcd5
```

**说明：**
- `logger -c <hash>` 查看指定 ClassLoader 的日志配置
- 根据返回信息判断哪个是正确的日志框架

### 步骤四：修改日志级别

确认正确的 ClassLoader 后，就可以修改具体类的日志级别：

```bash
# 修改指定类的日志级别为 ERROR
[arthas@317]$ logger --name com.example.module.common.base.context.LoginUserHolder \
    --level ERROR -c 1efdcd5
Update logger level success.
```

**参数说明：**
- `--name` 指定类的完整名称
- `--level` 指定日志级别（TRACE, DEBUG, INFO, WARN, ERROR）
- `-c` 指定 ClassLoader 的 hash 值

### 步骤五：验证修改结果

修改完成后，验证日志级别是否生效：

```bash
[arthas@317]$ logger --name com.example.module.kafka.consumer.WaybillBatchConsumer
 name                               com.example.module.kafka.consumer.WaybillBatchConsumer
 class                              ch.qos.logback.classic.Logger
 classLoader                        org.springframework.boot.loader.LaunchedURLClassLoader@63e2203c
 classLoaderHash                    63e2203c
 level                              ERROR
 effectiveLevel                     ERROR
```

**说明：**
- `level` 显示当前设置的日志级别
- `effectiveLevel` 显示实际生效的日志级别

## 实际应用案例

### 案例一：降低 Kafka Consumer 的日志级别

**场景：** Kafka Consumer 处理大量消息，DEBUG 日志太多，影响性能。

**操作：**
```bash
# 查看当前日志级别
[arthas@291]$ logger --name com.example.module.kafka.consumer.WaybillBatchConsumer
 name                               com.example.module.kafka.consumer.WaybillBatchConsumer
 class                              ch.qos.logback.classic.Logger
 classLoader                        org.springframework.boot.loader.LaunchedURLClassLoader@63e2203c
 classLoaderHash                    63e2203c
 level                              null
 effectiveLevel                     INFO

# 修改为 ERROR 级别
[arthas@291]$ logger --name com.example.module.kafka.consumer.WaybillBatchConsumer \
    --level ERROR -c 63e2203c
Update logger level success.

# 验证修改结果
[arthas@291]$ logger --name com.example.module.kafka.consumer.WaybillBatchConsumer
 name                               com.example.module.kafka.consumer.WaybillBatchConsumer
 class                              ch.qos.logback.classic.Logger
 classLoader                        org.springframework.boot.loader.LaunchedURLClassLoader@63e2203c
 classLoaderHash                    63e2203c
 level                              ERROR
 effectiveLevel                     ERROR
```

### 案例二：批量修改多个 Consumer 的日志级别

**场景：** 多个 Kafka Consumer 都需要调整日志级别。

**操作：**
```bash
# 修改 WaybillBatchConsumer
logger --name com.example.module.kafka.consumer.WaybillBatchConsumer \
    --level ERROR -c 63e2203c

# 修改 BuildingConsumer
logger --name com.example.module.kafka.consumer.BuildingConsumer \
    --level ERROR -c 63e2203c

# 修改 SpacingInfoConsumer
logger --name com.example.module.kafka.consumer.SpacingInfoConsumer \
    --level ERROR -c 63e2203c
```

### 案例三：排查问题时临时开启 DEBUG 日志

**场景：** 生产环境出现问题，需要临时开启 DEBUG 日志排查。

**操作：**
```bash
# 开启 DEBUG 日志
logger --name com.example.module.service.impl.CalculateServiceImpl \
    --level DEBUG -c 63e2203c

# 问题排查完成后，恢复为 INFO 级别
logger --name com.example.module.service.impl.CalculateServiceImpl \
    --level INFO -c 63e2203c
```

## Arthas 常用命令

### 1. 查看类信息

```bash
# 搜索类
sc com.example.module.*

# 查看类详细信息
sc -d com.example.module.service.UserService
```

### 2. 查看方法调用

```bash
# 监控方法调用
watch com.example.module.service.UserService getUser '{params, returnObj}'

# 监控方法执行时间
trace com.example.module.service.UserService getUser
```

### 3. 查看对象内容

```bash
# 查看 Spring Bean
vmtool --action getInstances \
    --className com.example.module.service.UserService \
    --limit 1

# 查看对象字段
ognl '@com.example.module.service.UserService@instance'
```

### 4. 动态修改配置

```bash
# 修改静态变量
ognl -c 63e2203c '@com.example.module.config.Config@setValue("newValue")'

# 调用方法
ognl -c 63e2203c '@com.example.module.service.CacheService@refresh()'
```

## 注意事项

### 1. 生产环境使用

- **谨慎操作** - 生产环境修改可能影响业务
- **及时恢复** - 临时修改后及时恢复原配置
- **记录操作** - 记录所有修改操作，便于追溯

### 2. 性能影响

- **监控命令有开销** - 频繁使用可能影响性能
- **及时退出** - 诊断完成后及时退出 Arthas
- **避免高峰期** - 尽量在低峰期进行诊断

### 3. 安全问题

- **权限控制** - 确保只有授权人员可以使用
- **审计日志** - 记录所有操作日志
- **网络隔离** - 避免在公网暴露 Arthas 端口

## 最佳实践

### 1. 日志级别管理

**推荐的日志级别策略：**

- **生产环境** - INFO 或 WARN
- **测试环境** - DEBUG 或 INFO
- **开发环境** - DEBUG 或 TRACE
- **问题排查** - 临时调整为 DEBUG

### 2. 批量操作脚本

对于需要批量修改的场景，可以编写脚本：

```bash
#!/bin/bash

# 定义要修改的类列表
CLASSES=(
    "com.example.module.kafka.consumer.WaybillBatchConsumer"
    "com.example.module.kafka.consumer.BuildingConsumer"
    "com.example.module.kafka.consumer.SpacingInfoConsumer"
)

# ClassLoader hash
CLASSLOADER_HASH="63e2203c"

# 批量修改
for class in "${CLASSES[@]}"; do
    echo "Modifying $class ..."
    logger --name $class --level ERROR -c $CLASSLOADER_HASH
done
```

### 3. 监控和告警

**建议：**

- 监控日志级别变化
- 设置告警机制
- 定期审查日志配置

## 总结

Arthas 是一个强大的 Java 诊断工具，动态修改日志级别只是其众多功能之一。

**核心要点：**

1. **无需重启** - 在线修改日志级别，不影响业务
2. **实时生效** - 修改后立即生效，便于排查问题
3. **灵活控制** - 可以精确控制到具体的类
4. **功能丰富** - 除了日志管理，还支持方法监控、对象查看等

**使用场景：**

- 生产环境问题排查
- 性能优化调优
- 紧急故障处理
- 临时调试需求

掌握 Arthas 的使用，可以大大提高 Java 应用的运维效率和问题排查能力。

---

## 参考资料

- [Arthas 官方文档](https://arthas.aliyun.com/doc/)
- [Arthas GitHub 仓库](https://github.com/alibaba/arthas)
- [Arthas 使用教程](https://arthas.aliyun.com/doc/quick-start.html)
