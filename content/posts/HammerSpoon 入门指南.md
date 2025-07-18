---
title: 'HammerSpoon 入门指南'
draft: false
date: 2024-06-16T11:20:31+08:00
description: ''
author: 'Cassius0924'
tags: ["HammerSpoon", "macOS", "Automation", "Lua", "Scripting", "Tutorial"]
---

# HammerSpoon 入门指南

## 介绍

HammerSpoon 这是一款强大的OS X自动化工具。

HammerSpoon 本质上就是操作系统和Lua脚本引擎之间的桥梁。

HammerSpoon 之所以强大，在于它提供了一系列向用户公开特定系统功能模块的扩展。有了这些扩展，用户便可利用Lua脚本来控制 macOS 的各个方面。

## 安装

建议直接使用 Homebrew 安装。

```shell
brew install hammerspoon --cask
```

手动安装参考[官方 Github](https://github.com/Hammerspoon/hammerspoon)。

## 参考文档

- [HammerSpoon 入门指南](https://www.hammerspoon.org/go/)
- [HammerSpoon API 文档](https://www.hammerspoon.org/docs/)
- [HammerSpoon 常见问题](https://www.hammerspoon.org/faq/)
- [HammerSpoon 配置示例](https://github.com/Hammerspoon/hammerspoon/wiki/Sample-Configurations)

## 快速开始

### 启动应用

安装完毕后启动 HammerSpoon，设置中打开辅助功能。

![HS Preferences](https://s2.loli.net/2023/08/06/ZkNiYmXvIRPJ4bs.png)

接着点击 Open Config 打开配置文件。下面开始教程。

### Hello World

```lua
-- Hello World
hs.hotkey.bind({"cmd", "alt", "shift", "ctrl"}, "W", function()
    hs.alert.show("HammerSpoon is working!")
end)
```

> 每次修改配置文件后都需要点击 Reload Config。

以上代码实现了点击快捷键 `command` + `option` + `shift` + `control` + `w`，在屏幕中间显示弹窗提示的功能。

![Hello World](https://s2.loli.net/2023/08/06/PR3k6Ty4K5AVIqx.png)

也可以使用 macOS 原生通知形式

```lua
-- Hello World（macOS native notification）
hs.hotkey.bind({"cmd", "alt", "shift", "ctrl"}, "Q", function()
    hs.notify.new({title="macOS Native Notification", informativeText="HammerSpoon is working"}):send()
end)
```

![macOS native notification](https://s2.loli.net/2023/08/06/HwxemCVBGf1KNnh.png)
![[bg right opacity]](https://s2.loli.net/2023/08/06/HwxemCVBGf1KNnh.png)

> [hs.hotkey API 介绍](https://www.hammerspoon.org/docs/hs.hotkey.html)
> 
> [hs.notify API 介绍](https://www.hammerspoon.org/docs/hs.notify.html)
