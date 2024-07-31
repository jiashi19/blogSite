---
title: Redis
categories:
  - [others]
  - 秋招

date: 2024-07-22 20:56:40
tags:
password: 12345r
---

# Redis 

[图解Redis介绍 | 小林coding (xiaolincoding.com)](https://xiaolincoding.com/redis/)

## 常见数据类型和应用场景

<!-- more -->

### String

key-value结构

内部实现：String类型的底层的数据结构实现主要是int和SDS

命令：

- get
- set
- mget
- mset
- expire

场景：

常规计数：因为 Redis 处理命令是单线程，所以执行命令的过程是原子的。因此 String 数据类型适合计数场景，比如计算访问次数、点赞、转发、库存数量等等。

分布式锁：暂时没学懂

共享 Session 信息：在分布式系统中服务器分别保存session，则会导致用户出现重复登陆的情况（分布式系统可能会把请求随机到不同的服务器）-->解决：同一redis 管理session信息

### List

简单的字符串列表，可从头或尾添加元素

内部实现：底层数据结构是由双向列表或压缩链表 。在 Redis 3.2 版本之后，List 数据类型底层数据结构就只由 quicklist 实现了，替代了双向链表和压缩列表

命令：

- LPUSH/RPUSH
- LPOP/RPOP
- BLPOP/BRPOP

场景：

消息队列：

消息保序需求：先进先出的顺序对数据进行存取。List 可以使用 LPUSH + RPOP （or RPUSH+LPOP）命令实现消息队列。BRPOP命令也称为阻塞式读取，客户端在没有读到队列数据时，自动阻塞，直到有新的数据写入队列，再开始读取新数据

### Hash

Hash 是一个键值对（key - value）**集合**（ 或者说key-field-value），其中 value 的形式如： `value=[{field1，value1}，...{fieldN，valueN}]`。Hash 特别适合用于存储对象。

内部实现：Hash 类型的底层数据结构是由**压缩列表或哈希表**实现的。在 Redis 7.0 中，压缩列表数据结构已经废弃了，交由 listpack 数据结构来实现了。
一般对象用 String + Json 存储，对象中某些频繁变化的属性可以考虑抽出来用 Hash 类型存储。

命令：

- hmset
- hgetall

场景：购物车

## 缓存问题

### 缓存雪崩

redis大量缓存**同时**过期 or redis故障-->大量请求访问sql数据库-->数据库乃至系统崩溃

大量缓存同时过期：

1. 均匀设置过期时间（随机数）

2. 互斥锁：如果发现访问的数据不在 Redis 里，就加个互斥锁，保证同一时间内只有一个请求来构建缓存

3. 后台更新缓存：业务线程不再负责更新缓存，缓存也不设置有效期，设置缓存“永久有效”，并将更新缓存的工作交由后台线程**定时更新**。

   

### 缓存击穿

与缓存雪崩有点类似：热点数据缓存过期。

### 缓存穿透

缓存和数据库中均无数据：业务误操作 or 黑客恶意攻击

**策略**：

- 限制非法请求（大量非法请求也会导致缓存穿透）

- 缓存空值或者默认值
- 布隆过滤器

#### **布隆过滤器**

一个二进制向量加上多个hash函数；有误判；不能删除元素

https://github.com/daydreamdev/MeetingFilm/blob/master/note/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8%E8%A7%A3%E5%86%B3%E7%BC%93%E5%AD%98%E7%A9%BF%E9%80%8F.md

## Redis未授权访问漏洞

[Redis未授权漏洞复现及利用（window,linux）_windows下redis未授权访问漏洞修复-CSDN博客](https://blog.csdn.net/dreamthe/article/details/123427989)
