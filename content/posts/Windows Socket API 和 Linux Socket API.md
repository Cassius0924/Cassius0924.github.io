---
title: 'Windows Socket API 和 Linux Socket API'
draft: false
date: 2024-06-16T19:13:10+08:00
description: ''
author: 'Cassius0924'
tags: ["C++", "Linux", "Network", "Socket", "Windows"]
---

# Windows Socket API 和 Linux Socket API 的区别

本文章主要介绍 Windows 下和 Linux 下的 Socket 编程区别，即 Windows Socket API 和 Linux Socket API 的区别。

## 头文件

Windows 环境下的 Socket 编程需要以下头文件:

- `<WinSock2.h>`
- `<WS2tcpip.h>`

> [!NOTE]
>
> 如果使用 MSVC 编译器，那么还需要使用预处理指令 `#pragma comment(lib, "Ws2_32.lib")` 来链接 `Ws2_32.lib` 库。

而 Linux 环境下的 Socket 编程可能会用到以下头文件:

- `<sys/socket.h>`
- `<netinet/in.h>`
- `<arpa/inet.h>`
- `<unistd.h>`
- `<fcntl.h>`

```c++
#indef __linux__
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <fcntl.h>

#elif defined(_WIN32)
#include <winsock2.h>
#include <WS2tcpip.h>
```

## socket 套接字类型

- Linux 环境下为 `int`

- Windows 环境下为 `SOCKET`

## 函数返回值

Windows 环境下有专门定义操作成功的宏 `NO_ERROR`，Linux 环境下则没有。

## close 和 closesocket

关闭一个套接字需要使用 `close` 或 `closesocket`，Linux 环境下需要使用 `close` 删除套接字描述符，Windows 环境下则使用 `closesocket`。

- Linux 环境

```cpp
close(fd);
```

返回值：

| 返回值 | 描述 |
| --- | --- | 
| `EBADF` | fd 不是有效的文件描述符。 |
| `EINTR` | 执行被信号中断。 |
| `EIO` | 先前未提交的 write(2) 遇到输入/输出错误。 |

- Windows 环境

```cpp
closesocket(fd);
```

返回值：

| 返回值 | 描述 |
| --- | --- | 
| `WSANOTINITIALIZED` | 在使用此函数之前，必须成功调用 WSAStartup。 |
| `WSAENETDOWN`       | 网络子系统发生故障。 |
| `WSAENOTSOCK`       | 描述符不是套接字。 |
| `WSAEINPROGRESS`    | 阻止 Windows 套接字 1.1 调用正在进行，或者服务提供商仍在处理回调函数。|
| `WSAEINTR`          | (阻止) Windows 套接字 1.1 调用已通过 WSACancelBlockingCall 取消。 |
| `WSAEWOULDBLOCK`    | 套接字标记为非阻止，但 linger 结构的 l_onoff 成员设置为非零，而 linger 结构的 l_linger 成员设置为非零超时值。 |


## getsockopt 和 setsockopt

- Linux 环境下的函数原型

```cpp
int getsockopt(
    int sockfd, 
    int level,
    int optname,
    void optval[restrict *.optlen], 
    socklen_t *restrict optlen
);
int setsockopt(
    int socket, 
    int level, 
    int option_name, 
    const void *option_value, 
    socklen_t option_len
);
```

- Windows 环境下的函数原型

```cpp
int getsockopt(
    [in]      SOCKET s,
    [in]      int    level,
    [in]      int    optname,
    [out]     char   *optval,
    [in, out] int    *optlen
);
int setsockopt(
    [in] SOCKET     s,
    [in] int        level,
    [in] int        optname,
    [in] const char *optval,
    [in] int        optlen
);
```

二者区别在于 Windows 下的 `optval` 是 `char *` 类型，而 Linux 下的 `optval` 是 `void *` 类型。

所以建议使用 `char *` 类型，这样可以在 Windows 和 Linux 下都能正常使用。

## fcntl 和 ioctlsocket

这两个函数都是用来设置套接字的属性，比如设置非阻塞模式等。

- Linux 环境下的函数原型

```cpp
int fcntl(int fd, int cmd, ... /* arg */ );
```

cmd 参数的取值：

- `F_DUPFD`：复制文件描述符
- `F_GETFL`：获取文件状态标志，返回值是文件状态标志的副本；
- `F_SETFL`：设置文件状态标志，设置文件状态标志为 arg 参数的值；
- `F_GETFD`：获取文件描述符标志，返回值是文件描述符标志的副本；
- `F_SETFD`：设置文件描述符标志，设置文件描述符标志为 arg 参数的值；

> [!NOTE] FL 和 FD 的区别
>
> 文件描述符标志（FD Flags）和文件状态标志（FL Flags）都是与文件描述符相关的属性，但它们的用途和含义有所不同。
> 
> - 文件描述符标志（FD Flags）：这些标志控制文件描述符的行为。目前，只有一个文件描述符标志，即 FD_CLOEXEC。
> 
> - 文件状态标志（FL Flags）：这些标志描述了文件的状态，例如文件是以读模式、写模式还是读写模式打开的，文件是否支持非阻塞操作，等等。

- Windows 环境下的函数原型

```cpp
int WSAAPI ioctlsocket(
  [in]      SOCKET s,
  [in]      long   cmd,
  [in, out] u_long *argp
);
```

cmd 参数的取值：

- `FIONBIO`：设置非阻塞模式，如果设置为0，那么套接字是阻塞的，否则是非阻塞的；
- `FIONREAD`：获取接收缓冲区中的字节数，如果是流式套接字（TCP），那么返回的是接收缓冲区中的字节数，如果是数据报套接字（UDP），那么返回的是下一个数据报的大小；
- `SIOCATMARK`：确定是否已经读取了所有的Out-of-Band数据（带外数据），此命令只适用于设置了SO_OOBINLINE选项的流式套接字，如果没有OOB数据等待读取，那么返回TRUE，否则返回FALSE；

返回值：

| 错误代码 | 含义 |
| --- | --- |
| `WSANOTINITIALIZED` | 在使用此函数之前，必须成功调用 WSAStartup 。 |
| `WSAENETDOWN` | 网络子系统发生故障。 |
| `WSAEINPROGRESS` | 阻止 Windows 套接字 1.1 调用正在进行，或者服务提供商仍在处理回调函数。 |
| `WSAENOTSOCK` | 描述符 s 不是套接字。 |
| `WSAEFAULT` | argp 参数不是用户地址空间的有效部分。 |
