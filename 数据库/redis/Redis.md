## redis 简介

- redis 是一款开源，c 语言编写的，高级键值(key-value)缓存和支持永久存储的 nosql 数据库
- redis 采用内存(In-Memory)数据集(DataSet)
- 支持多数据类型
- 运行于大多数 posix，如 linux、`os` x 等

官网：[https://redis.io/](https://redis.io/)

中文网站：http://redis.cn/

## redis 特性优势

```
高速读写
数据类型丰富
支持持久化
支持高可用
支持分布式分片集群
多种内存分配回收策略
支持事务
消息队列、消息订阅

```

## redis 与 memcached 对比

**redis:**

优点：高性能读写，多数据类型支持，数据持久化，高可用架构，支持自定义虚拟内存，支持分布式卡片集群，单线程读写性能极高

缺点：多线程读写比 memcached 慢

**memcached:**

优点：高性能读写，单数据类型，支持客户端分布式集群，一致性 hash 多核结构，多线程读写性能高

缺点：没有持久化，节点故障可能出现缓存穿透，分布式需要客户端实现，跨机房数据同步困难，架构扩容复杂度高

![reids](./redis_img/reids.png)

## redis 应用场景

- 数据高速缓存(mysql...热点内容)提高读取性能
- web 会话缓存(session)
- 排行榜应用
- 消息队列
- 发布订阅
