---
layout: '../../layouts/MarkdownPost.astro'
title: 'apt autoremove 误删依赖包后自救指南'
pubDate: 2023-03-24
description: 'apt autoremove后程序不能用了？别慌，跟着教程一步一步来。'
author: 'Cassius0924'
cover:
    url: 'https://s2.loli.net/2023/03/24/SnH3LoVEqXbgalP.jpg'
    square: 'https://s2.loli.net/2023/03/24/SnH3LoVEqXbgalP.jpg'
    alt: 'Linux APT'
tags: ["Linux", "Ubuntu", "Shell", "Skill"]
theme: 'light'
featured: ture
---

# apt autoremove 误删依赖包后自救指南

![Linux APT](https://s2.loli.net/2023/03/24/SnH3LoVEqXbgalP.jpg)

在使用 Ubuntu 系统的过程中，我们常常需要使用 apt 命令来安装、升级和删除软件包。其中，apt autoremove 命令可以自动删除无用的依赖包，以释放硬盘空间。然而，有时候我们会不小心误删了一些必要的依赖包，导致某些程序无法正常运行。

## 方法一：手动查看历史记录（推荐）

1. 查看APT历史日志

	```bash
	sudo vim /var/log/apt/history.log
	```

	直接输入大写`G`，跳转到最后一行，找到相应时间的Remove内容。

	![APT histroy log](https://s2.loli.net/2023/03/23/5tGMYikenKFyHhr.png)

	若删除的包较少可以逐个`apt install`。若包较多可以使用正则表达式，具体操作如下。

2. 复制所有被删除的包名

  如果你的VIM开启了行号显示，请先临时禁用行号，目的是避免复制到多余的空格。

  禁用行号，在命令模式下输入以下命令：

  ```bash
  :set nonumber
  ```

  先复制你所有的删除的包，即Remove后的内容。

  开启行号：

  ```bash
  :set number
  ```

  输入`:q`退出VIM。

3. 利用正则表达式处理包名

	执行以下命令，用正则表达式删去版本信息和逗号：

	```bash
	echo "粘贴在这" | sed 's/([^()]*)[,]*//g'
	```
	
	复制输出的内容。

4. 执行安装命令

   一口气全安装即可：

	```bash
	sudo apt install "粘贴在这"
	```

## 方法二：使用 aptitude 进行恢复

aptitude 是一款强大的包管理器，它可以自动解决依赖关系，并且可以清晰地显示出哪些包被删除、哪些包被保留。因此，我们可以使用 aptitude 来恢复误删的依赖包。

首先，我们需要安装 aptitude：

```
sudo apt-get install aptitude
```

然后，使用以下命令来查看被删除的软件包：

```
sudo aptitude search '~c'
```

接下来，使用以下命令来恢复被删除的软件包：

```
sudo aptitude install <package-name>
```

