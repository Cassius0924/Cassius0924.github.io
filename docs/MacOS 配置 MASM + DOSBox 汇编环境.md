---
layout: '../../layouts/MarkdownPost.astro'
title: 'MacOS 配置 MASM + DOSBox 汇编环境'
pubDate: 2023-04-12
description: '该文章旨在帮助在 MacOS 系统下想要学习汇编语言的开发者配置 MASM + DOSBox 环境。'
author: 'Cassius0924'
cover:
    url: 'https://s2.loli.net/2023/04/12/SVgEioRKfzWXQt8.png'
    square: 'https://s2.loli.net/2023/04/12/SVgEioRKfzWXQt8.png'
    alt: 'DOSBox'
tags: ["MacOS", "MASM", "DOSBox", "Assembly"]
theme: 'light'
featured: ture
---

# macOS 配置 MASM + DOSBox 汇编环境

该文章旨在帮助在 MacOS 系统下想要学习汇编语言的开发者配置 MASM + DOSBox 环境。

## 配置步骤

1. 克隆该项目到本地：

```bash
git clone https://github.com/cassius0924/MASMForMac.git
```

2. 将项目文件夹移动某个位置：

   例如我将这个文件夹放到了 `~/Documents/`中

```bash
mv MASMForMac ~/Documents/.
```

3. 安装 DOSBox：

   用 Finder 打开 MASMForMac/mac 目录，打开 dmg 安装包。再打开 Application 文件夹，将 DOSBox 应用程序拖入 Application 文件夹中。

   ![Install DOSBox](https://s2.loli.net/2023/04/12/VFnbHXMK2WDEJ49.png)

4. 在 DOSBox 中挂载目录：

```bash
mount c ~/Documents/MASMForMac
```

此目录将会把`~/Documents/MASMForMac`挂载到 C 盘。路径根据自己情况修改。

每次打开 DOSBox 应用程序都需要挂载目录。可以配置自动挂载，详见下文**「配置文件」**。

5. 在 DOSBox 中进入 C 盘：

```bash
c:
```

6. 编译 `hello.asm` 文件：

已经预先写好了一个`hello.asm`汇编测试示例，用于测试汇编环境。

```bash
masm hello.asm
```

连按三下回车，此命令会生成`obj`文件。

7. 生成`hello.exe`可执行文件：

```bash
link hello.obj
```

同样连按三下回车。

7. 运行 `hello.exe` 文件：

```bash
hello.exe
```

正常情况，会输出字符串：Hello world!

![Hello world!](https://s2.loli.net/2023/04/12/MFtVCEZekNRgSIQ.png)

## 文件结构

```
.
├── CREF.EXE
├── ERROUT.EXE
├── EXEMOD.EXE
├── EXEPACK.EXE
├── LIB.EXE
├── LINK.EXE
├── MAKE.EXE
├── MASM.EXE
├── README.DOC
├── SETENV.EXE
├── debug.exe
├── hello.asm
└── mac
    ├── DOSBox-0.74-3-3.dmg
    └── masm5.zip
```

## 配置文件

MacOS 的 DOSBox 配置文件位于 `~/Library/Preferences/DOSBox\ 0.74-3-3\ Preferences`

```bash
vim ~/Library/Preferences/DOSBox\ 0.74-3-3\ Preferences
```

### 配置自动挂载

在文件末尾加入两行命令：

```
mount c ~/Documents/MASMForMac
c:
```

每次启动 DOSBox 将会自动运行这两行命令。

### 调整窗口大小

找到`windowresolution=origin`，将`origin`修改成 `长度x高度`。例如：

```
windowresolution=1024x680
```

### 其他配置

更多配置详解见[官方说明](https://www.dosbox.com/wiki/Dosbox.conf#Mac_OS_X)。

