---
title: 'TCP 的常见拥塞控制算法学习笔记'
draft: false
date: 2024-06-27T13:54:31+08:00
description: ''
author: 'Cassius0924'
tags: ["TCP", "Congestion Control", "Algorithm", "Network", "Notes"]
---

# TCP 的常见拥塞控制算法学习笔记

TCP 的拥塞控制算法有几种：

- Tahoe

- Reno

- NewReno

- SACK

- BIC

- CUBIC

- BBR

> [!NOTE]
>
> MSS: Maximum Segment Size，最大分段大小。
>
> cwnd: Congestion Window，拥塞窗口，单位为 MSS。
>
> ssthresh: Slow Start Threshold，慢启动阈值，单位为 MSS。
>
> RTO: Retransmission TimeOut，重传超时时间。

## Tahoe

Tahoe 只有两种状态：慢启动和拥塞避免。

![Tahoe](https://s2.loli.net/2024/06/27/oDsi6fncYx9Vadu.png)

慢启动阶段：`cwnd = 1; ssthresh = 65535;`。

每次收到一个 ACK，`cwnd = cwnd + 1;`，所以一个 RTT 内 `cwnd` 会翻倍，`cwnd` 呈指数增长。

拥塞避免阶段：`cwnd = cwnd + 1;`。

每次收到一个 ACK，`cwnd = cwnd + 1 / cwnd;`，所以每个 RTT 内 `cwnd` 只会增加 1，`cwnd` 呈线性增长。

触发事件：RTO 超时 或 3 个冗余 ACK。

进入慢启动状态，`cwnd = 1; ssthresh = cwnd / 2;`。

Tahoe 算法还有一个快速重传机制，当收到 3 个冗余 ACK 时，立即重传丢失的数据包。但无论是重传还是超时，都会将 `ssthresh` 设置为 `cwnd / 2`，并将 `cwnd` 设置为 1。反应很激烈，所以效率不高。

![Tahoe](https://s2.loli.net/2024/06/27/MgunrEARJC9mkcf.png)

## Reno

Reno 在 Tahoe 的基础上增加了快速恢复机制。

![Reno](https://s2.loli.net/2024/06/27/kEyoLCBlq3HmzKx.jpg)

触发事件：3 个冗余 ACK。

进入快速恢复状态，`ssthresh = cwnd / 2; cwnd = ssthresh + 3;`。

在快速恢复状态下，每收到一个冗余 ACK，`cwnd = cwnd + 1`，直到收到新的 ACK，证明原来丢失的数据包已经被接收，回到拥塞避免状态，并且将 `cwnd` 设置为 `ssthresh`。

触发事件：RTO 超时。

和 Tahoe 一样，Reno 在 RTO 超时时，会将 `ssthresh` 设置为 `cwnd / 2`，并将 `cwnd` 设置为 1。

相较于 Tahoe，Reno 在收到 3 个冗余 ACK 时，不会将 `cwnd` 设置为 1，而是设置为 `ssthresh + 3`，这样可以更快地恢复拥塞窗口。

## NewReno

NewReno 在 Reno 的基础上优化了快速恢复机制。在网络中只有一个丢包时，NewReno 算法和 Reno 算法的表现是一样的。但在网络中有多个丢包时，NewReno 算法可以更快地恢复拥塞窗口。 

`PACK`：Partial ACK，部分 ACK。

`RACK`：Recovery ACK，恢复 ACK。

## SACK

SACK：Selective Acknowledgment，选择性确认。

SACK 是一种可选的 TCP 选项，可以在 TCP 连接建立时协商是否使用 SACK。旨在解决 NewReno 一次 RTT 只能恢复一个丢包的问题。

SACK 在 Reno 的基础上增加了选择确认机制和选择重传机制。

通过 `left_edge` 和 `right_edge` 来标记已经接收到的数据包，当收到冗余 ACK 时，可以根据 SACK 选项来选择重传哪些数据包。

## BIC

BIC 是一种新的拥塞控制算法，可以更好地适应高速网络。简单来说是使用了二分搜索来寻找最佳拥塞窗口。

## CUBIC

优化后的 BIC 算法。

## BBR

BBR 是谷歌开发的一种拥塞控制算法，可以更好地适应高速网络和高延迟网络。它不再是基于丢包的拥塞控制算法，而是基于带宽和延迟的拥塞控制算法。

## 参考

- [TCP拥塞控制算法之NewReno和SACK](https://blog.csdn.net/m0_38068229/article/details/80417503)（文章）

- [拥塞控制算法（从Tahoe到PCC Vivace）](https://blog.csdn.net/weixin_45662974/article/details/122400338)（文章）
