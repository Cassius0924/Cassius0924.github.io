---
title: 'Unity 脚本使用 NuGet 安装解决方案包'
draft: false
date: 2023-06-01T10:00:00+08:00
description: ' 本文旨在教大家如何在 Unity 中使用 NuGet 安装解决方案包。'
author: 'Cassius0924'
tags: ["Unity", "NuGet", "Package Management", "C#", "Tutorial"]
---

# Unity 脚本使用 NuGet 安装解决方案包

本文旨在教大家如何在 Unity 中使用 NuGet 安装解决方案包。

Visual Studio 集成了 NuGet 管理器，可以方便的安装和卸载解决方案包。但在使用 Viusal Studio 开发 Unity C# 脚本时，不能直接在内置的 NuGet 管理器安装解决方案包。因为 Unity 工程打开或运行时会刷新工程文件，导致我们在 Visual Studio 内置 NuGet 管理器安装的解决方案包失效。所以 Unity 使用 NuGet 安装解决方案包需要特殊步骤。

## 步骤

### 下载解决方案包

在[NuGet官网](https://www.nuget.org/)下载你需要解决方案包，例如Google.Protobuf。

直接下载会下载最新预览版，稳定版需要点击右上角的 Full stats，查看所有版本。

![Full stats](https://s2.loli.net/2023/05/08/1mRe8FJbMDO3q2i.png)

找到稳定版 3.22.4，点击即可下载。

![Protobuf 3.22.4](https://s2.loli.net/2023/05/08/S8LCYDNglpVZbP7.png)

### 解压包

下载后会得到拓展名为`.nupkg`的文件，先将文件拓展名改为`.zip`，接着解压即可。

![Unzip](https://s2.loli.net/2023/05/08/RvoI35FghtaudVO.png)

### 导入dll文件

解压后可以得到 dll 文件，位于 google.protobuf.3.22.4/lib/**/Google.Protobuf.dll ，将dll文件复制到 Unity 工程文件夹的 Assets/Plugins 文件夹下，若没有这个文件夹自己手动新建一个。

![Google.Protobuf.dll](https://s2.loli.net/2023/05/08/LXITjqHRUPaVSvK.png)

### 配置Unity项目

接着需要将 Unity 项目的 Api Compatibility Level 更改为与 dll 文件对应的版本。打开 Unity 的 Project Settings，找到 Player > Other Settings > Api Compatibility Level。

![Api Compatibility Level](https://s2.loli.net/2023/05/08/BMLFqRVNXmadybo.png)

### 重启 Unity 项目

如果 Unity 报错提示刚刚导入的解决方案包 unsafe 的话，重启 Unity 项目即可。

这样就愉快的可以在 Unity C# 脚本中使用刚刚导入的解决方案包啦。