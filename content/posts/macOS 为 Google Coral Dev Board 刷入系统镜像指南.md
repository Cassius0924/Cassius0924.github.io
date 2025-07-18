---
title: 'macOS 为 Google Coral Dev Board 刷入系统镜像指南'
draft: false
date: 2024-06-16T11:20:31+08:00
description: ''
author: 'Cassius0924'
tags: ["macOS", "Google Coral", "Edge TPU", "Flash", "Mendel Linux", "Tutorial"]
---

# macOS 为 Google Coral Dev Board 刷入系统镜像指南

本指南旨在帮助 macOS 开发者为 Google Coral Dev Board （Google Edege TPU） 通过 OTG 刷入 Mendel Linux 系统。

## 第一步：连接到开发板串行控制台

### 安装 USB 转 UART Bridge VCP 驱动程序

官方下载地址：[https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers?tab=downloads](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers?tab=downloads)

![CP210x VCP](https://s2.loli.net/2023/07/25/5n7sKqk1ValvcFR.png)

下载完成后，双击运行并安装即可。

### 安装辅助程序

通过 Homebrew 安装：

```shell
sudo brew install screen fastboot
```

### 连接开发板

先不给开发板供电，使用 micro-B USB 线将 Mac 连接到开发板。连接成功后会开发板会亮起绿色🟩和橙色🟧指示灯。

![micro USB](https://s2.loli.net/2023/07/26/8miISKlWOMy149J.jpg)

随后使用命令查看串口连接是否正常工作：

```shell
ls /dev/cu*
```

连接正常应该会列出`/dev/cu.SLAB_USBtoUART`，如下图所示。

![ls /dev/cu*](https://s2.loli.net/2023/07/26/yafnbF9HUe8DQqT.png)

然后使用`screen`命令连接开发板串行命令行：

```shell
screen /dev/cu.SLAB_USBtoUART 115200
```

此时命令行应为空白，因为开发板未供电开机。

### 给开发板供电

如图所示，将开发板电源插入，电源接口为右边的 USB-C 接口。

![Power](https://s2.loli.net/2023/07/26/sHYdWNjafQCeTwl.jpg)

连接后开发板会亮起电源指示灯并转动风扇。串行命令行则会打印开发板的开机信息：

![Coral Dev Board power on](https://s2.loli.net/2023/07/26/h8JnuNpzWBsG25I.png)

接着在串行命令行输入命令使开发板进入 fastboot 模式：
```shell
fastboot 0
```

![fastboot 0](https://s2.loli.net/2023/07/26/owkygYAm5JTUxjt.png)

## 第二步：刷入 Mendel Linux 系统

### 下载 Mendel Linux 系统镜像

回到 Mac，下载 Mendel Linux 系统镜像，官方下载地址：[https://coral.ai/software/#mendel-dev-board](https://coral.ai/software/#mendel-dev-board)

![Mendel Linux Download](https://s2.loli.net/2023/07/26/srVpvxDLg21Rodf.png)

下载完成后解压。

### 刷入系统

如图所示，给开发板插入 OTG 数据线。

![OTG 数据线](https://s2.loli.net/2023/07/26/ux7SbIMkl29qjZa.jpg)

在 Mac 中新建一个终端界面，输入命令：

```shell
fastboot devices
```

若 OTG 连接成功，则会显示一串序列号`1b0741d6f0609912        fastboot`。

连接成功后，转到刚刚下载的 Mendel Linux 文件夹，运行刷入脚本：

```shell
bash flash.sh
```

刷入过程大约5分钟，刷入成功后，开发板将会自动重启。

### 连接网络

刷完系统后，需要先登录用户，敲击几下回车键。

输入用户名：mendel

输入密码：mendel

进入系统后先连接好网络，方便后续远程连接到开发板。

可以使用有线连接，也可以使用 Wi-Fi 连接。Wi-Fi 连接则运行以下命令：

```shell
nmtui
```

然后选择 `Activate a connection` 并按照命令行 UI 连接网络即可。

## 第三步：配置 SSH 免密登录

由于从 MacOS 10.15 (Catalina) 开始，MDT 无法通过 USB 创建到开发板，但还可以通过 SSH 远程连接。

### 创建 Mendel SSH 密钥

在 Mac 上新建一个终端，输入命令：

```shell
ssh-keygen -t rsa -m PEM
```

输入文件名“mendel”，密码则留空。

这将在当前目录中创建一个名为 `mendel` 的私钥和一个名为 `mendel.pub` 的公钥。

修改文件权限：

```bash
chmod 600 mendel
```

将文件移动到 MDT 文件夹：

```shell
mkdir -p ~/.config/mdt/keys && mv mendel ~/.config/mdt/keys/mdt.key
```

### 将公钥复制到开发板

输入`cat mendel.pub`并复制公钥内容。

转到串行控制台，为 SSH 密钥创建一个新文件：（注意是在开发板上创建）

```shell
mkdir /home/mendel/.ssh && vi /home/mendel/.ssh/authorized_keys
```

然后粘贴刚刚复制的公钥内容。完成之后保存并关闭（:wq）。

### 开启 SSH 密码登录权限

因为 Coral Dev Board 默认禁用了 SSH 密码登录权限，需要手动开启。

在串行命令行中打开 SSH 配置文件：

```shell
vim /etc/ssh/sshd_config
```

找到`PasswordAuthentication no`并修改为`PasswordAuthentication yes`。

重启 SSH 服务：

```shell
/etc/init.d/sshd restart
```

## 第四步： 安装 MDT 开发工具

Mendel Development Tool（MDT）是一个命令行工具，可让你的电脑与 Mendel Linux 系统的设备进行通信。

使用以下命令方式安装 MDT：

```shell
python3 -m pip install --user mendel-development-tool
```

安装完毕后需要配置环境变量：

```shell
echo 'export PATH="$PATH:$HOME/Library/Python/3.11/bin/"' >> ~/.bash_profile
source ~/.bash_profile
```

此处的路径与你所使用的Python版本有关。

使用以下命令连接开发板。 

```shell
mdt shell
```

若提示未找到命令，则表示为环境变量为配置正确，尝试重新配置。

成功进入提示如下图：
