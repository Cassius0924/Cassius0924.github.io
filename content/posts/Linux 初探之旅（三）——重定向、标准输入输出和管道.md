---
title: 'Linux 初探之旅（三）——重定向、标准输入输出和管道'
draft: false
date: 2024-06-16T11:20:31+08:00
description: ''
author: 'Cassius0924'
tags: ["Linux", "Shell", "Redirect", "Pipe", "Standard I/O", "Tutorial"]
---

# Linux 初探之旅（三）——重定向、标准输入输出和管道



## 标准输入输出

我们先从Linux最基础的交互来讲起，我们究竟是如何和Linux系统内核进行交互的？换句话说，如何通过在按下键盘，就能让系统实现对应的操作的？

这一切都离不开它，终端——用户与Linux建立起联系的桥梁。

当我们在终端上输入命令的时候，直到我们未按下回车键之前，我们输入的所有内容都储存在终端的缓冲区（Buffer）里。此时我们输入的内容不会被系统所读到，只有在我们按下回车键后，缓冲区里的内容才会被发往 stdin （Standrad input 标准输入），stdin 是 Linux 内核和用户交互的通道。

接着，Shell 会介入，它会将 stdin 收到的内容翻译成操作并执行，Shell 译为壳层，与 Kernel（内核）相对应。Shell在外，Kernel在内。例如我们输入`ls`，那么 Shell 就会找到 `ls` 这条命令对应的二进制文件并执行。

执行完命令后，Shell 会将获取到的结果发送到另一条与 stdin 相对应的通道中，即 stdout（Standard output 标准输出）。终端会不间断地从 stdout 里读取内容，然后打印到屏幕上，即我们在终端中看到的输出。

总的来说，用户的输入的内容会发往 stdin 并被 Shell 读取，Shell 将找到内容所对应的二进制文件并执行，执行完后 Shell 会将结果发往 stdout 中，及时的显示在屏幕上。这就是一次与 Linux 交互的过程。 

我们总说，在 Linux 中一切皆为文件，那么 stdin 和 stdout 也不例外，它们也是系统的两个文件。除了 stdin 和 stdout 之外，还有第三条通道 stderr（Standard error 标准错误）。stderr 和 stdout 一样，它们都是用于存储执行结果的地方，区别在于，Shell 执行完命令后，会将正常的结果发往 stdout 里，将错误的结果发往 stderr 里。同样地，stderr 也会被终端所读取，最后显示在屏幕上。

## 输出重定向

理解了标准输入输出后，我们能弄懂重定向的原理了，实际上就是 Shell 执行完命令后不把输出结果存放在 stdout 或 stderr 。而是存放到另一个文件当中。

下面说一下重定向的简单用法，即在命令后面加箭头（>）：

```shell
echo "Hello World" > test.txt 	#将 Hello World 覆盖式写入 test.txt 文件中
```

重定向还有支持更高阶的操作，比如我们可以分别指定标准输出和标准错误的输出位置。

在重定向的时候，`1`代表的就是 stdin，`2`代表的就是 stderr。例如：

```shell
cmd 1>outfile.txt 2>errfile.txt		#cmd 为你想执行的命令
```

我们还可以将一条通道重定向到另一条通道中：

```shell
cmd 2>&1		#将标准错误重定向到标准输出中
```

当箭头后方是一条输出通道的时候，需要加上 & 符号，表示指定的是 1 号通道，而不是一个名为“1”的文件。

将标准错误和标准输出都存放到一个文件内：

```shell
cmd 1>outfile.txt 2>&1
```

注意这里的顺序，`2>&1`必须在后面，因为重定向并不是简单的将两个通道相连接，而是当箭头右侧为一条输出通道时，重定向会将箭头左侧的内容直接连接右侧通道的输出端。

另外，我们可以把不希望输出的内容重定向到`/dev/null`中，这个文件相当于 Linux 的垃圾桶（黑洞）。

## 输入重定向

将箭头反过来（<）就表示输入重定向了。这里用 Linux 中十分常用的一条命令`grep`作为例子，`grep`的作用是在给定内容中寻找目标字符串所在的一行。

```shell
grep "int main" < Hello.cpp
```

## 管道

实际上`grep`命令经与管道（pipe）一起用，管道符号为`|`，其作用也很简单，就是将管道左侧命令的标准输出与管道右侧命令的标准输入相连接。

例如上面的`grep`命令的例子也能用管道实现：

```shell
cat Hello.cpp | grep "int main"
```

下面再举几个`|`的例子：

与`ifconfig`命令配合使用，查询局域网 ipv4 地址：

```shell
ifconfig | grep 192.168
```

与`ps`命令配合使用，查询后台程序是否正在运行：
```shel
ps aux | grep chrome
```
