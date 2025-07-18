---
title: '解决 Open3D 同时链接其他库时的 Undefined Reference 错误'
draft: false
date: 2023-06-09T10:00:00+08:00
description: '本文旨在解决C++ Open3D库与其他库冲突问题。'
author: 'Cassius0924'
tags: ["Linux", "Open3D"]
---

# 解决 Open3D 同时链接其他库时的 Undefined Reference 错误

当你的 Open3D 项目同时使用了 OpenCV 或 Protobuf 等其他库时，在链接库时可能会出现 Undefined Reference 的错误。这是因为 Open3D 默认使用的 C++ ABI 版本与其他库不一致导致的。

为了解决这个问题，可以在重新编译安装 Open3D 时打开 `-DGLIBCXX_USE_CXX11_ABI=ON` 选项，即使用 C++11 ABI 版本。以下是具体的步骤：

## 解决方法

### 找到 Open3D 源码

```bash
cd open3d
```

找不到请在 Github 上重新下载。

### 重新编译安装 Open3D

进入 Open3D 的源代码目录的 `build` 子目录进行编译安装。

```bash
cd build
```

在 `build` 子目录中执行 CMake 命令生成 Makefile。**在命令行中添加 `-DGLIBCXX_USE_CXX11_ABI=ON` 选项。**

```bash
cmake .. -DBUILD_SHARED_LIBS=ON -DGLIBCXX_USE_CXX11_ABI=ON -DCMAKE_BUILD_TYPE=Release
```

最后，执行 `make` 命令编译并安装 Open3D。

```bash
make -j6
sudo make install
```

### 使用 Open3D

重新编译安装后的 Open3D 就能够正常链接其他库了。
