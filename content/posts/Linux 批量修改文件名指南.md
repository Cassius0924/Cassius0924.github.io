---
title: 'Linux 批量修改文件名指南'
draft: false
date: 2024-06-16T11:20:31+08:00
description: ''
author: 'Cassius0924'
tags: ["Linux", "Shell", "Bash", "CLI", "Tutorial"]
---

# Linux 批量修改文件名指南

使用 bash 脚本，先创建一个 .sh 结尾的脚本：

```vim
vim rename.sh
```

以下是示例 bash 脚本内容，作用是将 /path/to/dir 目录下所有包含冒号`:`的文件名，将冒号替换为减号。

```bash
# !/bin/bash

find /path/to/dir -type f -name '*:*' -exec bash -c 'mv "$0" "${0//:/-}"' {} \;
```

## 代码解释

### 使用`find`查找需要更改的文件

```bash
find /path/to/dir -type f -name '*:*'
```

- `/path/to/dir` 应该替换为包含你要修改文件名的文件夹的实际路径。
- `-type f` 表示只查找普通文件，而不包括目录。
- `-name '*:*'` 是一个查找条件，用于匹配包含冒号的文件名。

### 使用`mv`和`bash`执行文件名更改

一旦找到需要更改的文件，可以使用`mv`命令结合`bash`来执行文件名更改操作。

```bash
find /path/to/dir -type f -name '*:*' -exec bash -c 'mv "$0" "${0//:/-}"' {} \;
```

这个命令中的 `-exec` 标志用于在`find`查找到的每个文件上执行指定的命令。`bash -c`之后的部分将执行文件名更改操作。

- `mv "$0" "${0//:/-}"` 使用`mv`命令将文件名中的冒号替换为减号。`${0//:/-}` 部分是一个bash子shell，其中的`${0}`表示当前文件名，`//`后跟着`:`和`-`是用来替换的正则表达式。
