---
title: 'HTTP 协议版本变化学习笔记'
draft: false
date: 2024-06-27T20:12:42+08:00
description: ''
author: 'Cassius0924'
tags: ["HTTP", "Network", "Web", "Protocol", "Optimization", "Concurrency"]
---

# HTTP 协议版本变化学习笔记

互联网发展至今，HTTP 协议已经发展了多个版本，分别为 `HTTP/1.0`、`HTTP/1.1`、`HTTP/2.0`、`HTTP/3.0`，本文将对这几个版本的变化进行学习笔记。

![HTTP 协议层](https://s2.loli.net/2024/06/27/6KE87inXyFe4DbW.png)

## HTTP/1.0

HTTP/1.0 是最早的 HTTP 协议版本，它的特点是：

1. 每次请求都会建立一个新的 TCP 连接，请求结束后立即断开连接。

2. 每个请求都会包含完整的请求头和请求体。

3. 不支持持久连接，每次请求都需要重新建立连接。

4. 不支持管道化，即同一个连接中不能同时发送多个请求。

## HTTP/1.1

HTTP/1.1 是对 HTTP/1.0 的改进，它的特点是：

1. 支持持久连接（Keep-Alive），即同一个连接可以发送多个请求。

2. 支持管道化，即同一个连接中可以同时发送多个请求。但是，由于 HTTP/1.1 中的管道化存在队头阻塞问题，所以很少被使用。默认为关闭状态，并且大多数浏览器也不支持。 **所以我们认为 HTTP/1.1 不支持管道化** 。

HTTP/1.1 的缺点：

1. 队头阻塞问题：如果一个请求响应时间过长，那么后面的请求就会被阻塞。

2. 明文传输：HTTP/1.1 的数据传输是纯文本且未加密的，容易被窃听。比如状态码 `200` 会被分为 `2`、`0`、`0` 三个字节传输。这点会在 HTTP/2.0 中得到改进。

3. 头部冗余：每次请求都需要携带完整的请求头，头部信息冗余。

![]()

## HTTPS

在讲述 HTTP/2.0 之前，我们先来了解一下 HTTPS。

HTTPS 是在 HTTP 的基础上加入了 SSL/TLS 加密层，使得数据传输更加安全。HTTPS 的特点是：

1. 数据加密：HTTPS 使用 SSL/TLS 加密传输数据，保证数据传输的安全性。

2. 身份认证：HTTPS 使用证书机制对服务器和客户端进行身份认证，防止中间人攻击。

3. 数据完整性：HTTPS 使用数字签名对数据进行完整性校验，防止数据被篡改。

加密方式：对称加密、非对称加密、数字签名。

通信前使用非对称加密协商对称加密的密钥，通信过程使用对称加密传输数据，保证数据传输的安全性。

![HTTPS](https://s2.loli.net/2024/06/27/wcztbDYBevjP26L.png)

## HTTP/2.0

HTTP/2.0 是对 HTTP/1.1 的重大升级，它的特点是：

1. 头部压缩：HTTP/2.0 使用 HPACK 算法对头部进行压缩，如果请求头中包含相同的字段，只需要发送一次，后续请求只需要发送索引，接着从静态表或动态表中获取对应的值。解决了 HTTP/1.1 中头部冗余的问题。

2. 二进制传输：HTTP/2.0 使用二进制格式传输数据，而不是 HTTP/1.x 的文本格式。如状态码 `200` 只需要传输一个字节，但不是直接将 `200` 表示为二进制，而是通过 `HPACK` 算法压缩后的索引。HTTP/2.0 + HTTPS 解决了 HTTP/1.1 明文传输的问题。

3. 多路复用（并发传输）：HTTP/2.0 支持多路复用，即在一个连接上可以同时发送多个请求和响应。HTTP/2.0 引入了 `Stream` 流的概念，每个 `Stream` 都有一个唯一的标识符，一个连接上可以有多个 `Stream`。一定程度上缓解了 HTTP/1.1 的队头阻塞问题。

4. 服务端主动推送：HTTP/2.0 支持服务端主动推送资源，即在客户端请求一个资源时，服务端可以主动推送相关资源给客户端，减少客户端请求次数。

HTTP/2.0 的缺点：

1. 队头阻塞问题：虽然 HTTP/2.0 引入了多路复用，但是由于 TCP 的特性，依然存在队头阻塞问题。因为 TCP 是一个有序的传输协议，如果一个数据包丢失，后续的数据包需要等待重传，导致后续数据包被阻塞。

2. TCP 与 TLS 握手延迟：TCP 建立连接需要三次握手，TLS 握手需要四次握手，这些握手过程会增加延迟。合并之后仍然需要 3 个 RTT 的时延。

![HTTP/2.0](https://s2.loli.net/2024/06/27/fbDGehmnSacylzO.png)

3. 网络迁移需要重新建立连接：一个 TCP 连接是由四元组（源 IP 地址，源端口，目标 IP 地址，目标端口）确定的，这意味着如果 IP 地址或者端口变动了，就会导致需要 TCP 与 TLS 重新握手，这不利于移动设备切换网络的场景，比如 4G 网络环境切换成 WiFi。


## HTTP/3.0

### QUIC

在讲述 HTTP/3.0 之前，我们先来了解一下 QUIC。

![QUIC](https://s2.loli.net/2024/06/27/Ccd8sbMvz6ODoUZ.png)

QUIC 是基于 UDP 的传输协议，由 Google 开发，用于替代 TCP。QUIC 是在应用层，也就是用户空间实现的协议，相比在内核实现的 TCP 协议，QUIC 协议不再僵硬，可以十分灵活的进行设置。

QUIC 的特点是：

1. 无队头阻塞：QUIC 使用多路复用，每个数据流独立，不会相互影响。相比于 HTTP/2.0 基于 TCP 的多路复用，由于 UDP 是无序传输的，所以 QUIC 不会出现队头阻塞问题。

2. 更快的握手：QUIC 优化了建立连接的握手过程，且由于 QUIC 内部包含了 TLS，所以 QUIC 仅需要一个握手过程即可建立连接。有效负载数据可以与握手数据一起发送，做到了 0-RTT 的握手。

3. 连接迁移：QUIC 的连接是由 Connection ID 确定的，这意味着即使 IP 地址或者端口变动了，只要 Connection ID 不变，就可以保持连接。然而这在 TCP 中是做不到的，因为 TCP 一个连接标识是一个四元组：源 IP、源端口、目的 IP、目的端口，只要其中一个发生变化，就需要重新建立连接。

### HTTP/3.0

HTTP/3.0 是基于 QUIC 的 HTTP 协议，所以它包含了 QUIC 的所有特点。

## 参考

- [HTTP 常见面试题](https://www.xiaolincoding.com/network/2_http/http_interview.html)（文章）

- [QUIC 协议详解](https://zhuanlan.zhihu.com/p/405387352)（文章）
