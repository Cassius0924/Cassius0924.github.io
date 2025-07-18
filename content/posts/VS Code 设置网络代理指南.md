---
title: 'VS Code 设置网络代理指南'
draft: false
date: 2024-06-16T11:20:31+08:00
description: ''
author: 'Cassius0924'
tags: ["VSCode", "Proxy", "Network", "Configuration", "Tutorial"]
---

# VS Code 设置网络代理指南

当你使用 VS Code 时，有时你可能需要配置网络代理来访问特定的网络资源（魔法）。

例如在局域网远程开发时使用 GitHub Copilot Chat 插件并且远程主机无魔法时就可以进行配置网络代理。

## 配置教程

打开设置，搜索 proxy 找到 Http: Proxy，填入代理地址即可。

![VSCode Proxy](https://s2.loli.net/2023/06/30/tQqCS5XvzgmLbFK.png)

注意在远程开发时只能设置远程主机的Http代理，无法设置 VS Code 本机的网络代理。
