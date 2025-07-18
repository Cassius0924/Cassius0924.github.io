---
title: '解决 “warning C4819- 该文件包含不能在当前代码页(936)中表示的字符” 问题指南'
draft: false
date: 2024-06-16T19:13:10+08:00
description: ''
author: 'Cassius0924'
tags: ["C++", "CMake", "Windows", "macOS", "Encoding", "Troubleshooting", "Tutorial"]
---

# 解决 “warning C4819: 该文件包含不能在当前代码页(936)中表示的字符” 问题指南

起因是因为我在对我的 C++ 项目进行跨平台适配，从 macOS 平台移植到 Windows 平台时，在使用 Cmake + MSVC 编译后，出现了这个问题。

## 问题原因

这是由于 Windows 平台默认使用的是 GBK 编码，而 macOS 平台上使用的是 UTF-8 编码。

## 解决方法

### 方法一

在 CMakeLists.txt 文件中添加如下代码：

```cmake
add_compile_options("$<$<C_COMPILER_ID:MSVC>:/source-charset:utf-8>")
```

### 方法二

在 CMakeLists.txt 文件中添加如下代码：

```cmake
if(MSVC)
    target_compile_options(<你的项目名> PRIVATE "/utf-8")
endif()
```
