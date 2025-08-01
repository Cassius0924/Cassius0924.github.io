---
title: 'Redis 为什么采用单线程，为什么性能好'
draft: false
date: 2024-06-26T15:40:01+08:00
description: ''
author: 'Cassius0924'
tags: ["Redis", "Single-Threaded", "Performance", "Architecture", "Notes"]
---

# Redis 为什么采用单线程，为什么性能好

Redis 采用单线程的原因是因为在 **内存中** 进行读写操作，**CPU不是Redis的性能瓶颈**，而是内存和带宽，所以采用单线程可以避免 **线程切换和锁的开销**。

## Redis 是什么？

Redis 是一个开源的内存数据库，它可以存储键值对，支持多种数据结构，如字符串（string）、哈希（hash）、列表（list）、集合（set）、有序集合（zset）等。

## Redis 是不是单线程？

实际上，Redis 是多线程的，其内部有以下几个线程：

- `redis-server`：主线程，负责接收客户端的连接，读取请求，发送响应。

- `bio-close-file`：负责异步关闭大文件。

- `bio-aof-fsync`：负责将 AOF 文件异步刷到磁盘。

- `bio-lasy-free`：负责异步释放大内存。

- `jemalloc-bg-thread`：负责内存碎片整理。

- `io-thread`：IO 线程，负责 read/write，decode/encode。

> [!NOTE]
> 
> **AOF（Append Only File）** 是 Redis 的一种持久化方式，将所有写操作追加到文件末尾，重启时重新执行 AOF 文件中的命令即可恢复数据。实时硬盘操作，不会丢失数据，但是会影响性能。
> 
> 除了 AOF，Redis 还有一种持久化方式是 **RDB（Redis DataBase）** ，它是将内存中的数据快照保存到磁盘上。非实时硬盘操作，可能会丢失数据，但是性能更好。

## Redis 为什么采用单线程？

- Redis 不是 CPU 密集型应用，CPU 不是 Redis 的性能瓶颈。

- 如果采用多线程，会导致加锁解锁的开销大，CPU 上下文切换开销大。数据库请求量变化大，一会有请求，一会无请求，多线程会在运行和阻塞状态来回切换。

## 单线程的 Redis 为什么速度很快？

### Redis 机制

- 内存数据库：Redis 是一个内存数据库，内存的读写速度相比于磁盘的读写速度要快很多。

- 数据结构高效：Redis 使用了很多高效的数据结构，如哈希表、跳表等。

- 使用了 Reactor 模型：Redis 使用了 Reactor 模型，采用了 IO 多路复用技术，可以处理多个客户端请求。采用的是非阻塞 IO，不会因为一个 IO 阻塞导致其他任务无法执行。

### Redis 采用的优化

- 耗时的操作，另起线程：如 AOF 文件的刷盘操作，Redis 会另起线程异步执行。

- 拆分大任务：例如 Dict 的扩容（rehash）操作，Redis 会将其拆分为多个小任务，分批执行。

- 不同的对象类型选择不同的数据结构：根据节点数量，Redis 会选择不同的数据结构，以取得空间和时间上的平衡。
