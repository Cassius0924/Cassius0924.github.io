---
title: 'Jetson nano 安装 PCL 指南'
draft: false
date: 2023-03-24T10:00:00+08:00
description: '本指南帮助 ARM64 架构的 Jetson Nano 安装 PCL。'
author: 'Cassius0924'
tags: ["Jetson", "PCL", "Point Cloud", "Installation", "Tutorial"]
---

# Jetson nano 安装 PCL 指南

本指南帮助 ARM64 架构的 Jetson Nano 安装 PCL（点云库）。

## 安装步骤

### 第一步：安装依赖

在终端中运行以下命令，安装 PCL 所需的依赖：

```bash
sudo apt-get update
sudo apt-get install git build-essential linux-libc-dev
sudo apt-get install cmake cmake-gui
sudo apt-get install libusb-1.0-0-dev libusb-dev libudev-dev
sudo apt-get install mpi-default-dev openmpi-bin openmpi-common
sudo apt-get install libpcap-dev
sudo apt-get install libflann1.9 libflann-dev
sudo apt-get install libeigen3-dev
sudo apt-get install libboost-all-dev
sudo apt-get install vtk6 libvtk6.3 libvtk6-dev libvtk6.3-qt libvtk6-qt-dev
sudo apt-get install libqhull-dev libgtest-dev
sudo apt-get install freeglut3-dev pkg-config
sudo apt-get install libxmu-dev libxi-dev
sudo apt-get install mono-complete
sudo apt-get install libopenni-dev libopenni2-dev
sudo apt install build-essential libssl-dev
```

### 第二步：安装Eigen库

先卸载Eigen库

```bash
sudo updatedb
locate eigen3
sudo rm -rf /usr/include/eigen3 /usr/lib/cmake/eigen3 /usr/share/doc/libeigen3- dev /usr/share/pkgconfig/eigen3.pc /var/lib/dpkg/info/libeigen3-dev.list /var/lib/dpkg/info/libeigen3-dev.md5sums
```

安装：

```bash
wget https://gitlab.com/libeigen/eigen/-/archive/3.3.7/eigen-3.3.7.tar.gz
sudo tar -xvf eigen-3.3.7.tar.gz
cd eigen-3.3.7
mkdir build && cd build
sudo cmake ..
sudo make install
sudo cp -r /usr/local/include/eigen3/Eigen /usr/local/include
```

### 第三步：下载 PCL 源码

在终端中运行以下命令，下载 PCL 源码：

```bash
git clone https://github.com/PointCloudLibrary/pcl.git
```

### 第四步：编译 PCL 源码

进入 PCL 源码目录，先切换到`1.9.1`版本：

```bash
cd pcl
git checkout pcl-1.9.1
```

创建`build`文件夹：

```bash
mkdir build && cd build
```

开始编译：

```bash
cmake -DCMAKE_BUILD_TYPE=None \
-DCMAKE_INSTALL_PREFIX=/usr/local \
-DBUILD_GPU=ON \
-DBUILD_CUDA=ON \
-DBUILD_apps=ON \
-DBUILD_examples=ON ..
```

```bash
make
```

编译过程可能需要较长时间，请耐心等待。

> Jetson Nano 编译 PCL 真的很慢。

> 过程中可能遇到一些错误，解决方法见**报错解决** 。

### 第五步：安装 PCL

```bash
sudo make install
```

### 第六步：测试 PCL 安装是否成功

在终端中运行以下命令，测试 PCL 是否安装成功：

```bash
cd ../test
./all_tests
```

如果所有测试都通过，说明 PCL 安装成功。

## 报错解决

### internal compiler error: Segmentation fault (program cc1plus)

![Error: Segmentation fault](https://s2.loli.net/2023/03/22/b2WsoMnBfXatSzv.png)

此错误是 stack size 太小导致的。

使用`ulimit -a`查看系统的限制参数。

![ulimit -a](https://s2.loli.net/2023/03/22/pauXTL7FedP4xVS.png)

解决方法：增加 stack size 的大小即可。

```bash
ulimit -s 102400
```

### namespace “thrust” has no member “device_malloc”

> 完蛋，报错图片没保存 :( 大家自行脑补。

此错误是`point_cloud.h`文件缺少头文件导致的。

解决方法：找到namespace “thrust” has no member “device_malloc”这句话前面的文件，使用`vi`或`vim`打开编辑，在顶部添加一句`#include <thrust/device_malloc.h>`即可。

### Pax assembly aborted due to errors

![Ptx assembly aborted due to errors](https://s2.loli.net/2023/03/22/zI2W3ykXlJB5wCG.png)

此错误是由于PTX ISA版本6.4不支持sm_70及更高版本的目标。需要将CUDA_ARCH_BIN cmake变量限制为低于7.0的架构。

```bash
vim ../CMakeList.txt
```

添加上`set(CUDA_ARCH_BIN "6.1")`即可。

![CMakeList.txt](https://s2.loli.net/2023/03/22/kmswYOlR2MQI3fE.png)

> 这个错误在网上找了一圈没找到解决方法，最后还是 New Bing 救了我。
