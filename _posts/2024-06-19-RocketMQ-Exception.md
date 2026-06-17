---
layout: post
title: RocketMQ RemotingTooMuchRequestException
date: 2024-06-20
author: ingerchao
category: blog
toc: true
tag: 
- Java
---

```bash
org.apache.rocketmq.remoting.exception.RemotingTooMuchRequestException: sendDefaultImpl call timeout
        at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.sendDefaultImpl(DefaultMQProducerImpl.java:640)
        at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.send(DefaultMQProducerImpl.java:1310)
        at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.send(DefaultMQProducerImpl.java:1256)
        at org.apache.rocketmq.client.producer.DefaultMQProducer.send(DefaultMQProducer.java:339)
        at com.example.app.bsc.rocketmq.RocketMqService.sendMessage(RocketMqService.java:79)
```

问题：rocketmq 服务端游太多请求，导致消息生产异常。

Step 1: 查看服务器 cpu、内存、磁盘、网络资源使用情况

```bash
[admin@gz-cvm-ebuild-wanyihua-dev001 ~]$ top
top - 14:23:43 up 757 days, 21:08,  1 user,  load average: 0.05, 0.05, 0.05
Tasks: 425 total,   1 running, 424 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.9 us,  0.2 sy,  0.0 ni, 98.8 id,  0.1 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 16265548 total,  1005372 free,  3501676 used, 11758500 buff/cache
KiB Swap:        0 total,        0 free,        0 used.  3859440 avail Mem

[admin@gz-cvm-ebuild-wanyihua-dev001 ~]$ free -h
              total        used        free      shared  buff/cache   available
Mem:            15G        3.3G        984M        8.2G         11G        3.7G
Swap:            0B          0B          0B

# 确认 rockmq 服务启动
[admin@gz-cvm-ebuild-wanyihua-dev001 ~]$ netstat -tln | grep 8876
tcp6       0      0 :::8876                 :::*                    LISTEN
```

Setp 2: 查看 RocketMQConsole ，发现消息有 308条，全都是 2022年的历史消息。

```bash
#!/bin/sh
# createRocketMqConsole.sh

docker run -d   -v /home/admin/rocketmq-console:/tmp/rocketmq-console/data -e "JAVA_OPTS=-Drocketmq.namesrv.addr=10.189.64.136:8876 -Dcom.rocketmq.sendMessageWithVIPChannel=false -Drocketmq.config.loginRequired=true" -p 8877:8080 styletang/rocketmq-console-ng:latest
```

Step3: 修改 broker.conf 设置消息过期时间

```bash
[admin@gz-cvm-ebuild-wanyihua-dev001 conf]$ cat broker.conf
brokerClusterName = DefaultCluster
brokerName = broker-a
# 如果消息在3天内没有被消费，它将被自动删除
messageExpiredTime = 3
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
# 新增的配置，broker默认端口
listenPort=8877
brokerIP1=10.189.64.136
```

查看 broker 运行状态：

```bash
[admin@gz-cvm-ebuild-wanyihua-dev001 conf]$ ps -ef | grep broker
admin    9959     1  0  2022 ?        00:00:00 sh bin/mqbroker -n localhost:8876
admin    9963  9959  0  2022 ?        00:00:00 sh /home/admin/rocketmq/bin/runbroker.sh org.apache.rocketmq.broker.BrokerStartup -n localhost:8876
admin    9966  9963  3  2022 ?        19-02:46:22 /bin/java -server -Xms128m -Xmx128m -Xmn512m -XX:+UseG1GC -XX:G1HeapRegionSize=16m -XX:G1ReservePercent=25 -XX:InitiatingHeapOccupancyPercent=30 -XX:SoftRpefLRUPolicyMSPerMB=0 -XX:SurvivorRatio=8 -verbose:gc -Xloggc:/dev/shm/mq_gc_%p.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCApplicationStoppedTime -XX:+PrintAdaptiveSizePolicy -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=30m -XX:-OmitStackTraceInFastThrow -XX:+AlwaysPreTouch -XX:MaxDirectMemorySize=15g -XX:-UseLargePages -XX:-UseBiasedLocking -Djava.ext.dirs=/jre/lib/ext:/home/admin/rocketmq/bin/../lib -cp .:/home/admin/rocketmq/bin/../conf: org.apache.rocketmq.broker.BrokerStartup -n localhost:8876
```

重启broker

```bash
sh bin/mqbroker -n localhost:8876 &
```

:sparkles: 历史消息已经清空了。

发现问题依然存在：参考 ref2 修改发送消息时间从 3000ms 到 60000 出现了下面问题：

Q2:

```bash
2024-06-19 16:57:34.061 logId[2a56e487865e4e91a2a6537c62347233] [http-nio-8896-exec-3] ERROR com.example.app.bsc.rocketmq.RocketMqService  send rocket mq failed, body: {"businessId":"1718787448001002","system":"BSC","businessType":4,"customerId":4480,"operateType":"UPDATE","operateContent":"新增组织[123]","operatorId":0,"operatorName":"system","operateTime":1718787448019}
org.apache.rocketmq.client.exception.MQClientException: Send [3] times, still failed, cost [6031]ms, Topic: bsc-dev, BrokersSent: [gz-cvm-ebuild-wanyihua-dev001.gz.admin.com, gz-cvm-ebuild-wanyihua-dev001.gz.admin.com, gz-cvm-ebuild-wanyihua-dev001.gz.admin.com]
See http://rocketmq.apache.org/docs/faq/ for further details.
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.sendDefaultImpl(DefaultMQProducerImpl.java:638)
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.send(DefaultMQProducerImpl.java:1310)
	at org.apache.rocketmq.client.impl.producer.DefaultMQProducerImpl.send(DefaultMQProducerImpl.java:1256)
	at org.apache.rocketmq.client.producer.DefaultMQProducer.send(DefaultMQProducer.java:339)
	at com.example.app.bsc.rocketmq.RocketMqService.sendMessage(RocketMqService.java:79
```

AK, SK 有问题删除配置可以了

----

参考资料:

1. [RocketMQ官方文档](https://rocketmq.apache.org/)

2. [CSDN blog1](https://blog.csdn.net/Dream_xun/article/details/109555340)