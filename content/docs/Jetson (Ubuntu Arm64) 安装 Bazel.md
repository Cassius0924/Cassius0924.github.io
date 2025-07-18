---
layout: '../../layouts/MarkdownPost.astro'
title: 'Jetson (Ubuntu Arm64) 安装 Bazel'
pubDate: 2023-06-14
description: '本指南帮助 ARM64 架构的 Jetson 安装 Bazel'
author: 'Cassius0924'
cover:
    url: 'https://s2.loli.net/2023/06/14/pdJIcoZsxq9Ou27.png'
    square: 'https://s2.loli.net/2023/06/14/pdJIcoZsxq9Ou27.png'
    alt: 'Bazel'
tags: ["Linux", "Ubuntu", "Jetson", "Bazel"]
theme: 'light'
featured: ture
---

# Jetson (Ubuntu Arm64) 安装 Bazel

## 简介
本文旨在帮助用户在 Jetson 上的 Ubuntu Arm64 系统上安装 Bazel。Bazel 是一个开源的构建工具，它专注于构建和测试大型软件项目，并且被广泛应用于机器学习和深度学习领域。通过使用 Bazel，您可以更高效地管理和构建您的项目。

## 步骤 1：安装OpenJDK
在开始安装 Bazel 之前，我们需要安装 OpenJDK。在终端中执行以下命令：

```shell
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install openjdk-11-jdk
```

## 步骤 2：下载 Bazel 安装包
在安装 OpenJDK 之后，我们需要下载 Bazel 的安装包。在终端中执行以下命令：

```shell
wget https://github.com/bazelbuild/bazel/releases/download/6.2.1/bazel-6.2.1-dist.zip
```

或者，您也可以从[ Bazel 的 Github 仓库](https://github.com/bazelbuild/bazel/releases)下载最新版本的安装包。（必须下载dist.zip文件）

## 步骤 3：安装 Bazel
下载完成后，我们可以使用以下命令来安装 Bazel：

```shell
unzip bazel-6.2.1-dist.zip -d bazel-6.2.1
bash./compile.sh
sudo cp output/bazel /usr/local/bin
```

## 步骤 4：验证安装
安装完成后，我们可以验证 Bazel 是否成功安装。在终端中执行以下命令：

```shell
bazel version
```

如果一切正常，您应该能够看到如下输出：
![bazel version](https://s2.loli.net/2023/06/14/emMDIGbSiWsKgo9.png)


