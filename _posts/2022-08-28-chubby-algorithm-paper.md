---
layout: post
title: "论文阅读笔记：The Chubby lock service for loosely-coupled distributed systems"
date: 2022-09-21
author: ingerchao
category: blog
tag:
- Distributed System
---

### Chubby Algorithm -- Mike Burrows, Google Inc. (2006)

### 摘要

我们描述了我们使用 Chubby 锁服务的经验，该服务旨在为松耦合（loosely-coupled）的分布式系统提供粗粒度（coarse-grained）的锁以及可靠的（尽管是低容量的）存储。 Chubby 提供的接口很像带有咨询锁的分布式文件系统，但设计重点是可用性（availability）和可靠性（reliability），而不是高性能。该服务的许多实例已经使用了一年多，其中几个实例同时处理了数万个客户端。本论文描述了初始设计和预期用途，将其与实际用途进行了比较，并解释了如何修改设计以适应差异。



-----

References:

- [Difference between Loosely Coupled and Tightly Coupled Multiprocessor System](https://www.geeksforgeeks.org/difference-between-loosely-coupled-and-tightly-coupled-multiprocessor-system/)

