---
layout: '../../layouts/MarkdownPost.astro'
title: 'Linux 命令行网络连接指南'
pubDate: 2023-04-14
description: '本文旨在为 Linux 用户介绍提供关于使用命令行连接网络的方法。'
author: 'Cassius0924'
cover:
    url: 'https://s2.loli.net/2023/04/14/ZthLdpQJjDc2XGC.png'
    square: 'https://s2.loli.net/2023/04/14/ZthLdpQJjDc2XGC.png'
    alt: 'Network Manager'
tags: ["Linux", "Network", "CLI"]
theme: 'light'
featured: ture
---

# Linux 命令行网络连接指南

本文旨在为 Linux 用户介绍提供关于使用命令行连接网络的方法。 本文将详细介绍利用 nmcli 工具连接 Wi-Fi 和以太网的教程，包括查看连接状态、控制 Wi-Fi、控制以太网等等。

## 安装 nmcli

nmcli 是 NetworkManager 的命令行工具，它可以用于管理和配置网络连接。如果你的 Linux 系统上没有安装 nmcli，请使用以下命令进行安装：

```bash
sudo apt-get install network-manager
```

## 查看连接状态

在连接网络之前，我们需要先查看网络状态。使用以下命令可以查看当前网络状态：

```bash
nmcli general status
```

> 以上命令可以简写为：
>
> ```bash
> nmcli g   # g 表示 general，默认为 status
> ```

![nmcli general status](https://s2.loli.net/2023/04/14/H59AWqixYr4O3eC.png)

- `STATE`：这是 NetworkManager 的当前状态。它可以是 "connected"、"connecting"、"disconnected"、"disconnecting" 或 "asleep" 等。

  | Connectd | Connecting | Disconnected | Disconnecting | Asleep |
  | -------- | ---------- | ------------ | ------------- | ------ |
  | 已连接 | 连接中 | 未连接 | 断连中 | 休眠 |

- `CONNECTIVITY`：这是系统的网络连接状态。它可以是 "full"、"limited" 、"Portal" 或 "none"。

  | Full | Limited | Portal | None |
  | ---- | --- | --- | --- |
  | 已联网，可上网 | 已联网，但不可上网 | 已联网，但需要认证 | 未联网 | 

- `WIFI-HW`：这是 WiFi 硬件的状态。它可以是 "enabled" 或 "disabled"。

- `WIFI`：这是 WiFi 的状态。它可以是 "enabled" 或 "disabled"。

- `WWAN-HW`：这是无线广域网（WWAN）硬件的状态。它可以是 "enabled" 或 "disabled"。

- `WWAN`：这是 WWAN 的状态。它可以是 "enabled" 或 "disabled"。

  | Enabled | Disabled |
  | --- | --- |
  | 已启用 | 未启用 |


## 打开和关闭 Wi-Fi

如果你的 Linux 系统支持 Wi-Fi，你可以使用以下命令打开或关闭 Wi-Fi：

```bash
nmcli radio wifi on  # 打开 Wi-Fi
nmcli radio wifi off # 关闭 Wi-Fi
```

> 以上命令可以简写为：
> 
> ```bash
> nmcli r w on  # r 表示 radio，w 表示 wifi
> nmcli r w off
> ```

## 查看 Wi-Fi

查看当前可用的 Wi-Fi 网络：

```bash
nmcli dev wifi
```

> 以上命令可以简写为：
>
> ```bash
> nmcli d w   # d 表示 dev，w 表示 wifi
> ```

![nmcli dew wifi](https://s2.loli.net/2023/04/14/hUM4gOPcRVJ5G3b.png)

带`*`星号的当前目前连接的Wi-Fi。

## 连接和断开 Wi-Fi

连接 Wi-Fi 网络需要知道网络名称和密码。使用以下命令连接一个 Wi-Fi 网络：

```bash
nmcli dev wifi con <network-name> password <password>
```

> 以上命令可以简写为：
>
> ```bash
> nmcli d w c <network-name> password <password>    # d 表示 dev，c 表示 con
> ```

如果连接成功，命令行将不会输出任何信息。使用以下命令查看当前连接的 Wi-Fi 网络：

```bash
nmcli con show
```

> 以上命令可以简写为：
>
> ```bash
> nmcli c   # c 表示 con，默认为 show
> ```

如果你需要断开当前连接的 Wi-Fi 网络，可以使用以下命令：

```bash
nmcli con down <connection-name>
```

其中，<connection-name> 是需要断开的连接名称。

> 以上命令可以简写为：
>
> ```bash
> nmcli c d <connection-name>   # d 表示 down
> ```

## 忘记 Wi-Fi

如果你不再需要连接到某个 Wi-Fi 网络，可以使用以下命令忘记该网络：

```bash
nmcli connection delete <connection-name>
```

其中，<connection-name> 是需要忘记的连接名称。

> 以上命令可以简写为：
> 
> ```bash
> nmcli c de <connection-name>   # de 表示 delete
> ```

## 连接以太网

如果你需要连接以太网，你可以使用以下命令连接：

```bash
nmcli dev con <interface-name>
```

其中，<interface-name> 是需要连接的网络接口名称。

> 以上命令可以简写为：
>
> ```bash
> nmcli d c <interface-name>
> ```

## 查看以太网

查看当前可用的以太网网络：

```bash
nmcli dev status
```

> 以上命令可以简写为：
>
> ```bash
> nmcli d   # d 表示 dev，默认为 status
> ```

## 断开以太网

如果你需要断开当前连接的以太网网络，可以使用以下命令：

```bash
nmcli dev discon <interface-name>
```

其中，<interface-name> 是需要断开的网络接口名称。

> 以上命令可以简写为：
>
> ```bash
> nmcli d d <interface-name>   # d 表示 discon
> ```

## 更多信息

若想了解更多关于 nmcli 的信息，可以使用`nmcli -h`命令查看帮助信息。或者访问 [NetworkManager 官方文档](https://developer.gnome.org/NetworkManager/stable/nmcli.html)。
