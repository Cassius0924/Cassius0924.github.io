# Jetson Xavier NX 升级或重新安装 JetPack 指南

## 前提

你需要拥有一台 x86 架构（非ARM64）的 Ubuntu 主机。

## 下载安装 SDK Manager

### 下载

在 [Nvidia SDK Manager](https://developer.nvidia.com/sdk-manager) 官网下载，下载自己系统对应的 .deb 或 .rpm 安装包。

下载之前需要注册 Nvidia 开发者帐号，SDK Manager 也需要开发者帐号才能使用。

### 安装

Ubuntu 系统使用 apt 安装刚刚下载的 .deb 文件：

```bash
sudo apt install ./sdkmanager_[version]-[build]_amd64.deb
```

CentOS 8.0 和 8.2 系统使用 dnf 安装 .rpm 文件：

```bash
sudo dnf install ./sdkmanager_[version]-[build].x86_64.rpm
```

CentOS 7.6 使用 yum 安装：

```bash
sudo yum install ./sdkmanager_[version]-[build].x86_64.rpm
```

## 启动 SDK Manager

在终端使用命令启动 SDK Manager：

```bash
sdkmanager
```

启动后需要登录 Nvidia 开发者帐号。

![SDK Manager Login](https://s2.loli.net/2023/10/13/Za4GUdmqTVO2lij.png)

## 安装 JetPack

### 连接 Jetson

先将 Jetson 与 SDK Manager 主机使用 USB to Micro-USB 数据线相连接。

连接成功后，SDK Manager 会自动弹窗。

![SDK Manager Popup](https://s2.loli.net/2023/10/13/3BAnU4IJXOZo1bR.png)

### STEP 01

取消选择 Host Machine，这个选项会在 SDK Manager 所在的主机上也安装开发包。

Linux JetPack 选择自己需要的版本，这里以最新的 5.1.2 为例。

完毕后点击左下角的 CONTINUE。

### STEP 02

如果是升级 JetPack，则需要勾选上 Jetson Linux image，重装系统。这个系统会安装在 Jetson 自带的 SSD 里，不会影响到 Jetson 的 NVMe 固态硬盘。重装系统完毕后只需要重新运行 [rootOnNVMe](https://github.com/jetsonhacks/rootOnNVMe.git) 即可。



如果你是重新安装 JetPack（版本不变），则只需要勾选下方 Jetson SDK Componets即可，一般不勾选 Deverloper Tools。



勾选许可证条款，并点击 CONTINUE，如何输入 SDK Manager 主机的密码，注意这个密码不是 Jetson 的密码。

### STEP 03

输入 Jetson 的用户名和密码。

点击 Install，开始安装，过程大约半个小时。

### STEP 04

安装完毕。

