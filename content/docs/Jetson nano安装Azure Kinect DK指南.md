---
layout: '../../layouts/MarkdownPost.astro'
title: 'Jetson nano 安装 Azure Kinect DK 指南'
pubDate: 2023-03-24
description: '这是一个简单教程，旨在帮助大家实现免密SSH登录，省去每次输入用户名和密码的烦恼。'
author: 'Cassius0924'
cover:
    url: 'https://s2.loli.net/2023/03/24/HZq6GLmaM839s7b.jpg'
    square: 'https://s2.loli.net/2023/03/24/HZq6GLmaM839s7b.jpg'
    alt: 'Azure Kinect'
tags: ["Linux", "Ubuntu", "Jetson", "Azure Kinect", "Microsoft"]
theme: 'light'
featured: ture
---

# Jetson nano 安装 Azure Kinect DK 指南

![Azure Kinect](https://s2.loli.net/2023/03/24/IwUxHR56CdVWjOZ.jpg)

该项目提供了一个简单的指南，帮助用户在 NVIDIA Jetson Nano 上正确安装 Azure Kinect DK。

Jetson nano是ARM64架构，而非AMD64架构。所以环境配置起来会和AMD64架构的有所不同。

## 前提

在开始安装 Azure Kinect DK 之前，请确保您的 Jetson Nano 满足以下要求：

- 版本为Ubuntu 18.04 LTS
- 已安装cURL

> 不再费文笔说明如何安装上述工具。

## 安装步骤

### 第一步：配置微软软件包储存库

> 系统版本必须为Ubuntu 18.04，其他版本参考[Microsoft 产品的 Linux 软件存储库](https://learn.microsoft.com/zh-cn/windows-server/administration/linux-package-repository-for-microsoft-software)。

依次运行下列命令：

- 添加微软GPG公钥：

  ```shell
  curl -sSL https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
  ```

- 添加微软软件包源：

	```shell
	sudo apt-add-repository https://packages.microsoft.com/ubuntu/18.04/prod
	```
	
- 下载最新软件包列表：
	```shell
	sudo apt-get update
	```

### 第二步：安装依赖项

>  依赖项需要手动安装ARM64版本的。

- 安装libk4a

	```shell
	curl -O https://packages.microsoft.com/ubuntu/18.04/multiarch/prod/pool/main/libk/libk4a1.4/libk4a1.4_1.4.1_arm64.deb && sudo dpkg -i libk4a1.4_1.4.1_arm64.deb
	```

- 安装libk4a-dev

  ```shell
  curl -O https://packages.microsoft.com/ubuntu/18.04/multiarch/prod/pool/main/libk/libk4a1.4-dev/libk4a1.4-dev_1.4.1_arm64.deb && sudo dpkg -i libk4a1.4-dev_1.4.1_arm64.deb
  ```

### 第三步：安装 k4a-tools

> k4a-tools也需要手动安装ARM64版本的。

```shell
curl -O https://packages.microsoft.com/ubuntu/18.04/multiarch/prod/pool/main/k/k4a-tools/k4a-tools_1.4.1_arm64.deb && sudo dpkg -i k4a-tools_1.4.1_arm64.deb
```

### 第四步：插上 Azure Kinect 设备

将 Azure Kinect 插到 Jetson nano 上，注意必须使用支持 USB3.0 的数据线并插到 USB 3.0 的接口上，否则程序会报错。

### 第五步：运行 k4a_viewer

必须使用`sudo`运行，否则程序找不到设备。

```bash
sudo k4aviewer
```

