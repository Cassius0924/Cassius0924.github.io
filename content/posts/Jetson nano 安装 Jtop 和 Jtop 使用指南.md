---
title: 'Jetson nano 安装 Jtop 和 Jtop 使用指南'
draft: false
date: 2024-06-16T11:20:31+08:00
description: ''
author: 'Cassius0924'
tags: ["Jetson", "Jtop", "System Monitor", "Installation", "Tutorial"]
---

# Jetson nano 安装 Jtop 和 Jtop 使用指南

本文将为你介绍 Jtop，它是一个基于 Python 的系统监控工具。Jtop 通过终端界面展示系统资源的使用情况，包括 CPU、内存、磁盘、网络等。本文将详细介绍 Jtop 各个面板的作用和功能。

## 安装

先安装`pip3`

```bash
sudo apt install python3-pip
```

Jtop 可以通过 pip 来安装，您可以通过以下命令来安装：

```bash
sudo pip3 install -U jetson-stats
```

## 使用

在终端中输入以下命令来启动 Jtop：

```bash
jtop
```

在 Jtop 启动后，您可以使用键盘的左右箭头来选择面板，使用 Tab 键来切换到不同的面板，使用 Ctrl + C 命令来退出 Jtop。

## 面板介绍

### ALL 面板

ALL 面板简要展示了主板的各种信息，包括 CPU、GPU、内存、磁盘、风扇以及关于 jetson_clocks、NVPmodel 等的信息。

### GPU 面板

GPU 面板主要展示了系统 GPU 的使用情况。在 GPU 面板中，您可以看到系统当前的 GPU 利用率、GPU 温度、GPU 风扇转速、 GPU 内存使用情况等信息。同时，您也可以查看各个进程对 GPU 的使用情况。

### CPU 面板

CPU 面板主要展示了系统 CPU 的使用情况。在 CPU 面板中，您可以看到系统当前的 CPU 利用率、每个 CPU 核心的利用率、以及各个进程的 CPU 使用情况。您可以使用键盘上下箭头来选择进程，使用回车键来查看进程详情。

### MEM 面板

内存面板主要展示了系统内存的使用情况。在内存面板中，您可以看到系统当前的内存利用率、总内存、可用内存、已经使用的内存等信息。同时，您也可以查看各个进程的内存使用情况。

### ENG 面板

ENG 面板主要展示了系统的发热情况。在 ENG 面板中，您可以看到系统当前的 CPU 温度、GPU 温度、以及各个进程的 CPU 温度。

### CTRL 面板

CTRL 面板主要展示了系统的控制情况。在 CTRL 面板中，您可以看到系统当前的 CPU 频率、GPU 频率、以及各个进程的 CPU 频率。

### INFO 面板

INFO 面板主要展示了系统的信息。在 INFO 面板中，您可以看到系统当前的 IP 地址、MAC 地址、以及各个进程的 CPU 频率。

## 📝 结尾

希望本文介绍的内容能够对您使用 Jtop 有所帮助。如果您在使用 Jtop 的过程中遇到了问题，可以到项目的 Github 页面中提出 issues，我们会尽快解答您的问题。同时，我们也欢迎各位开发者为 Jtop 的开发做出贡献，感谢您的支持！
