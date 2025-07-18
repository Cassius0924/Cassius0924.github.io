---
title: 'Ubuntu 安装软件包时遇到 unmet dependencies 的解决方法'
draft: false
date: 2023-03-24T10:00:00+08:00
description: '本文旨在解决Ubuntu用户遇到“The following packages have unmet dependencies”错误的问题'
author: 'Cassius0924'
tags: ["Linux", "Ubuntu", "apt", "Troubleshooting", "Package Management"]
---

# Ubuntu 安装软件包时遇到 unmet dependencies 的解决方法

在 Ubuntu 中安装软件包时，有时会遇到 “The following packages have unmet dependencies” 的错误，这通常是由于缺少软件包的依赖项和软件包冲突引起的。

## 解决方法

逐个安装缺少的依赖包，直到提示删除冲突的软件包并继续安装。

### 示例

例如在安装`libvtk7.1-qt`时提示如下图所示：

![install libvtk7.1-qt](https://s2.loli.net/2023/03/18/EyZJSiPqD1G39tb.png)

执行命令：
```shell
sudo apt-get install libvtk7.1:amd64
```

> 注意：包名要完整，libvtk7.1:amd64不能少了:amd64

再次报错：

![install libvtk7.1:amd64](https://s2.loli.net/2023/03/18/YxH2EDdRI9vSM61.png)

继续安装：

```shell
sudo apt-get install libhdf5-openmpi-100:amd64
```

再次报错：

![install libhdf5-openmpi-100:amd64](https://s2.loli.net/2023/03/18/eFDxj1bpiwK4JTG.png)

继续安装：

```shell
sudo apt-get install libopenmpi2:amd64
```

又双叒叕报错：

![install libopenmpi2:amd64](https://s2.loli.net/2023/03/18/95FMZPXOfKVmS3N.png)

继续安装：

```shell
sudo apt-get install libhwloc-plugins:amd64
```

直到提示The following packages will be REMOVED：

![install libhwloc-plugins:amd64](https://s2.loli.net/2023/03/18/ZPAuxI5JHWQyLq7.png)

直接回车删除冲突软件包并继续安装即可。

最后重新安装libvtk7.1：

```shell
sudo apt-get install libvtk7.1-qt
```

