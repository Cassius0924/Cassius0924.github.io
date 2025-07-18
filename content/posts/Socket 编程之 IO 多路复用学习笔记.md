# Socket 编程之 IO 多路复用学习笔记

## 什么是 IO 多路复用？

### 阻塞 IO 与 非阻塞 IO

我们先了解一下阻塞 IO，阻塞 IO 是指应用程序在读写数据时，如果没有数据可读或者写，应用程序会一直 **阻塞在那里** ，直到有数据可读或者写。

与它相反的是非阻塞 IO，是指应用程序在读写数据时，无论是否有数据可读写，都 **立即返回** ，若没有数据可读写将会返回一个错误码，通过不断轮询来检查是否有数据可读或者写。

### IO 多路复用

由于 Scoket 默认是阻塞 IO，所以很多初学者在处理多个连接时，会为每个连接创建一个线程来处理，但这样做会引起 CPU 的上下文切换，降低系统的性能。

有一种更“优雅”的方式，那就是 IO 多路复用，也称为事件驱动模型（Event-driven IO）。IO 多路复用是指内核一旦发现进程指定的一个或者多个 IO 事件已经就绪，就通知该进程。IO 多路复用模型中，只需要一个线程就可以同时处理多个连接。

通俗易懂的说，IO 多路复用就是将多个 IO 事件交给内核，内核帮我们监听这些 IO 事件，当有 IO 事件就绪时，内核会通知我们，我们只需要处理就绪的 IO 事件即可。

## IO 多路复用的优点

- 一个线程可以同时处理多个连接，减少线程的创建和销毁

- 降低了系统开销，提高了系统的并发性能

## IO 多路复用的实现方式

- select

- poll

- epoll (Linux)

- kqueue (FreeBSD)

- IOCP（Windows）

其中 epoll 是 Linux 下的 IO 多路复用机制，kqueue 是 FreeBSD（macOS 就属于 FreeBSD）下的 IO 多路复用机制。

下面的伪代码是 IO 多路复用的最基本实现方式：

``` c++
while (1) {
    for (fd : fds) {    // 遍历所有的文件描述符
        if (fd 有数据) {    
            处理数据;   // 如果文件描述符有数据则处理数据
        }
    }
}
```

如果我们在自行编写的程序中使用上面的伪代码，那么每次判断 `fd` 是否有数据时，都需要询问内核，所以每次判断都会引起一次系统调用，也就是会从用户态切换到内核态，这样的效率是不够高的。

所以操作系统为我们提供了 `select`、`poll`、`epoll`、`kqueue`、`ICOP` 这几种解决方案，它们可以在内核中直接监听文件描述符，当文件描述符就绪时，内核会通知我们，这样就不需要我们自己去轮询文件描述符了。

下面介绍一下五种方式的区别：

### select

> [!NOTE]
>
> **前置知识**
> 
> - **fd_set**
>     
>     fd_set（file descriptor set）是一个文件描述符集合，它本质上是一个 bitmap（位图），每个文件描述符对应一个位，如果文件描述符在集合中，则对应的位为 1，否则为 0。例如我们有 3 个文件描述符，分别为 `3`、`4`、`8`，那么 fd_set 的值为 `000110001`。
> 
> - **struct timeval**
> 
>     struct timeval 是一个结构体，用来表示时间，它有两个成员变量，分别是 `tv_sec` 和 `tv_usec`，分别表示秒和微秒。

> [!TIP]
>
> **文件描述符补充知识**
>
> 在 Linux 中，用户创建的文件描述符最小值为 3，因为 0、1、2 分别为标准输入、标准输出和标准错误。

``` c
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

- **nfds**：监听的文件描述符的最大值加 1

- **readfds**：读事件 fd 集合（输出参数）

- **writefds**：写事件 fd 集合（输出参数）

- **exceptfds**：异常事件 fd 集合（输出参数）

- **timeout**：超时时间，类型为 struct timeval

- **函数返回值**：就绪的文件描述符的数量，如果超时返回 0，出错返回 -1

从函数原型可以看出，`select` 函数中间三个参数都是输出参数，我们只需要传入 `fd_set` 结构体的指针即可。我们需要给予 `select` 函数的数据只有监听的 `fd` 的最大值加 1 和监听超时时间，然后 `select` 函数就会帮我们监听这些文件描述符。

至于 nfds 为什么要加 1，是因为 `select` 函数是从 0 开始遍历文件描述符的。用刚刚 fd_set 的例子来说，如果我们不加 1，那么 `select` 函数只会遍历到 `00011000` 七个 fd，而不会遍历最后一个 `1` 对应的 fd。

`timeout` 参数除了设置一个具体的时间值外，还可以设置为 `-1`，表示永久阻塞，直到有文件描述符就绪。如果设置为 `0`，则表示立即返回，不会阻塞。

select 的工作原理大致如下：

1. 用户态将需要监听的文件描述符集合传递给内核

2. 内核将这些文件描述符集合拷贝到内核空间

3. 内核循环遍历这些 fd ，如果有 fd 可读，则将 readfds 对应的位设置为 1，如果有 fd 可写，则将 writefds 对应的位设置为 1，如果有异常，则将 exceptfds 对应的位设置为 1

4. 函数返回

`select` 函数每次调用时会将需要遍历的文件描述符直接一次全量拷贝到内核空间，并在内核态进行轮询判断，这样就不会引起用户态和内核态的频繁切换，自然也就提高了效率。

当 `select` 函数返回后，我们就可以通过判断 `readfds`、`writefds`、`exceptfds` 中的位来判断哪些文件描述符有数据可读、可写或者有异常。

`select` 函数的使用示例：

``` c linenums="1" hl_lines="9-17"
// ...
// 假设 fds 为一个 fd 数组，里面存放了需要监听的文件描述符，fd_max 为 fds 中最大的 fd

fd_set r_fdset;
fd_set w_fdset;
fd_set e_fdset;

while (1) {
    FD_ZERO(&r_fdset);  // 清空 fd_set
    FD_ZERO(&w_fdset);
    FD_ZERO(&e_fdset);

    for (int i = 0; i < fds.size(); i++) {  // 初始化 fd_set
        FD_SET(fds[i], &r_fdset);
        FD_SET(fds[i], &w_fdset);
        FD_SET(fds[i], &e_fdset);
    }

    struct timeval timeout;
    timeout.tv_sec = 5;  // 设置超时时间为 5 秒
    timeout.tv_usec = 0;

    int ret = select(fd_max + 1, &r_fdset, &w_fdset, &e_fdset, &timeout);
    if (ret == 0) {
        printf("select timeout\n");
        continue;
    }
    if (ret < 0) {
        perror("select error");
        break;
    }

    for (int i = 0; i < fds.size(); i++) {
        if (FD_ISSET(fds[i], &r_fdset)) {
            // 处理读事件
        }
        if (FD_ISSET(fds[i], &w_fdset)) {
            // 处理写事件
        }
        if (FD_ISSET(fds[i], &e_fdset)) {
            // 处理异常事件
        }
    }
}
```

`select` 函数的缺点：

- fd_set 的大小有限，一般为 1024，所以 `select` 函数最多只能监听 1024 个文件描述符
- fd_set 不可重用，每次调用 `select` 函数都需要重新初始化 fd_set。上述代码中，每次 while 循环都需要重新初始化 r_fdset、w_fdset 以及 e_fdset
- `select` 函数的时间复杂度为 O(n)，n 为文件描述符的数量
- 每次调用 `select` 函数都会将所有的文件描述符集合从用户态拷贝到内核态，这样仍有一定的开销
- 判断哪些文件描述符有数据可读、可写或者有异常时，需要再次遍历所有的文件描述符，时间复杂度为 O(n)，效率不高

`select` 函数有这些缺点也不奇怪，因为它是最早的 IO 多路复用函数，随着时间的推移，就出现了第二个 IO 多路复用函数 `poll`。

### poll

> [!NOTE]
> 
> **前置知识**
>
> - **struct pollfd**
> 
>   pollfd 是一个结构体，用来存放需要监听的文件描述符，它的定义如下：
>
>   ``` c
>   struct pollfd {
>       int fd;         // 文件描述符
>       short events;   // 需要监听的事件
>       short revents;  // 返回的事件
>   };
>   ```
> 
>   events 和 revents 都是一个位掩码，用来表示需要监听的事件和返回的事件，它们可以是以下几个值的组合（使用或 `|` 运算符）：
>
>   - `POLLIN`：有数据可读，相当于 `POLLRDNORM | POLLRDBAND`
>   - `POLLOUT`：有数据可写，相当于 `POLLWRNORM | POLLWRBAND`
>   - `POLLERR`：有错误
>   - `POLLHUP`：挂起
>   - `POLLNVAL`：无效请求
>   - `POLLPRI`：有紧急数据
>   - `POLLRDBAND`：有带外数据可读
>   - `POLLRDNORM`：有普通数据可读
>   - `POLLWRBAND`：有带外数据可写
>   - `POLLWRNORM`：有普通数据可写

``` c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

- **fds**：pollfd 结构体数组，用来存放需要监听的文件描述符

- **nfds**：fds 数组的大小

- **timeout**：超时时间，单位为毫秒

- **函数返回值**：与 `select` 函数一样，返回就绪的文件描述符的数量，如果超时返回 0，出错返回 -1

可以看出，`poll` 比 `select` 少了两个参数，`fds` 既为输入参数，又为输出参数，`nfds` 为 fds 数组的大小，`timeout` 为超时时间。`poll` 函数在有 IO 事件或错误发生时，会将对应的 fd 的 revents 设置为对应的事件，我们只需要遍历 fds 数组，判断 revents 的值即可。

`revents` 的默认值为 0，因此我们在处理完一个 fd 后，只需要将其 revents 设置为 0，这相比于 `select` 函数的 fd_set 初始化要简洁一些。

`poll` 函数的最大亮点就是使用了 pollfd 结构体存放需要监听的文件描述符，所以 `poll` 函数不会有文件描述符个数的限制，相比于 `select` 函数能承受更高的并发量。

`poll` 函数的使用示例：

``` c linenums="1" hl_lines="21 25 29"
// ...
// 假设 pollfds 为一个 pollfd 数组，里面存放了需要监听的文件描述符，pollfds_size 为 pollfds 的大小

for (int i = 0; i < pollfds_size; i++) {
    pollfds[i].events = POLLIN | POLLOUT | POLLERR;  // 设置需要监听的事件
}

while(1) {
    int ret = poll(pollfds, pollfds_size, 5000);  // 设置超时时间为 5 秒
    if (ret == 0) {
        printf("poll timeout\n");
        continue;
    }
    if (ret < 0) {
        perror("poll error");
        break;
    }

    for (int i = 0; i < pollfds_size(); i++) {
        if (pollfds[i].revents & POLLIN) {
            pollfds[i].revents = 0;  // 处理完后将 revents 设置为 0
            // 处理读事件
        }
        if (pollfds[i].revents & POLLOUT) {
            pollfds[i].revents = 0;
            // 处理写事件
        }
        if (pollfds[i].revents & POLLERR) {
            pollfds[i].revents = 0;
            // 处理异常事件
        }
    }
}
```

`poll` 函数解决了 `select` 函数的 fd_set 大小有限、不可重用的问题，但是它仍有以下两个缺点：

- 每次调用 `poll` 函数都会将所有的文件描述符集合从用户态拷贝到内核态
- `poll` 函数与 `select` 函数一样，为了判断文件描述符是否就绪，需要遍历所有的文件描述符

是否能够不用每次调用都将文件描述符集合从用户态拷贝到内核态呢，而是只值传递一次呢？是否能够不用每次都遍历所有的文件描述符呢，而是只遍历就绪的文件描述符呢？答案是肯定的，这就是 **epoll**。

### epoll

> [!NOTE]
>
> **前置知识**
>
> - **epoll_event**
>
>   epoll_event 是一个结构体，用来存放需要监听的文件描述符，它的定义如下：
>
>   ``` c
>   typedef union epoll_data {
>       void *ptr;
>       int fd;
>       __uint32_t u32;
>       __uint64_t u64;
>   } epoll_data_t;
>
>   struct epoll_event {
>       __uint32_t events;  // 需要监听的事件
>       epoll_data_t data;  // 用户数据
>   };
>   ```
>
>   events 是需要监听的事件，它可以是以下几个值的组合（使用或 `|` 运算符）：
>
>   - `EPOLLIN`：有数据可读
>   - `EPOLLOUT`：有数据可写
>   - `EPOLLERR`：有错误
>   - `EPOLLHUP`：挂起
>   - `EPOLLPRI`：有 OOB 紧急数据
>   - `EPOLLRDHUP`：断开连接或者半关闭的情况，只有在边缘触发模式下才有效（关于边缘触发会在下文中介绍）
>   - `EPOLLONESHOT`：只监听一次事件，发生一次事件后，此文件描述符就不再收到事件通知。需要使用 `epoll_ctl` 的 `EPOLL_CTL_MOD` 修改事件
>   - `EPOLLET`：设置为边缘触发模式
>   - `EPOLLWAKEUP`：唤醒进程
>   - `EPOLLEXCLUSIVE`：独占模式
>
>   data 是用户数据，一般用于存放文件描述符。

epoll 共有三个函数：

- `epoll_create`：创建一个 epoll 实例

- `epoll_ctl`：控制 epoll 实例中的文件描述符

- `epoll_wait`：等待就绪的文件描述符

epoll 的 “e” 代表了 “event”，这是一个事件驱动的模型，即响应式的模型。内核不再需要 **主动** 轮询文件描述符，而是通过回调函数来实现通知进程，属于 **被动响应** 。

select 和 poll 都只有一个函数，所有功能都聚合到一个函数中，因此每次调用都需要传入需要监听的文件描述符，也就导致了每次都需要将文件描述符集合从用户态拷贝到内核态，增加了开销。

epoll 将 **维护文件描述符集合** 和 **判断文件描述符是否就绪** 两个功能进行了拆分，整个过程只需要使用 `epoll_ctl` 函数添加一次文件描述符，在 `epoll_wait` 函数中不再需要传入需要监听文件描述符，这样就大大减少了从用户态到内核态切换的开销。

![](https://s2.loli.net/2024/06/20/1NOKzukrtp5Aev6.jpg)

---

``` c
int epoll_create(int size);
```

> *Linux 手册：*
>
> *epoll_create() creates a new epoll(7) instance.  Since Linux 2.6.8, the size argument is ignored, but must be greater than zero; see HISTORY.*

根据 Linux 手册，`epoll_create` 函数的 `size` 参数在 Linux 2.6.8 之后被忽略，但是必须大于 0。所以我们可以将 `size` 参数设置为 `1`。根本原因是内核支持了动态扩容，所以 `size` 参数不再有意义，但为了向下兼容，`size` 参数仍然需要传入。

`epoll_create` 函数作用是创建一个 eventpoll 结构体（ *epoll 实例* ），函数返回值是一个非负的文件描述符，用于操作 eventpoll，如果返回 `-1` 则表示创建失败，并且 `errno` 会被设置为相应的错误码。这个文件描述符与其他文件描述符一样，需要通过 `close` 函数关闭，否则文件描述符会被耗尽。

eventpoll 是一个内核数据结构，由操作系统管理，其内部有三个重要的字段：

- **红黑树**：用来存放需要监听的文件描述符，时间复杂度为 `O(log n)`

- **就绪列表**：用来存放就绪的文件描述符，是一个双向循环链表，其作用就是告诉内核哪些文件描述符的哪些事件已经就绪

- **等待队列**：用来存放等待的进程/线程 

通过 eventpoll 可知，与 select 和 poll 采用的 **线性结构** 的方式存储文件描述符不同，eventpoll 维护了一个名为 `rbr` 的红黑树，用来存放需要监听的文件描述符，采用了 **树形结构** 存储文件描述符，将查询操作的时间复杂度从 `O(n)` 降低到了 `O(log n)`。


eventpoll 还维护了一个名为 `rdllist` 的就绪列表，用来存放就绪的文件描述符。当文件描述符就绪时，其会被 `rdllist` 引用，因此用户只需要获取 `rdllist` 中的内容即可知道哪些文件描述符就绪。

由于一个 epoll 实例可以被多个进程共享，即一个 `epfd` 可以被多个进程的 `epoll_wait` 函数调用，所以 eventpoll 还维护了一个名为 `wq` 的等待队列，用来存放等待中（阻塞中）的线程。

`epoll_create` 的工作原理大致如下：

- 为 `eventpoll` 申请空间，并且初始化其成员，如红黑树、就绪列表和等待队列等

- 为 `eventpoll` 申请一个未使用的文件描述符 `fd`

- 申请一个满足 VFS 的虚拟文件结构体 `file`

- `eventpoll` 与 `file` 关联，将 `fd` 和 `file` 关联起来

- 返回 `fd`

---

``` c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

- **epfd**：epoll 实例的文件描述符，即 `epoll_create` 函数返回的文件描述符

- **op**：操作类型，Linux 为我们定义了三个宏：  

    - `EPOLL_CTL_ADD`：添加一个文件描述符到 epoll 实例中

    - `EPOLL_CTL_MOD`：修改一个文件描述符在 epoll 实例中的事件

    - `EPOLL_CTL_DEL`：从 epoll 实例中删除一个文件描述符

- **fd**：需要监听的文件描述符

- **event**：epoll_event 结构体，用来存放需要监听的事件

- **函数返回值**：成功返回 0，失败返回 -1，并且 `errno` 会被设置为相应的错误码

`epoll_ctl` 函数（ *epoll control* ）用来控制 epoll 实例中的文件描述符，可以添加、修改或者删除文件描述符。

阅读下面代码语句：

``` c linenums="1"
epoll_ctl(A, EPOLL_CTL_ADD, B, &C);
epoll_ctl(A, EPOLL_CTL_DEL, B, NULL);
```

第一条语句的意思是在 epoll 实例 `A` 中注册（添加）文件描述符 `B`，并监听参数 `C` 中的事件。 第二条语句的意思是在 epoll 实例 `A` 中删除文件描述符 `B`。可以看出在删除文件描述符时，不需要传入事件参数。

由于 `epoll_ctl` 函数的实现里有互斥锁的存在，所以它是一个线程安全的函数，多个线程/进程可以并发调用 `epoll_ctl` 函数。

在调用 `epoll_ctl` 函数时，`epoll_ctl` 函数会为每个文件描述符创建一个 epitem 结构体，这个结构体用来存放文件描述符的信息，包括文件描述符、事件、回调函数等。这个 `epitem` 结构体会被添加到 eventpoll 的红黑树中。

epitem 结构体的定义如下：

``` c
struct epitem {
    struct rb_node rbn;  // 红黑树节点
    struct list_head rdllink;  // 就绪列表节点
    struct eppoll_entry *pwqlist;  // socket 等待队列
    struct eventpoll* ep;   // 指向 eventpoll 结构体
    struct epoll_event event;  // 事件
}
```

`epoll_ctl(ADD)` 函数的工作原理大致如下：

1. 将 `event` 拷贝到内核空间

2. 尝试在红黑树中查找 `fd` 对应的 `epitem` 结构体

3. 找不到则创建一个新的 `epitem`，并将 `epitem` 添加到红黑树中

4. 创建一个 `epitem` 对应的 `eppoll_entry` 用于链接到 socket 等待队列，并为 `eppoll_entry` 设置回调函数

5. 将 `eppoll_entry` 添加到 socket 等待队列中

6. 检查该 socket 的读写缓冲区和状态，如果有事件发生，则将 `epitem` 添加到就绪列表中，并且唤醒 `eventpoll.wq` 中的等待进程

7. 函数返回

---

``` c
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

- **epfd**：epoll 实例的文件描述符

- **events**：用来存放就绪的文件描述符

- **maxevents**：events 数组的大小

- **timeout**：超时时间，单位为毫秒

- **函数返回值**：返回就绪的文件描述符的数量，如果超时返回 0，出错返回 -1

其中 `epfd` 和 `timeout` 参数皆为输入参数，`events` 为输出参数，我们只需要传入空的 `epoll_event` 数组即可。`events` 数组会被填充就绪的文件描述符，所以其大小 `maxevents` 应该大于等于监听的文件描述符的数量。

`epoll_wait` 函数会检查 epoll 实例的就绪链表，如果就绪链表中有文件描述符，则将其添加到 `events` 数组中，然后返回就绪的文件描述符的数量。如果就绪链表为空，则会阻塞直到有文件描述符就绪或者超时。

`epoll_wait` 函数的工作原理大致如下：

1. 通过传入的 `epfd` 找到对应的 `eventpoll` 结构体

2. 判断 `eventpoll.rdllist` 是否有 `epitem`，有则代表有 `epitem` 就绪，但不一定是我们感兴趣的事件，所以需要进一步判断

3. 若 `rdllist` 不为空，则遍历它，获取 `epitem` 的 `socket` 后，在 `rdllist` 中删除此 `epitem`，接着判断 `socket` 具体触发的事件类型是否为我们需要监听的事件

4. 若是我们需要监听的事件，则将此事件拷贝回用户态的 `events` 数组中，函数返回

5. 若是水平触发模式，则将 `epitem` 重新添加到 `rdllist` 中，以便下次再次判断

6. 若 `rdllist` 为空，则判断是否超时，若超时则直接返回

7. 未超时则将该进程加入到 `eventpoll.wq` 进程等待队列中，并且将进程置为可中断睡眠状态，等待唤醒

8. 唤醒后，将进程设置为运行状态，并在 `wq` 中删除该进程

9. 重复 2-8 步骤，直到超时或者有文件描述符就绪

> [!NOTE]
>
> 注意，epoll 并没有采用共享内存的方式，阅读源码可知，epoll 采用了内核态和用户态的数据拷贝：
>
> ``` c
> epoll_put_uevent(__poll_t revents, __u64 data, struct epoll_event __user *uevent)
> {
>   // __put_user 是内核态到用户态的数据拷贝的函数
> 	if (__put_user(revents, &uevent->events) || __put_user(data, &uevent->data))
> 		return NULL;
> 	return uevent+1;
> }
> ```

> [!NOTE]
>
> **epoll 的两种触发模式**
>
> epoll 有两种触发模式，分别是 **水平触发模式**（ *Level Trigger* ）和 **边缘触发模式**（ *Edge Trigger* ）。水平触发模式是 epoll 默认的工作模式。
> 
> - **水平触发模式（ *LT模式* ）**：相当于时序电路中的高电平触发。关注点是 **数据**，只要读操作缓冲区不为空或写操作缓冲区不为满，`epoll_wait` 就会返回就绪事件。
>
> - **边缘触发模式（ *ET模式* ）**：相当于时序电路中的边沿出发。关注点是 **变化**，只有当有数据写接收进缓冲区或者从发送缓冲区读出数据时，`epoll_wait` 才会返回就绪事件。
>

`epoll` 函数的使用示例：

``` c linenums="1"
// ...
// 假设 epfd 为 epoll 实例的文件描述符，fds 为需要监听的文件描述符数组，fds_size 为 fds 的大小

struct epoll_event events[5]
int epfd = epoll_create(1);

for (int i = 0; i < 5; i++) {
    static struct epoll_event ev;
    ev.events = EPOLLIN | EPOLLOUT | EPOLLERR;  // 设置需要监听的事件
    ev.data.fd = fds[i];  // 设置需要监听的文件描述符
    epoll_ctl(epfd, EPOLL_CTL_ADD, ev.data.fd, &ev);  // 注册文件描述符和事件
}

while (1) {
    int ret = epoll_wait(epfd, events, 5, 5000);  // 设置超时时间为 5 秒
    if (ret == 0) {
        printf("epoll timeout\n");
        continue;
    }
    if (ret < 0) {
        perror("epoll error");
        break;
    }

    for (int i = 0; i < ret; i++) {
        if (events[i].events & EPOLLIN) {
            // 处理读事件，events[i].data.fd 为就绪的文件描述符
        }
        if (events[i].events & EPOLLOUT) {
            // 处理写事件，events[i].data.fd 为就绪的文件描述符
        }
        if (events[i].events & EPOLLERR) {
            // 处理异常事件，events[i].data.fd 为就绪的文件描述符
        }
    }
}
```


### kqueue


### IOCP


## 几个问题 

### 1. select 和 poll  一无是处了吗？

在学完这几个多路复用函数后，可能有些人会对 select 和 poll 感到失望。但是它们也有优点，比如 `select` 函数是跨平台的，无论你在 Linux、Windows 还是 macOS 下，都可以使用 `select` 函数。而改进的 IO 多路复用模型兼容性差，`epoll` 函数只能在 Linux 下使用，`kqueue` 函数只能在 FreeBSD 下使用，`IOCP` 函数只能在 Windows 下使用。

除此之外，未必所有的程序都需要处理大量的并发连接，如果一个程序只需要处理十几个连接，那么选择 `select` 函数是优于 `epoll` 函数的。

### 2. epoll 为什么使用红黑树而不是哈希表或 AVL 树？

其实历史版本的 Linux 内核中，`epoll` 使用的是哈希表，但后面改用了红黑树。

应用程序在调用 epoll 函数，尤其是当 epoll 监视的文件数量达到百万级的时候，对文件描述符的增删改查操作的频率很高，选用不同的数据结构带来的效率差异可能非常大。我们需要选择一个综合性能较好的数据结构。

| 数据结构 | 插入 | 删除 | 查找 | 特点 |
| --- | --- | --- | --- | --- |
| 红黑树 | O(log n) | O(log n) | O(log n) | 综合性能最优，最坏情况下时间复杂度为 O(log n) |
| 哈希表 | O(1) | O(1) | O(1) | ReHashing 开销大即扩展性差，且难以抉择哈希表大小 |
| AVL 树 | O(log n) | O(log n) | O(log n) | 查询效率稍快于红黑树，但插入删除效率稍慢于红黑树 |

综合考量，红黑树是最优选择。

### 3. 为什么是 `epitem` 的成员指向就绪列表，而不是列表元素指向 `epitem`？

有人可能会发现，在 `epitem` 结构体中，`rdllink` 指向就绪列表，而不是就绪列表的元素指向 `epitem`。这是由于 Linux 内核的链表设计哲学：“让万物包含链表，而不是链表包含万物。” Linux 内核可以通过 `container_of` 宏来获取链表元素的所在结构体地址，所以就绪列表的元素不需要指向 `epitem`。

## 参考资料

- [TCP/IP 网络编程（伊圣雨）](https://book.douban.com/subject/25911735)（书籍）

- [如果这篇文章说不清epoll的本质，那就过来掐死我吧！](https://zhuanlan.zhihu.com/p/63179839)（文章） 

- [源码解读epoll内核机制](https://gityuan.com/2019/01/06/linux-epoll/)（文章）

- [十个问题理解Linux epoll工作原理](https://mp.weixin.qq.com/s/h3CBZt2KEA-ScXFSKHaRBg)（文章）

- [【并发】IO多路复用select/poll/epoll介绍](https://www.bilibili.com/video/BV1qJ411w7du)（视频）

- [腾讯面试：请描述 select、poll、epoll 这三种IO多路复用技术的执行原理](https://www.bilibili.com/video/BV1gN411e7gd)（视频）


