---
layout: '../../layouts/MarkdownPost.astro'
title: '构建 Open3D 官方 Azure Kinect 示例时找不到 AzureKinect*** 的解决方法'
pubDate: 2023-04-08
description: '此文章旨在帮助解决C++版Open3D编译时找不到AzureKinect相关函数的问题。'
author: 'Cassius0924'
cover:
    url: 'https://s2.loli.net/2023/04/08/Urlin4NQLEc3kyV.png'
    square: 'https://s2.loli.net/2023/04/08/Urlin4NQLEc3kyV.png'
    alt: 'Azure Kinect with Open3D'
tags: ["Linux", "Ubuntu", "Open3D", "Kinect"]
theme: 'light'
featured: ture
---

# 构建 Open3D 官方 Azure Kinect 示例时找不到 AzureKinect*** 的解决方法

![Azure Kinect with Open3D](https://s2.loli.net/2023/04/08/Urlin4NQLEc3kyV.png)

此文章旨在帮助解决 C++ 版 Open3D 编译时找不到 AzureKinect 相关函数的问题。

## 问题描述

在尝试构建官方Azure Kinect示例时（`AzureKinectViewer.cpp`、`AzureKinectMKVReader.cpp`和`AzureKinectRecord.cpp`）报错：

```bash
[1/1] Linking CXX executable AzureKinectViewer
FAILED: AzureKinectViewer 
: && /usr/bin/c++   CMakeFiles/AzureKinectViewer.dir/AzureKinectViewer.cpp.o -o AzureKinectViewer -L/usr/local/lib   -L/usr/local/cuda/lib64 -Wl,-rpath,/usr/local/lib:/usr/local/cuda/lib64  /usr/lib/aarch64-linux-gnu/libk4a.so.1.4.1  /usr/local/lib/libOpen3D.so && :
CMakeFiles/AzureKinectViewer.dir/AzureKinectViewer.cpp.o: In function `main':
AzureKinectViewer.cpp:(.text+0x4fc): undefined reference to `open3d::io::AzureKinectSensor::ListDevices()'
AzureKinectViewer.cpp:(.text+0x50c): undefined reference to `open3d::io::AzureKinectSensorConfig::AzureKinectSensorConfig()'
AzureKinectViewer.cpp:(.text+0x75c): undefined reference to `open3d::io::AzureKinectSensor::AzureKinectSensor(open3d::io::AzureKinectSensorConfig const&)'
AzureKinectViewer.cpp:(.text+0x76c): undefined reference to `open3d::io::AzureKinectSensor::Connect(unsigned long)'
AzureKinectViewer.cpp:(.text+0x84c): undefined reference to `open3d::io::AzureKinectSensor::CaptureFrame(bool) const'
AzureKinectViewer.cpp:(.text+0x93c): undefined reference to `open3d::io::AzureKinectSensor::~AzureKinectSensor()'
AzureKinectViewer.cpp:(.text+0xb2c): undefined reference to `open3d::io::AzureKinectSensor::~AzureKinectSensor()'
CMakeFiles/AzureKinectViewer.dir/AzureKinectViewer.cpp.o: In function `open3d::io::AzureKinectSensorConfig::~AzureKinectSensorConfig()':
AzureKinectViewer.cpp:(.text._ZN6open3d2io23AzureKinectSensorConfigD2Ev[_ZN6open3d2io23AzureKinectSensorConfigD5Ev]+0xc): undefined reference to `vtable for open3d::io::AzureKinectSensorConfig'
AzureKinectViewer.cpp:(.text._ZN6open3d2io23AzureKinectSensorConfigD2Ev[_ZN6open3d2io23AzureKinectSensorConfigD5Ev]+0x10): undefined reference to `vtable for open3d::io::AzureKinectSensorConfig'
collect2: error: ld returned 1 exit status
ninja: build stopped: subcommand failed.
```

这是因为安装Open3D时未开启Azure Kinect的支持选项。



## 解决方法

重新安装Open3D。找到Open3D源码文件夹，执行以下命令：

```bash
cd build
```

开启Azure Kinect支持的构建选项：

```bash
sudo cmake -DBUILD_AZURE_KINECT=ON ..
```

编译安装：

```bash
sudo make
sudo make install
```







