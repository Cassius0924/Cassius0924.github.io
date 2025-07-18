# Socket 编程之 epoll 源码分析学习笔记

本文基于 Linux 6.9 内核源码进行分析。

## 几个数据结构

![数据结构](https://s2.loli.net/2024/06/30/fPingVOd2YcEFUX.jpg)

### eventpoll

这是 epoll 的主要数据结构，它用于存储 epoll 的相关信息，包括等待队列、就绪队列、红黑树等。

``` c linenums="1"
struct eventpoll {
	wait_queue_head_t wq;       // epoll 的等待队列：用于存储等待的进程/线程，指向等待队列头

	wait_queue_head_t poll_wait;// 这个 poll_wait 等待队列只有在 epoll 嵌套的情况下才会用到

	struct list_head rdllist;   // 就绪队列：用于存储就绪的 fd，指向就绪队列头

	struct rb_root_cached rbr;  // 红黑树：用于存储所有的 fd，指向红黑树根节点

	struct wakeup_source *ws;   // 一个唤醒源，用于唤醒进程
};
```

### epitem

epitem 的作用是将 fd、就绪队列、红黑树节点等信息封装在一起。

``` c linenums="1"
struct epitem {
    union {
        struct rb_node rbn;     // 红黑树节点，用于存储 fd，指向红黑树节点
        struct rcu_head rcu;    // 用于释放 epitem
    };

    struct list_head rdllink;   // 就绪队列节点，用于存储就绪的 fd，指向就绪队列节点

    struct eventpoll *ep;       // 指向 eventpoll

    struct epoll_filefd ffd;    // epoll 文件描述符

    struct wakeup_source *ws;   // 一个唤醒源，用于唤醒进程

    struct epoll_event event;   // 监听的事件
};
```

### ep_pqueue

给 poll 队列封装的结构体，用于存储 poll_table 和 epitem。

``` c linenums="1"
struct ep_pqueue {
	poll_table pt;
	struct epitem *epi;
};
```

### poll_table

poll_table 的作用是封装 poll 队列的处理函数和 key。

``` c linenums="1"
typedef struct poll_table_struct {
	poll_queue_proc _qproc;
	__poll_t _key;
} poll_table;
```

### eppoll_entry

``` c linenums="1"
struct eppoll_entry {
	struct eppoll_entry *next;  // 指向 epitem 的 epoll_entry

	struct epitem *base;    // 指向 epitem

	wait_queue_entry_t wait;    // 等待队列项

	wait_queue_head_t *whead;   // 指向 socket 等待队列头
};
```

### wait_queue_entry

wait_queue_entry 的作用是封装等待队列的相关信息。

``` c linenums="1"
struct wait_queue_entry {
	unsigned int		flags;
	void			*private;
	wait_queue_func_t	func;
	struct list_head	entry;
};
```

## epoll_create

``` c linenums="1"
// 定义 epoll_create 系统调用
SYSCALL_DEFINE1(epoll_create, int, size)
{
	if (size <= 0)
		return -EINVAL;

	return do_epoll_create(0);  // 调用 do_epoll_create 函数，[[具体实现见下面]]
}

static int do_epoll_create(int flags)
{
	int error, fd;
	struct eventpoll *ep = NULL;
	struct file *file;

	BUILD_BUG_ON(EPOLL_CLOEXEC != O_CLOEXEC);

	if (flags & ~EPOLL_CLOEXEC)
		return -EINVAL;

	error = ep_alloc(&ep);   // 为 eventpoll 分配内存，[[具体实现见下面]]
	if (error < 0)
		return error;

    // 创建 eventpoll 所需的东东。也就是一个文件结构和一个空闲的文件描述符。
	fd = get_unused_fd_flags(O_RDWR | (flags & O_CLOEXEC)); // 获取一个未使用的文件描述符
	if (fd < 0) {
		error = fd;
		goto out_free_ep;
	}

    // 创建一个文件结构
	file = anon_inode_getfile("[eventpoll]", &eventpoll_fops, ep, O_RDWR | (flags & O_CLOEXEC));
	if (IS_ERR(file)) {
		error = PTR_ERR(file);
		goto out_free_fd;
	}
	ep->file = file;    // 将文件结构赋值给 eventpoll

    // 将文件描述符和文件结构关联起来
	fd_install(fd, file);
	return fd;

out_free_fd:
	put_unused_fd(fd);
out_free_ep:
	ep_clear_and_put(ep);
	return error;
}

static int ep_alloc(struct eventpoll **pep)
{
	struct eventpoll *ep;

	ep = kzalloc(sizeof(*ep), GFP_KERNEL);  // 为 eventpoll 分配内存
	if (unlikely(!ep))
		return -ENOMEM;

	mutex_init(&ep->mtx);   // 初始化锁
	rwlock_init(&ep->lock);
	init_waitqueue_head(&ep->wq);   // 初始化等待队列
	init_waitqueue_head(&ep->poll_wait);    // 初始化 poll 等待队列
	INIT_LIST_HEAD(&ep->rdllist);   // 初始化就绪队列
	ep->rbr = RB_ROOT_CACHED;       
	ep->ovflist = EP_UNACTIVE_PTR;  
	ep->user = get_current_user();
	refcount_set(&ep->refcount, 1); // 引用计数初始化为 1

	*pep = ep;  // 将 eventpoll 赋值给 pep

	return 0;
}
```

<img src="https://s2.loli.net/2024/06/30/N4KUv2DYEW6OS5g.jpg" height=1000/>

## epoll_ctl

``` c linenums="1"
// 定义 epoll_ctl 系统调用
SYSCALL_DEFINE4(epoll_ctl, int, epfd, int, op, int, fd,
		struct epoll_event __user *, event)
{
	struct epoll_event epds;

    // ep_op_has_event 用于告诉 epoll_ctl 是否需要将 event 拷贝到内核空间
    // ep_op_has_event 的实现就一行，return op != EPOLL_CTL_DEL;
	if (ep_op_has_event(op) &&
	    copy_from_user(&epds, event, sizeof(struct epoll_event)))
        // 将 event 拷贝到内核空间，epds 是内核空间的 epoll_event
		return -EFAULT;

	return do_epoll_ctl(epfd, op, fd, &epds, false);    // 调用 do_epoll_ctl 函数，[[具体实现见下面]]
}

int do_epoll_ctl(int epfd, int op, int fd, struct epoll_event *epds, bool nonblock)
{
	int error;
	int full_check = 0;
	struct fd f, tf;
	struct eventpoll *ep;
	struct epitem *epi;
	struct eventpoll *tep = NULL;

	error = -EBADF;
	f = fdget(epfd);    // 获取 epfd 对应的文件描述符
	if (!f.file)
		goto error_return;

	tf = fdget(fd);     // 获取 fd 对应的文件描述符
    // f 为 epoll 文件描述符，tf 为我们要监听的文件描述符
	if (!tf.file)
		goto error_fput;

	error = -EPERM;
	if (!file_can_poll(tf.file))    // 检查 tf 是否支持 poll，不支持直接返回
		goto error_tgt_fput;

	if (ep_op_has_event(op))        // 如果 op 是 EPOLL_CTL_ADD 或 EPOLL_CTL_MOD
		ep_take_care_of_epollwakeup(epds);  // 保持线程唤醒状态，防止线程被挂起

	error = -EINVAL;
    // 如果 epfd 和 fd 是同一个文件描述符
    // 或者 epfd 不是 epoll 文件描述符，直接返回
	if (f.file == tf.file || !is_file_epoll(f.file))
		goto error_tgt_fput;

    // 省略一段检查错误的代码

	ep = f.file->private_data;  // 获取 epfd 中的 eventpoll 结构体

	error = epoll_mutex_lock(&ep->mtx, 0, nonblock);
	if (error)
		goto error_tgt_fput;

    // 此处省略了一坨用于处理 epoll 嵌套的情况的代码，不管

	epi = ep_find(ep, tf.file, fd); // 在红黑树中查找 fd 对应的 epitem，可能找不到

	error = -EINVAL;
	switch (op) {   // 根据 op 的不同，执行不同的操作
    // ADD
	case EPOLL_CTL_ADD:
		if (!epi) { // 红黑树里没有 fd 才插入
			epds->events |= EPOLLERR | EPOLLHUP;
			error = ep_insert(ep, epds, tf.file, fd, full_check);   // 插入 fd 到红黑树，[[具体实现见下面]]
		} else
			error = -EEXIST;
		break;
    // DEL
	case EPOLL_CTL_DEL:
		if (epi) {
			ep_remove_safe(ep, epi);    // 在红黑树删除 fd，[[具体实现见下面]]
			error = 0;
		} else {
			error = -ENOENT;
		}
		break;
    // MOD
	case EPOLL_CTL_MOD:
		if (epi) {
			if (!(epi->event.events & EPOLLEXCLUSIVE)) {
				epds->events |= EPOLLERR | EPOLLHUP;
				error = ep_modify(ep, epi, epds);   // 修改 fd 的监听事件，[[具体实现见下面]]
			}
		} else
			error = -ENOENT;
		break;
	}
	mutex_unlock(&ep->mtx);

error_tgt_fput:
	if (full_check) {
		clear_tfile_check_list();
		loop_check_gen++;
		mutex_unlock(&epnested_mutex);
	}
	fdput(tf);
error_fput:
	fdput(f);
error_return:
	return error;
}
```

### ep_insert

``` c linenums="1"
static int ep_insert(struct eventpoll *ep, const struct epoll_event *event,
		     struct file *tfile, int fd, int full_check)
{
	int error, pwake = 0;
	__poll_t revents;
	struct epitem *epi;
	struct ep_pqueue epq;       // 一个 epitem 和回调函数的包装
	struct eventpoll *tep = NULL;   // 这个是用于处理 epoll 嵌套的情况

    // 省略一段不重要的代码，问题不大

    // 为 epitem 分配内存
    if (!(epi = kmem_cache_zalloc(epi_cache, GFP_KERNEL))) {
		percpu_counter_dec(&ep->user->epoll_watches);
		return -ENOMEM;
	}

    // 构造 epitem
	INIT_LIST_HEAD(&epi->rdllink);
	epi->ep = ep;
	ep_set_ffd(&epi->ffd, tfile, fd);
	epi->event = *event;
	epi->next = EP_UNACTIVE_PTR;

    // 省略不重要的代码

	ep_rbtree_insert(ep, epi);  // 将 epitem 插入红黑树

    // 省略不重要的代码

	epq.epi = epi;  // 将 epitem 放入 ep_pqueue 这个队列
    // 将 ep_ptable_queue_proc 函数赋值给 ep_pqueue.pt 的函数指针，用于处理 poll 队列，会在下一行的 ep_item_poll 函数中回调
	init_poll_funcptr(&epq.pt, ep_ptable_queue_proc);   

	revents = ep_item_poll(epi, &epq.pt, 1);    // 调用 ep_item_poll 函数 [[具体实现见下面]]

    // 省略不重要的代码

    // 如果 fd 已就绪，将其放入就绪队列
	if (revents && !ep_is_linked(epi)) {
		list_add_tail(&epi->rdllink, &ep->rdllist); // 加入到就绪队列尾部
		ep_pm_stay_awake(epi);  // 保持唤醒状态

		if (waitqueue_active(&ep->wq))
			wake_up(&ep->wq);   // 唤醒 wq 等待队列
        // 省略和 epoll 嵌套有关的代码
	}
	write_unlock_irq(&ep->lock);

    // 省略和 epoll 嵌套有关的代码
	return 0;
}

static __poll_t ep_item_poll(const struct epitem *epi, poll_table *pt, int depth) 
{
	struct file *file = epi_fget(epi);
	__poll_t res;

	if (!file)
		return 0;

	pt->_key = epi->event.events;
	if (!is_file_epoll(file))
        // 如果不是 epoll fd，即普通 fd（socket fd），调用 vfs_poll 函数
        // 大部分都是这一情况!!!
		res = vfs_poll(file, pt);
	else
        // 这种情况属于嵌套 epoll，调用 __ep_eventpoll_poll 函数
		res = __ep_eventpoll_poll(file, pt, depth);     // res 就是 socket 所有就绪的事件
	fput(file);
	return res & epi->event.events; // 这里才进一步判断是否是我们关心的事件（监听的事件）
}

// 普通 fd 的处理函数
static inline __poll_t vfs_poll(struct file *file, struct poll_table_struct *pt)
{
	if (unlikely(!file->f_op->poll))
		return DEFAULT_POLLMASK;
    // 这里实际上是调用了 socket 的 poll 函数，然后会根据 socket 类型调用不同的函数
    // 如 tcp_poll、udp_poll 等
	return file->f_op->poll(file, pt);  // 先通过 sock_poll，然后这里以 tcp_poll 为例 [[具体实现见下面]]
}

static __poll_t sock_poll(struct file *file, poll_table *wait)
{
	struct socket *sock = file->private_data;
	const struct proto_ops *ops = READ_ONCE(sock->ops);

    // 略一部分，flag 与 busy_poll 有关，不重要

	return ops->poll(file, sock, wait) | flag;  // 调用 tcp_poll 函数 [[具体实现见下面]]
}

__poll_t tcp_poll(struct file *file, struct socket *sock, poll_table *wait)
{
	__poll_t mask;
	struct sock *sk = sock->sk;
	const struct tcp_sock *tp = tcp_sk(sk);
	u8 shutdown;
	int state;

	sock_poll_wait(file, sock, wait);   // sock_poll_wait 函数 [[具体实现见下面]]

	state = inet_sk_state_load(sk);
	if (state == TCP_LISTEN)    // 如果是监听状态，查看 accept 队列是否为空
		return inet_csk_listen_poll(sk);    // 具体实现是如果不为空，则返回 EPOLLIN | EPOLLRDNORM

    // 不是监听状态，就是连接状态，就有可能读写就绪
    // 下面一大段就是根据连接状态、接收缓冲区、发送缓冲区等情况来判断是否可读可写
    // 这里就省略了
    // ...

	return mask;
}

static inline void sock_poll_wait(struct file *filp, struct socket *sock, poll_table *p)
{
	if (!poll_does_not_wait(p)) {
		poll_wait(filp, &sock->wq.wait, p);   // [[具体实现见下面]]
		smp_mb();
	}
}

static inline void poll_wait(struct file * filp, wait_queue_head_t * wait_address, poll_table *p)
{
	if (p && p->_qproc && wait_address)
		p->_qproc(filp, wait_address, p); // 到这，实际上是调用了 ep_ptable_queue_proc 函数
}

// 重点是这个函数，这是用于将我们的等待队列添加到目标文件唤醒列表的回调。
static void ep_ptable_queue_proc(struct file *file, wait_queue_head_t *whead, poll_table *pt)
{
    // container_of 是 Linux 的一个骚操作，用于通过结构体成员获取其所在的结构体的指针，有兴趣可以自行搜索
    // 这里是通过 pt 获取 ep_pqueue 结构体的指针，进而获取 epitem
	struct ep_pqueue *epq = container_of(pt, struct ep_pqueue, pt);
	struct epitem *epi = epq->epi;
	struct eppoll_entry *pwq;

	if (unlikely(!epi))
		return;

    // 为 pwq 分配内存
	pwq = kmem_cache_alloc(pwq_cache, GFP_KERNEL);
	if (unlikely(!pwq)) {
		epq->epi = NULL;
		return;
	}

	init_waitqueue_func_entry(&pwq->wait, ep_poll_callback);    // 将我们的回调函数赋值给 wait
	pwq->whead = whead;
	pwq->base = epi;
	if (epi->event.events & EPOLLEXCLUSIVE)
		add_wait_queue_exclusive(whead, &pwq->wait);
	else
		add_wait_queue(whead, &pwq->wait);  // 将 pwq 加入到 socket 的等待队列
        // 这个 pwq 会在 socket 网卡有数据时被中断程序调用，然后调用 pwq 里存的回调函数，即 ep_poll_callback
	pwq->next = epi->pwqlist;
	epi->pwqlist = pwq;
}
```

<img src="https://s2.loli.net/2024/06/30/7MiSVynwNpEYQcR.jpg" height=1700/>

### ep_modify

### ep_remove_safe

## epoll_wait

``` c linenums="1"
SYSCALL_DEFINE4(epoll_wait, int, epfd, struct epoll_event __user *, events,
		int, maxevents, int, timeout)
{
	struct timespec64 to;

    // 将 timeout 转换为 timespec64 结构体
    // do_epoll_wait [[具体实现见下面]]
	return do_epoll_wait(epfd, events, maxevents, ep_timeout_to_timespec(&to, timeout));
}

static int do_epoll_wait(int epfd, struct epoll_event __user *events,
			 int maxevents, struct timespec64 *to)
{
	int error;
	struct fd f;
	struct eventpoll *ep;

    // 这里的 EP_MAX_EVENTS = INT_MAX / sizeof(struct epoll_event)
	if (maxevents <= 0 || maxevents > EP_MAX_EVENTS) // maxevents 必须大于 0 且小于 EP_MAX_EVENTS
		return -EINVAL;

    // 验证用户传递的区域是否可写
	if (!access_ok(events, maxevents * sizeof(struct epoll_event)))
		return -EFAULT;

	f = fdget(epfd);    // 获取 epfd 对应的文件描述符
	if (!f.file)
		return -EBADF;

	error = -EINVAL;
	if (!is_file_epoll(f.file)) // 确保 epfd 是 epoll 文件描述符
		goto error_fput;

	ep = f.file->private_data;  // 取出 eventpoll 结构体

	error = ep_poll(ep, events, maxevents, to); // 调用 ep_poll 函数，[[具体实现见下面]]

error_fput:
	fdput(f);
	return error;
}

static int ep_poll(struct eventpoll *ep, struct epoll_event __user *events,
		   int maxevents, struct timespec64 *timeout)
{
	int res, eavail, timed_out = 0;
	u64 slack = 0;
	wait_queue_entry_t wait;
	ktime_t expires, *to = NULL;

	lockdep_assert_irqs_enabled();

    // 计算超时时间
	if (timeout && (timeout->tv_sec | timeout->tv_nsec)) {
        // 这里是用户指定了超时时间
		slack = select_estimate_accuracy(timeout);
		to = &expires;
		*to = timespec64_to_ktime(*timeout);    // 将时间转换为 ktime_t 结构体
	} else if (timeout) {
        // 这里是用户没有指定超时时间，将会在检查一次事件后返回
		timed_out = 1;
	}

	eavail = ep_events_available(ep);   // 检查是否有事件可用 [[具体实现见下面]]

	while (1) {
		if (eavail) {   // 有事件可用
            // 尝试将事件传递给用户空间
			res = ep_send_events(ep, events, maxevents);
			if (res)    // 传递成功
				return res; // 这里是有事件情况的函数出口
		}

		if (timed_out)
			return 0;   // 这里是超时情况的函数出口

        // 这里省略了 busy loop 的代码

		if (signal_pending(current))    // 如果有信号挂起，直接返回
			return -EINTR;

		init_wait(&wait);   // 初始化等待队列项，这里就和 ep.wq 有关了
        // 设置回调函数，在唤醒进程后自动删除该进程在 ep.wq 中对应的项
		wait.func = ep_autoremove_wake_function;

		write_lock_irq(&ep->lock);

		__set_current_state(TASK_INTERRUPTIBLE);    // 设置当前进程状态为 可中断睡眠

		eavail = ep_events_available(ep);   // 再次检查是否有事件可用
		if (!eavail)    // 没有事件可用，将当前进程加入到等待队列
			__add_wait_queue_exclusive(&ep->wq, &wait);

		write_unlock_irq(&ep->lock);

		if (!eavail)
            // 没事件，将当前进程挂起，如果有中断信号，则会被唤醒，超时也会被唤醒。
            // 如果是被有事件的中断信号唤醒的，
            // 则先会调用 epitem 的回调函数，即在 epoll_ctl 中注册的 ep_poll_callback 函数，
            // 将就绪的 fd 放入 rdllist 就绪队列，再唤醒刚刚加入到 ep->wq 的线程。
            // 然后再返回。
			timed_out = !schedule_hrtimeout_range(to, slack, HRTIMER_MODE_ABS);
		__set_current_state(TASK_RUNNING);  // 设置当前进程状态为运行态

		eavail = 1;

		if (!list_empty_careful(&wait.entry)) {
			write_lock_irq(&ep->lock);
			if (timed_out)
				eavail = list_empty(&wait.entry);
			__remove_wait_queue(&ep->wq, &wait);
			write_unlock_irq(&ep->lock);
		}
	}
}

static inline int ep_events_available(struct eventpoll *ep)
{
	return !list_empty_careful(&ep->rdllist) ||
		READ_ONCE(ep->ovflist) != EP_UNACTIVE_PTR;
}

static int ep_send_events(struct eventpoll *ep, struct epoll_event __user *events, int maxevents)
{
	struct epitem *epi, *tmp;
	LIST_HEAD(txlist);  // txlist 是用来向用户空间拷贝数据的链表
	poll_table pt;
	int res = 0;

	if (fatal_signal_pending(current))
		return -EINTR;

	init_poll_funcptr(&pt, NULL);

	mutex_lock(&ep->mtx);
	ep_start_scan(ep, &txlist); // 这个函数用于将 ep->rdllist 拷贝到 txlist

	list_for_each_entry_safe(epi, tmp, &txlist, rdllink) {  // 遍历 txlist
		struct wakeup_source *ws;
		__poll_t revents;

		if (res >= maxevents)
			break;

        // 确保进程唤醒
		ws = ep_wakeup_source(epi);
		if (ws) {
			if (ws->active)
				__pm_stay_awake(ep->ws);
			__pm_relax(ws);
		}

		list_del_init(&epi->rdllink);   // 从就绪队列中删除 epi

        // 注意！rdllist 里的 epi 有事件就绪，但不一定是我们感兴趣的事件！
        // 所以这里查询 socket 的具体事件，是否为我们感兴趣的事件！
        // 也就是说，rdllist 只能告诉我们可能有任何事件发生，
        // 但具体是什么事件，是否是我们感兴趣的事件，需要通过 ep_item_poll 函数来判断
		revents = ep_item_poll(epi, &pt, 1);
		if (!revents)
			continue;   // 没有我们感兴趣事件，遍历下一个 epi

        // 有我们感兴趣的事件，将 revents 和 epi->event.data 其拷贝到用户空间的 events 里
		events = epoll_put_uevent(revents, epi->event.data, events);    // [[具体实现见下面]]
		if (!events) {
			list_add(&epi->rdllink, &txlist);   // 拷贝失败，将 epi 放回 txlist
			ep_pm_stay_awake(epi);
			if (!res)
				res = -EFAULT;
			break;
		}
		res++;
		if (epi->event.events & EPOLLONESHOT) // 处理 EPOLLONESHOT 的情况
			epi->event.events &= EP_PRIVATE_BITS;
		else if (!(epi->event.events & EPOLLET)) {  // 处理 EPOLLET 的情况
            // 如果是 水平触发 模式，将 epi 放回就绪队列，等待下一次事件
			list_add_tail(&epi->rdllink, &ep->rdllist);
			ep_pm_stay_awake(epi);
		}
	}
    // 要理解这个函数，我们得知道，如果在扫描过程中有 fd 要加入就绪队列，
    // 由于 rdllist 已经被锁住了，所以会将这个 fd 放入 ovflist 中，
    // 这个函数就是将 ovflist 中的 fd 放回入 rdllist 中。
	ep_done_scan(ep, &txlist);
	mutex_unlock(&ep->mtx);

	return res;
}

epoll_put_uevent(__poll_t revents, __u64 data, struct epoll_event __user *uevent)
{
    // 注意这里依然使用的是 __put_user 函数，即从内核空间拷贝到用户空间，并没有使用共享内存！
	if (__put_user(revents, &uevent->events) || __put_user(data, &uevent->data))
		return NULL;
	return uevent+1;
}
```

<img src="https://s2.loli.net/2024/06/30/yjO4xHCiZMq5XpL.jpg" height=1400/>

### ep_poll_callback

``` c linenums="1"
// 这是传递给等待队列唤醒机制的回调函数。当存储的文件描述符有事件要报告时，它会被调用。
// 其实就干两件事，一是将就绪的 fd 放入就绪队列，二是唤醒等待队列。
static int ep_poll_callback(wait_queue_entry_t *wait, unsigned mode, int sync, void *key)
{
	int pwake = 0;
	struct epitem *epi = ep_item_from_wait(wait);
	struct eventpoll *ep = epi->ep;
	__poll_t pollflags = key_to_poll(key);
	unsigned long flags;
	int ewake = 0;

    // 一通检查猛如虎
	read_lock_irqsave(&ep->lock, flags);
	ep_set_busy_poll_napi_id(epi);
	if (!(epi->event.events & ~EP_PRIVATE_BITS))
		goto out_unlock;
	if (pollflags && !(pollflags & epi->event.events))
		goto out_unlock;

	if (READ_ONCE(ep->ovflist) != EP_UNACTIVE_PTR) {
		if (chain_epi_lockless(epi))
			ep_pm_stay_awake_rcu(epi);
	} else if (!ep_is_linked(epi)) {    // 如果 epi 不在就绪队列
        // fd 就绪，将其放入就绪队列
		if (list_add_tail_lockless(&epi->rdllink, &ep->rdllist))
			ep_pm_stay_awake_rcu(epi);
	}

    // 放入就绪队列后，唤醒等待队列
	if (waitqueue_active(&ep->wq)) {
		if ((epi->event.events & EPOLLEXCLUSIVE) && !(pollflags & POLLFREE)) {
			switch (pollflags & EPOLLINOUT_BITS) {
			case EPOLLIN:
				if (epi->event.events & EPOLLIN)
					ewake = 1;
				break;
			case EPOLLOUT:
				if (epi->event.events & EPOLLOUT)
					ewake = 1;
				break;
			case 0:
				ewake = 1;
				break;
			}
		}
		wake_up(&ep->wq);   // 唤醒进程
	}
    // 省略和 epoll 嵌套有关的代码

out_unlock:
	read_unlock_irqrestore(&ep->lock, flags);

    // 省略和 epoll 嵌套有关的代码

	return ewake;
}
```

<img src="https://s2.loli.net/2024/06/30/VKyeuZSxjkmwn1t.jpg" height=700/>