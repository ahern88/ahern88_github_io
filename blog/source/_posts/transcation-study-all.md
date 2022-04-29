---
title: 事务学习合集
comment: true
date: 2022-04-26 21:22:18
categories:
  - 数据库
tags:
  - 事务
  - 数据库
---

从JDBC的事务测试，到Spring的事务封装，XA事务测试，消息事务，TCC尝试，SAGA，到阿里分布式事务框架Seata的剖析。

## 事务基础

### 什么是事务？

事务的核心就是 锁与并发，通过锁实现数据的原子更新与一致性保证，并保证在并发的时候实现ACID等数据库特性。



## 单机事务

- ACID的实现
- 事务的隔离机制
- 多版本并发控制

## 分布式事务

- 如何实现XA事务？

- 使用消息中间件实现最终一致性事务

- 使用TCC实现分布式事务

- 使用SAGA模式实现分布式事务

- 使用Seata的多种模式实现分布式事务



## 参考资料

- 事务测试源代码：https://github.com/ahern88/transaction-test
- Seata源码：https://github.com/seata/seata
- Seata示例：https://github.com/seata/seata-samples
