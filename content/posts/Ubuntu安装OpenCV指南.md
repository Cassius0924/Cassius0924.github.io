---
title: 'Ubuntu 安装 OpenCV 指南'
draft: false
date: 2023-03-24T10:00:00+08:00
description: '本文是一个简单易用的Ubuntu安装OpenCV的指南，帮助用户轻松完成OpenCV的安装和配置。'
author: 'Cassius0924'
tags: ["Ubuntu", "OpenCV", "Installation", "Tutorial", "Computer Vision"]
---

# Ubuntu 安装 OpenCV 指南

![OpenCV](https://s2.loli.net/2023/03/16/HsmExtPf2vRegYG.png)

本文是一个简单易用的Ubuntu安装OpenCV的指南，帮助用户轻松完成OpenCV的安装和配置。

## 安装步骤

### 第一步：安装依赖项

```shell
sudo apt-get install cmake git build-essential libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev
```

> 一口气全安装。



###第二步：下载OpenCV源代码

- GitHub下载：

  - 切换到用户目录（也可以选择其他文件夹，本文以用户目录`~/`为例）：

  ```shell
  cd ~
  ```
  - 下载源码：

  ```shell
  git clone https://github.com/opencv/opencv.git
  cd opencv
  ```

  - 可以根据需要替换为其他版本号，建议使用最新版：

  ```shell
  git checkout 4.7.0
  ```

- OpenCV官网下载：

	- 若Git速度慢，也可以选择在[OpenCV官网](https://opencv.org/releases/)下载源码压缩包：
	  ![OpenCV Releases](https://s2.loli.net/2023/03/16/xP4Ve5OW63w2DsU.png)
	
	- 下载完成后将压缩包移到用户目录`~/`文件夹下，解压文件：
	
	  ```shell
	  unzip -q opencv-4.7.0.zip
	  cd opencv-4.7.0
	  ```
	
	  > 注意，不同版本的文件名不同。

 ### 第三步：编译和安装OpenCV

- 新建并进入`build`文件夹：

	```shell
	mkdir build && cd build
	```

- 编译OpenCV：
	```shell
	cmake ..
	make -j8
	```

- 安装OpenCV：
	```shell
	sudo make install
	```

### 第四步：配置OpenCV

- 获取`lib`文件夹路径：
	```shell
	cd lib && pwd
	```
	复制输出的路径 。

- 修改配置文件：
	
	将刚刚复制的路径粘贴在引号之间。
	
	```shell
	sudo echo 'include lib文件夹路径' >> /etc/ld.so.conf
	```
	
- 更新配置文件：
	```shell
	sudo ldconfig
	```

### 第五步：验证OpenCV
- 查看版本号：
	```shell
	pkg-config --modversion opencv4
	```

- 查看libs库：
	```shell
	pkg-config --libs opencv4
	```

如果能够输出OpenCV版本号和libs库，则表明安装成功。



## 测试程序

- 进入测试样例文件夹：

  ```shell
  cd ~/opencv-4.7.0/samples/cpp/example_cmake
  ```

- 依次执行以下命令：

  ```shell
  mkdir build && cd build
  cmake ..
  make
  ```

- 运行程序：
	```shell
  ./opencv_example
  ```
  
  正常运行时显示摄像头画面，窗口左上角显示Hello OpenCV。



