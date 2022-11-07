---
layout: post
title: "论文阅读-Zookeeper: Wait-free coordination for Internet-scale system"
date: 2022-11-07
author: ingerchao
category: blog
tag:
- Distributed System
---





----

## Zookeeper: 无需等待的互联网规模协调系统

Zookeeper 是什么？动物园管理者？一图胜千言。Zookeeper 是由雅虎研发的顶层软件项目，为大型分布式计算提供开源的分布式配置服务、同步服务和命名注册。

![image-20221107173955674](./../assets\images\program\backend\apache-and-zookeepter.png)

目前，分布式应用往往是由多个独立的程序运行在多个机器上的，开发人员必须要处理协调逻辑和应用程序本身的逻辑。

![主节点选择](./..\assets\images\program\backend\zookeeper-question-1)

![image-20221107175422692](./../assets\images\program\backend\zookeeper-question-2.png)

![image-20221107175453959](./..\assets\images\program\backend\zookeeper-question-3.png)