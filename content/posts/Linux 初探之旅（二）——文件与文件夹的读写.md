---
title: 'Linux 初探之旅（二）——文件与文件夹的读写'
draft: false
date: 2024-06-16T11:20:31+08:00
description: ''
author: 'Cassius0924'
tags: ["Linux", "Shell", "Filesystem", "Tutorial"]
---

# Linux 初探之旅（二）——文件与文件夹的读写

在上一篇文章中，我们学习了Linux中的文件浏览，知道了如何在Linux文件系统中定位文件。本篇文章中，我们将继续深入学习Linux中的文件及目录的读写操作。

## 读取文件

### cat 命令：读取文件全部内容

要读取一个文件的内容，可以使用cat命令，语法为：

```shell
cat 文件名
```

这会将文件的全部内容输出到终端。当文件比较大时，这样的输出会刷屏，不太方便查看。

### head 和 tail 命令：显示部分内容

Linux提供了head和tail命令来显示部分内容：

- head - 显示开头部分内容  
- tail - 显示结尾部分内容

head和tail可以指定显示的行数，例如：

```shell
head -n 3 文件名 # 显示前3行 
tail -n 5 文件名 # 显示后5行
```

### less 命令：分页显示

less命令可以分页方式显示文件内容，可以上下翻阅，是文件查看的首选工具。

less可以用方向键上下翻页，也支持各种快捷键，推荐大家阅读less的帮助文档。

## 写入文件

### echo 命令：输出到文件

使用echo命令可以向文件写入内容，语法为：

```shell
echo "要写入的内容" > 文件名	
```

这会覆盖文件原有内容。如果要附加内容，使用两个大于号：

```shell
echo "新增内容" >> 文件名 
```

### 文本编辑器：vim

对文件进行复杂编辑可以使用文本编辑器，Linux中的常用文本编辑器有vi、emacs、vim和nano等。这里简单介绍vim的使用。

使用`vim 文件名`可以打开vim编辑器。vim有三种模式，分别是命令模式、插入模式和底线命令模式。

- 命令模式：用于导航文件，可以进行复制、粘贴、删除等操作。
- 插入模式：用于输入文本，可以使用键盘输入文本。
- 底线命令模式：用于执行命令，例如保存文件、退出vim等。

初学者可以先了解以下几个快捷键：

- `i` - 进入插入模式
- `ESC` - 从插入模式回到命令模式
- `:w` - 保存文件
- `:q` - 退出vim

vim非常强大，建议大家自己练习熟悉其操作。

## 删除文件和目录

### rm 命令：删除文件

删除文件使用rm命令：

```shell
rm 文件名
```

### rm -r 命令：删除目录

如果要删除目录，需要添加-r参数：

```shell
rm -r 目录名
```

这会递归删除目录及其中的所有内容，需要小心使用。

## 复制和移动文件

- 复制使用cp命令，添加-r参数可以复制目录  

- 移动使用mv命令，mv同时支持文件和目录的移动

```shell
cp file1 file2  # 复制文件file1成为file2
cp -r dir1 dir2 # 复制目录dir1成为dir2

mv file1 file2  # 将文件file1移动并重命名为file2  
mv dir1 dir2   # 将目录dir1移动并重命名为dir2
```

今天学习的这些基础命令非常实用，建议大家练习熟练掌握。可以在home目录下创建文件夹，在里面练习这些命令。掌握后，文件操作会更加高效。
