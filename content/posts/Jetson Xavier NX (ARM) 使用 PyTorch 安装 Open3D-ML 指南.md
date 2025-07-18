---
title: 'Jetson Xavier NX (ARM) 使用 PyTorch 安装 Open3D-ML 指南'
draft: false
date: 2024-06-16T11:20:31+08:00
description: ''
author: 'Cassius0924'
tags: ["Jetson", "Open3D-ML", "PyTorch", "Installation", "Tutorial", "ARM"]
---

#  Jetson Xavier NX (ARM) 使用 PyTorch 安装 Open3D-ML 指南

由于 Jetson 为 ARM64 (aarch64) 的系统架构，所以不能用` pip install` 直接安装，需要通过源码编译。



## 升级系统 JetPack

由于 Open3D-ML 目前只支持 CUDA 10.0 以及 CUDA 11.*，并且  JetPack 的 CUDA 开发环境只有10.2、11.4以及12.2，所以我们只能选择 CUDA 11.4 进行安装。

使用 `jtop`命令查看 JetPack 版本。如果 Jetson 系统的 JetPack 低于 5.1.2 则需要通过 SDK Manager 升级到 JetPack 5.1.2。

如果你的系统已经是 JetPack 5.1.2 那我也推荐你通过 SDK Manager 重新安装一遍，排除难以发现的错误。

详细指南参考文章[Jetson Xavier NX 升级或重新安装 JetPack 指南]。



## 安装 PyTorch

安装教程参考文章[Jetson Xavier NX 安装 CUDA 支持的 Pytorch 指南]。

注意，PyTorch 的 CUDA 支持版本需要和 Open3D-ML 的一致，同为 CUDA 11.4。否则与最后安装 Open3D-ML 的时候会报错：

![Open3D-ML Pytorch CUDA Version dont match](https://s2.loli.net/2023/10/12/JKcUrdEhyHk51IO.png)

## 下载源码

### 下载 Open3D 源码

如果你之前下载过则不需要再下载：

```bash
git clone https://github.com/isl-org/Open3D.git
```

### 下载 Open3D-ML 源码

```bash
git clone https://github.com/isl-org/Open3D-ML.git
```

## 前提准备

安装依赖：

```bash
cd Open3D
bash util/install_deps_ubuntu.sh
```

```bash
pip install yapf
```

重新链接 6.0.29 的 libstdc++.so，否则会出现`undefined reference to 'std::__throw_bad_array_new_Length()@GLIBCXX_3.4.29'`的错误。

```bash
ln -sf /usr/local/lib64/libstdc++.so.6.0.29 /lib/aarch64-linux-gnu/libstdc++.so.6
```



## 编译安装

国际惯例，先创建一个 build 文件夹。

```bash
mkdir build
cd build
```

```bash
sudo cmake \
-DBUILD_CUDA_MODULE=ON \
-DBUILD_PYTORCH_OPS=ON \
-DBUILD_TENSORFLOW_OPS=OFF \
-DBUILD_EXAMPLES=OFF \
-DBUILD_SHARED_LIBS=ON \
-DCMAKE_BUILD_TYPE=Release \
-DGLIBCXX_USE_CXX11_ABI=ON \
-DBUILD_PYTHON_MODULE=ON \
-DBUNDLE_OPEN3D_ML=ON \
-DCUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda \
-DPython3_EXECUTABLE=/path/to/python \
-DOPEN3D_ML_ROOT=../../Open3D-ML \
..
```

`-DBUILD_PYTORCH_OPS`和`-DBUILD_TENSORFLOW_OPS`分别为构建 PyTorch 版和构建 Tensorflow 版选项，两个选择可以同时为`ON`，本指南只构建 PyTorch。

`-DBUILD_EXAMPLES`表示是否构建官方示例，这里设置`OFF`不构建，可以缩短构建时间。

如果没有使用 Python 虚拟环境，则不需要`-DPython3_EXECUTABLE`；如果使用了 Python 虚拟环境，需要改成虚拟环境的 Python 的路径，使用`which python`查看路径。

`-DOPEN3D_ML_ROOT`为刚刚下载的 Open-ML 的源代码路径。

编译：

```bash
make -j$(nproc)
```

编译过程需要数个小时的时间。

编译完毕后需要添加一个环境变量：

```bash
vim ~/.zsh #或者 ~/.bashrc
```

在文件末尾添加一行：

```bash
export LD_PRELOAD=/usr/local/lib/libOpen3D.so
```

安装 Open3D C++ 库：

```bash
make install
```

安装 Open3D 与 Open3D-ML Python 库：

```bash
make install-pip-package -j$(nproc)
```

## 错误解决

### undefined reference to '***' @GLIBCXX_3.4.29

这个错误问题出在系统的 libstdc++ 版本不够高。尝试使用`strings /lib/aarch64-linux-gnu/libstdc++.so.6 | grep GLIBCXX`命令查看 libstdc++ 是否支持 GLIBCXX_3.4.29，如果缺少 GLIBCXX_3.4.29 则请尝试软链接 libstdc++.so.6 到正确版本。

```bash
ln -sf /usr/local/lib64/libstdc++.so.6.0.29 /lib/aarch64-linux-gnu/libstdc++.so.6
```

如果提示找不到 libstdc++.so.6.0.29 文件，则请升级 libstdc++ 库。

![undefined reference to GLIBCXX_3.4.29](https://s2.loli.net/2023/10/14/rpVu1exNK9ZQSYq.jpg)

### CMAKE_CUDA_ARCHITECTURES now detected for NVCC

检查系统默认 CUDA 版本是否存在 nvcc 二进制文件。

```bash
sudo stat /usr/local/cuda/bin/nvcc
```

若提示不存在文件，则 CUDA 开发环境已经损坏，需要重新安装 JetPack。详细参考文章 [Jetson Xavier NX 升级或重新安装 JetPack 指南]。

### /lib/Release/libOpen3D.so: cannot allocate memory in static TLS block

```bash
vim ~/.zsh #或者 ~/.bashrc
```

在文件末尾添加一行：

```bash
export LD_PRELOAD=/usr/local/lib/libOpen3D.so
```

### ModuleNotFoundError: No module named ‘jupyter_packaging’

```bash
pip install jupyter-packaging
```

### ModuleNotFoundError: No module named ‘ipywidgets’

```bash
pip install ipywidgets
```

### Open3D was built with CUDA 11.4 but PyTorch was built with CUDA 10.2

这个警告表示，PyTorch 的 CUDA 版本和 Open3D-ML 的 CUDA 版本不一致。需要使用与 Open3D-ML 一致的 CUDA 版本重新构建安装 PyTorch。详细请参考文章[Jetson Xavier NX 安装 CUDA 支持的 Pytorch 指南]。

![8481697106387_.pic](https://s2.loli.net/2023/10/14/yrIhgb95SclpjKM.jpg)

## 测试验证

```bash
python3 -c "import open3d.ml.torch"
```

若无任何输出，则表示安装成功。
