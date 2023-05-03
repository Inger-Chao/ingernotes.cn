---
layout: post
title: "论文阅读-Zookeeper: Wait-free coordination for Internet-scale system"
date: 2022-11-07
author: ingerchao
category: blog
tag:
- Distributed System
---

## Zookeeper: 无需等待的互联网规模协调系统

Zookeeper 是什么？动物园管理者？一图胜千言。Zookeeper 是由雅虎研发的顶层软件项目，为大型分布式计算提供开源的分布式配置服务、同步服务和命名注册。

![image-20221107173955674](./../assets\images\program\backend\apache-and-zookeepter.png)

目前，分布式应用往往是由多个独立的程序运行在多个机器上的，开发人员必须要处理协调逻辑和应用程序本身的逻辑。

![image-20221108153726865](./../\assets\images\program\backend\zookeeper-question-1.png)

![image-20221108153758367](./../\assets\images\program\backend\zookeeper-question-2.png)

![image-20221108153824934](./../\assets\images\program\backend\zookeeper-question-3.png)

具体来说，需要为每个特定任务都开发一个协调逻辑：

- 主节点选择
- 锁服务
- 配置

总的来说：提供一个API来实现上述多个服务

ZooKeeper：提供了公共的API，以便于开发人员可以实现自己的原语。

#### 专业术语定义

- *Client*：客户端，ZooKeeper 服务的用户；
- *Server*: 服务端，提供 Zookeeper 服务的程序；
- *znode*: 内存里的 Zookeeper 数据节点，通过命名空间的层级来组织（the data tree）；
- *Update/write*: 任何操作，更新 data tree 的状态；
- 当客户端连接到 ZooKeeper 时建立会话（*Session*）

#### ZooKeeper 的数据模型

- znodes 由层级的命名空间来组织；
- 使用标准 Unix 文件系统路径的符号来表示；
- znodes 不是通常的数据存储结构，而是为了映射到客户端应用程序的抽象。

#### Znode

- Znode 通常由客户端显式创建或删除；

- 临时情况下：显式删除 znode，或让系统在会话创建终止时自动移除 znode；
- *SEQUENTIAL* 标志位：
  - 单调递增计数器，附加于znode的路径上
  - 父节点下的新 znode 的计数值总是大于已有的子节点 znode 的计数器值；
- *WATCH* 标志位：
  - 允许客户端在无需轮询的情况下及时接受变更的通知；
  - Watches 表明发生了变化，但不提供变更逻辑；

#### [ZooKeeper API](https://zookeeper.apache.org/doc/r3.4.6/api/org/apache/zookeeper/ZooKeeper.html)

| 方法    | 入参                | 返回值       | 说明                |
| ------- | ------------------- | ------------ | ------------------- |
| create  | path, data, flags   | String       | 创建znode           |
| delete  | path, version       | void         | 删除znode           |
| exists  | path, watch         | Stat         |                     |
| getData | path, watch         | (data, Stat) | 获取znode数据和状态 |
| setData | path, data, version | Stat         | 如果当前版本和入参  |

ZooKeeper 不提供句柄来访问 znodes；

所有的方法都有同步和异步两种实现方式。

#### ZooKeeper 原则

Zookeeper 线性写入：所有更新 Zookeeper 状态的请求都可序列化并服从优先级。

FIFO 客户端顺序：来自给定客户端的所有请求都按照客户端发送的顺序执行。

#### 原语的例子

当选取一个新的主节点时，我们**不希望其他进程使用的配置做出改变**。如果新的主节点在配置文件更新的过程中挂了，我们**不希望进程使用的是配置的一部分内容**。

因此，删除 ready 态的 znode，随后，znodes 更新各种配置文件，再创建ready 态的 znode。

**1. 配置（Configuration）**

1. 新请求如何查询配置文件？
   - 假设配置文件存储在 /app1/config 路径下， ` getData(/app/config, true)`
2. 管理员如何在运行时改变配置文件？
   - `setData(/app1/config/config_data, -1)`
3. 其他程序如何读取新配置文件？
   - `getData(/app1/config, true)`

**2. 组成员（Group Membership）**

![image-20221108175433393](./../\assets\images\program\backend\zookeeper-question-group-memberships.png)

**3. 简单的锁（Simple Locks）**

一个应用程序中的对象如何通过锁公用一个数据源？

![image-20221108175225762](./../\assets\images\program\backend\zookeeper-simple-locks.png)

**4. 没有羊群效应的简单锁（Simple Locks without Herd Effect）**

> Herd Effect 注：在计算机科学中，当事件发生时，大量等待事件的进程或线程被唤醒，但只有一个进程能够处理该事件时，就会出现从众问题。 当进程唤醒时，它们将各自尝试处理该事件，但只有一个会获胜。

![image-20221108193034660](./../\assets\images\program\backend\zookeeper-simple-locks-without-herd-effect.png)

**5. 读/写锁**

![image-20221108193220354](./../\assets\images\program\backend\zookeeper-simple-read-write-locks.png)

![image-20221108200009555](./../\assets\images\program\backend\read-write-lock.png)

吞吐量：设置250个客户端，每个客户端至少又 100 个未完成的请求（对1k数据的读或写）。

<img src="C:\Users\SFTC\AppData\Roaming\Typora\typora-user-images\image-20221108200319692.png" alt="image-20221108200319692" style="zoom:50%;" />

#### 总结

无需等待、事件排序、Watch 机制

-----

- [ZooKeeper wiki](https://cwiki.apache.org/confluence/display/ZOOKEEPER/Index)
- https://people.eecs.berkeley.edu/~istoica/classes/cs294/15/notes/18-zookeeper.pdf
- https://pdfs.semanticscholar.org/e310/b53377ae41eb56167b12c828f12d799073e1.pdf