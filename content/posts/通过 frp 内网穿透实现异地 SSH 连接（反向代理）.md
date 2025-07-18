---
title: '通过 frp 内网穿透实现异地 SSH 连接（反向代理）'
draft: false
date: 2024-06-16T11:20:31+08:00
description: ''
author: 'Cassius0924'
tags: ["FRP", "SSH", "Reverse Proxy", "NAT Traversal", "Tutorial"]
---

# 通过 frp 内网穿透实现异地 SSH 连接（反向代理）

起因是我放假回家，想在家里通过 SSH 连接放在学校的无显示器的 Linux，但是学校的 Linux 是内网，无法直接连接，且无显示器无法使用向日葵等远程桌面软件，所以想到了使用 frp 的反向代理功能实现内网穿透，进而实现异地 SSH 连接。

## 前提
- 一台具有公网 IP 的服务器（阿里云、腾讯云等）

## 配置远程主机
我们需要有三台主机，分别是：自己的电脑、远程 Linux 主机和具有公网 IP 的服务器。

只需要在远程 Linux 和具有公网 IP 的服务器上配置 frp 即可。

首先在远程 Linux 上下载 frp，[Github 下载地址](https://github.com/fatedier/frp/releases)。下载远程主机对应的版本，我这里是 ARM64 架构的 Linux，所以下载 `frp_0.51.2_linux_arm64.tar.gz`。

![frp GitHub](https://s2.loli.net/2023/08/01/P5MzmKDEeBjdJF3.png)



下载完毕后解压：
```shell
tar -xvf frp_0.51.2_linux_arm64.tar.gz
cd frp_0.51.2_linux_arm64
```

远程 Linux 为客户端，所以只需要保留 `frpc*` 文件即可，`frps`可以删除。

```shell
rm frps*
```

修改`frpc.ini`，只需要将`server_addr`修改为服务器的 IP 地址即可，`local_ip`不变。`server_port`和`remote_port`一般不变，若与其他服务冲突了可以修改。

```shell
vim frpc.ini	#:wq 退出
```

![vim frpc.ini](https://s2.loli.net/2023/08/01/r6dks5YSEm9WUXl.png)

## 配置服务器

同样下载好对应系统版本的 frp，解压后删除`frpc*`文件。

```shell
rm frpc*
```

修改`frpc.ini`，确保`bind_port`与 frpc 客户端，即远程主机的`server_port`一致。

再前往阿里云或腾讯云官网配置服务器防火墙规则，开放服务器的 6000 和 7000 端口（若修改了则开放修改后的端口）。

![aliyun](https://s2.loli.net/2023/08/01/gfIVoMU2mlDJY6t.png)

## 启动 frp

先启动服务器的`frps`：

```shell
./frps -c frps.ini
```

显示以下三行则代表启动成功：

![./frps](https://s2.loli.net/2023/08/01/veKSDbOCPAGoHVj.png)

再启动远程主机的`frpc`：

```shell
./frpc -c frpc.ini
```

## 连接远程主机

启动成功后就可以尝试 SSH 连接远程 Linux 主机了：

```shell
ssh -p <server_port> <user>@<server_addr>
```

其中，<server_port>为上面`frpc.ini`里的`server_port`，<server_addr>为上面`frpc.ini`里的`server_port`，<user>为远程 Linux 的用户名。
