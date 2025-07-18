---
title: 'MacOS 修改 App Store 和地图软件语言指南'
draft: false
date: 2023-04-14T10:00:00+08:00
description: '本文提供了一种简单的方法，让您能够修改 App Store 和 Maps 的语言设置。'
author: 'Cassius0924'
tags: ["macOS", "App Store", "Maps", "Language", "Configuration", "Tutorial"]
---

# macOS 修改 App Store 和地图软件语言指南

在 macOS 系统设置中，我们可以轻松地设置软件的语言，但有些应用程序如 App Store 和 Maps 却无法直接在设置中修改语言。这可能会让我们在使用这些应用程序时感到困惑，尤其是当我们需要使用不同语言的应用程序时。

本文提供了一种简单的方法，让您能够修改 App Store 和 Maps 的语言设置，以便在需要时更轻松地使用这些应用程序。

## 修改方法

修改**苹果地图**的语言为简体中文：

```bash
sudo defaults write com.apple.Maps AppleLanguages '("zh-CN")'
```

修改 **App Store** 的语言为美式英语：

```bash
sudo defaults write com.apple.AppStore AppleLanguages '("en-US")'
```

## 其他软件

其他软件均可在系统设置里直接设置。

## 其他语言

同理，其他语言只需修改命令最后的**「语言区域码」**即可。

| 语言名称         | 代码    |
| ----------- | ------- |
| 中文（简体）     | `zh-CN` |
| 中文（繁体）     | `zh-TW` |
| 英语（美国）     | `en-US` |
| 英语（英国）     | `en-GB` |
| 日语             | `ja`    |
| 法语             | `fr`    |
| 德语             | `de`    |
| 西班牙语         | `es`    |
| 韩语             | `ko`    |
| 俄语             | `ru`    |
| 葡萄牙语（巴西） | `pt-BR` |
| 阿拉伯语         | `ar`    |
| 意大利语         | `it`    |
| 土耳其语         | `tr`    |
| 印地语           | `hi`    |
| 印尼语           | `id`    |
| 荷兰语           | `nl`    |
| 波兰语           | `pl`    |
| 瑞典语           | `sv`    |
| 丹麦语           | `da`    |
| 芬兰语           | `fi`    |
| 挪威语           | `no`    |
| 希腊语           | `el`    |
