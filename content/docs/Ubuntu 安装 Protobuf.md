---
layout: '../../layouts/MarkdownPost.astro'
title: 'Ubuntu 安装 Protobuf 指南'
pubDate: 2023-06-12
description: '本指南将介绍如何在 Ubuntu 上安装 Protobuf。'
author: 'Cassius0924'
cover:
    url: 'https://s2.loli.net/2023/06/12/VSB1moNvdrglhnu.png'
    square: 'https://s2.loli.net/2023/06/12/VSB1moNvdrglhnu.png'
    alt: 'Protobuf'
tags: ["Linux", "Protobuf"]
theme: 'light'
featured: ture
---

# Ubuntu 安装 Protobuf 指南

Protobuf（Protocol Buffers）是一种轻量级的数据交换格式，常用于高效地序列化结构化数据。本指南将介绍如何在 Ubuntu 上安装 Protobuf。

## 步骤 1：更新系统

在安装 Protobuf 之前，我们首先需要确保系统已经更新到最新版本。打开终端并执行以下命令：

```shell
sudo apt update
sudo apt upgrade
```

这将更新系统的软件包并安装最新的安全补丁。

## 步骤 2：安装编译工具和依赖项

在安装 Protobuf 之前，我们需要安装一些编译工具和依赖项。执行以下命令来安装它们：

```shell
sudo apt install build-essential autoconf libtool
```

这些工具和依赖项将帮助我们编译和构建 Protobuf。

## 步骤 3：下载和编译 Protobuf

1. 首先，我们需要下载 Protobuf 的源代码。这里选择下载[v3.20.3版本的Protobuf源码压缩包](https://github.com/protocolbuffers/protobuf/releases/tag/v3.20.3)。（必须下载-all压缩包）

2. 解压压缩包

```shell
tar -zxvf protobuf-all-3.20.3.tar.gz
```

这将克隆 Protobuf 的源代码到当前目录。

3. 进入克隆下来的 Protobuf 目录：

```shell
cd protobuf-all-3.20.3
```

4. 在源代码目录中，运行以下命令来生成配置文件和构建系统：

```shell
./autogen.sh
```

5. 接下来，我们需要运行 `configure` 脚本来配置编译选项。可以使用以下命令进行配置：

```shell
./configure
```

6. 配置完成后，我们可以使用以下命令编译和安装 Protobuf：

```shell
sudo make
sudo make check #这一步可能会报错，解决方法见下文
sudo make install
sudo ldconfig
```

编译过程可能需要一些时间，请耐心等待。

## 步骤 4：验证安装

安装完成后，我们可以验证 Protobuf 是否成功安装。执行以下命令来检查 Protobuf 的版本信息：

```shell
protoc --version
```

如果安装成功，将显示 Protobuf 的版本号。

## 解决 make check 报错
在执行 `make check` 时，可能会报错，错误信息如下：

```shell
FAIL: protobuf-test
PASS: protobuf-lazy-descriptor-test
PASS: protobuf-lite-test
PASS: google/protobuf/compiler/zip_output_unittest.sh
PASS: google/protobuf/io/gzip_stream_unittest.sh
PASS: protobuf-lite-arena-test
PASS: no-warning-test
============================================================================
Testsuite summary for Protocol Buffers 3.19.4
============================================================================
# TOTAL: 7
# PASS:  6
# SKIP:  0
# XFAIL: 0
# FAIL:  1
# XPASS: 0
# ERROR: 0
============================================================================
See src/test-suite.log
Please report to protobuf@googlegroups.com
============================================================================
```
![Make Check Error](https://s2.loli.net/2023/06/13/N7k5xDE9AfhUTeM.png)

`protobuf-test` 测试不通过，这个错误是因为加载大型数据集时内存溢出导致的。

解决方法是加大可用内存或增加内存 Swap 分区，这里以 Jetson Xavier NX 增加内存 Swap 分区为例，此前请先确保已经安装 Jtop 工具：

```shell
sudo jtop
```

先切换到4号内存面板，接着点击右边的 Create new 按钮，点击三次，创建三个内存 Swap 分区即可。

![Jtop Men Panle](https://s2.loli.net/2023/06/13/GIaNmi436kAXz5l.png)

![Jtop Swapfile](https://s2.loli.net/2023/06/13/siFRxYDvQl3toAS.png)

最后按`Q`退出 Jtop 工具。

重新执行 `make check` 即可。

![Make Check Agian](https://s2.loli.net/2023/06/13/Vx5liabgdEqA4DK.png)

