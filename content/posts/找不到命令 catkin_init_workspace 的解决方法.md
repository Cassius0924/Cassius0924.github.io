---
title: '找不到 catkin_init_workspace 命令的解决方法'
draft: false
date: 2023-03-28T10:00:00+08:00
description: '本文旨在解决找不到 catkin_init_workspace 命令的问题。'
author: 'Cassius0924'
tags: ["Linux", "ROS"]
---

#找不到 catkin_init_workspace 命令的解决方法

![ROS Logo](https://s2.loli.net/2023/03/28/TPrkdjpvx2bwXMn.png)

本文提供了一个简单的解决方案，帮助你解决在ROS工作空间中使用`catkin_init_workspace`命令时出现的 command not found 的问题。

## 问题描述

在建立 ROS 工作空间的时候发现找不到初始化工作空间命令：

```bash
catkin_init_workspace
```

![catkin_init_workspace](https://s2.loli.net/2023/03/28/ln4ZiGTIFgDCVRo.png)

但是检查后发现明明已经安装了 ROS-melodic-catkin，应该可以用初始化命令才对：

![dpkg](https://s2.loli.net/2023/03/28/RBjeTduYykgtXUE.png)

最后在 ROS 官网找到答案，问题出在**环境配置**上。

## 解决方法

我们只需要 source 一下环境配置脚本即可：

```bash
source /opt/ros/melodic/setup.sh
```

> 注意：不同版本 ROS 的 setup.sh 脚本的文件路径不同。Noetic 版 ROS 的 setup.sh 脚本在 `/opt/ros/noetic`下。

执行以下命令可以在每次启动新的 Shell 窗口时自动的 source 这个脚本：

```bash
echo "source /opt/ros/melodic/setup.sh" >> ~/.bashrc
source ~/.bashrc
```

如果你用的是 Zsh，则执行以下命令：

```bash
echo "source /opt/ros/melodic/setup.sh" >> ~/.zshrc
source ~/.zshrc
```

