## 阻塞IO模型

应用程序发起一个recvfrom系统调用，如果没有数据到达，那么应用程序将处于一个阻塞的状态，直到数据到达，然后将到达的数据拷贝到程序buffer中。当系统调用结束之后，应用程序才能继续往下执行。

很显然，一旦发生系统调用，必定存在用户态和内核态的转换。

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/figure_6.1.png)

## 非阻塞IO模型

当socket被设置成非阻塞IO时，也就相对相当于告诉系统，当一个IO请求没有完成，也就是没有数据到达时，进程并不会阻塞，而是直接返回error；然后应用进程会不断进行recvfrom系统调用，知道return OK并获取到数据。

- 对于前面的三次系统调用，这里没有数据返回，而是操作系统内核会立刻返回error（EWOULDBLOCK）。
- 对于第四次系统调用，数据达到，然后将数据复制到buffer中，应用程序继续往下执行。

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/figure_6.2.png)

## IO多路复用模型

对于IO多路复用，我们在系统调用中调用select/poll中的一个并发生阻塞，而不是IO发生阻塞。

与阻塞IO相比：

- 缺点：使用select/poll需要两次系统调用
- 优点：能够监听多个文件描述符事件

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/figure_6.3.png)

## 信号驱动IO模型

信号驱动IO模型使用信号量，并告诉系统一旦有文件描述符事件就绪就通知应用程序。

- 我们首先为信号驱动的 I/O 启用套接字并使用 sigaction 系统调用安装信号处理程序。这个系统调用的返回是立即的，我们的过程继续进行；它没有被阻塞。

- 当数据可读时，系统会向应用程序发送一个SIGIO信号，我们可以做：
  - 通过信号处理器调用recvfrom读取数据，然后通知主程序数据已准备完毕

![Figure 6.4. Signal-Driven I/O model.](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/figure_6.4.png)

## 异步IO模型

这个模型与信号驱动IO模型的主要区别在于：信号驱动IO是由内核通知应用程序何时启动一个IO操作，而异步IO是由内核通知应用程序IO操作何时完成。

![Figure 6.5. Asynchronous I/O model.](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/figure_6.5.png)



## IO多路复用select

### bitmap

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220421191639045.png" alt="image-20220421191639045" style="zoom:30%;" />

### Select System Call

- select最多能监听1024个套接字，操作系统定义select的最大fd为1024个。
- 调用select的程序其实可以认为是被阻塞了的，但是并不是因为IO阻塞导致的，而是由于select阻塞造成的。

```c
/**
 * nfds表示fd_set中含有1位数的最大值
 * readfds 表示读文件描述符集合
 * writefds 表示写文件描述符集合
 * exceptfds表示异常文件描述符集合
 * timeout 超时时间
 */
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

使用的例子：

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <wait.h>
#include <signal.h>
#include <errno.h>
#include <sys/select.h>
#include <sys/time.h>
#include <unistd.h>

#define MAXBUF 256

void child_process(void)
{
  sleep(2);
  char msg[MAXBUF];
  struct sockaddr_in addr = {0};
  int n, sockfd,num=1;
  srandom(getpid());
  /* Create socket and connect to server */
  sockfd = socket(AF_INET, SOCK_STREAM, 0);
  addr.sin_family = AF_INET;
  addr.sin_port = htons(2000);
  addr.sin_addr.s_addr = inet_addr("127.0.0.1");

  connect(sockfd, (struct sockaddr*)&addr, sizeof(addr));

  printf("child {%d} connected \n", getpid());
  while(1){
        int sl = (random() % 10 ) +  1;
        num++;
     	sleep(sl);
  	sprintf (msg, "Test message %d from client %d", num, getpid());
  	n = write(sockfd, msg, strlen(msg));	/* Send message */
  }

}

int main()
{
  char buffer[MAXBUF];
  int fds[5];
  struct sockaddr_in addr;
  struct sockaddr_in client;
  int addrlen, n,i,max=0;;
  int sockfd, commfd;
  fd_set rset;
  for(i=0;i<5;i++)
  {
  	if(fork() == 0)
  	{
  		child_process();
  		exit(0);
  	}
  }
	//准备文件描述符
  sockfd = socket(AF_INET, SOCK_STREAM, 0);
  memset(&addr, 0, sizeof (addr));
  addr.sin_family = AF_INET;
  addr.sin_port = htons(2000);
  addr.sin_addr.s_addr = INADDR_ANY;
  bind(sockfd,(struct sockaddr*)&addr ,sizeof(addr));
  listen (sockfd, 5); 

  for (i=0;i<5;i++) 
  {
    memset(&client, 0, sizeof (client));
    addrlen = sizeof(client);
    fds[i] = accept(sockfd,(struct sockaddr*)&client, &addrlen);
    if(fds[i] > max)
    	max = fds[i];
  }
  //执行select，监听文件描述符对应的文件是否有数据到达
  while(1){
  //On return , select changes the set to contains only the file descriptors that is ready so we need to build the set again every iteration.
	FD_ZERO(&rset);
  	for (i = 0; i< 5; i++ ) {
  		FD_SET(fds[i],&rset);
  	}

   	puts("round again");
	select(max+1, &rset, NULL, NULL, NULL);

	for(i=0;i<5;i++) {
		if (FD_ISSET(fds[i], &rset)){
			memset(buffer,0,MAXBUF);
			read(fds[i], buffer, MAXBUF);
			puts(buffer);
		}
	}	
  }
  return 0;
}
```

### 总结

> - 每一次调用都需要重新创建set
> - 函数将会一直检查每一个bit位直到达到max
> - 当select返回时，需要遍历文件描述符判断哪个文件描述符数据已就绪
> - select最大的优势是可移植性强



## IO多路复用poll

### pollfd

```c
struct pollfd {
      int fd;//文件描述符
      short events; //被指定的事件类型
      short revents;//返回的事件类型
};
```



### Poll System Call

```c
int poll (struct pollfd *fds, unsigned int nfds, int timeout);
```

例子：

```c
for (i=0;i<5;i++) 
  {
    memset(&client, 0, sizeof (client));
    addrlen = sizeof(client);
    pollfds[i].fd = accept(sockfd,(struct sockaddr*)&client, &addrlen);
    pollfds[i].events = POLLIN;
  }
  sleep(1);
  while(1){
  	puts("round again");
    poll(pollfds, 5, 50000);

    for(i=0;i<5;i++) {
      if (pollfds[i].revents & POLLIN){
        pollfds[i].revents = 0;
        memset(buffer,0,MAXBUF);
        read(pollfds[i].fd, buffer, MAXBUF);
        puts(buffer);
      }
    }
  }
```

### 总结

> - poll()使用pollfd结构体，传递结构体数组给内核，让内核监听结构体中的fd，一旦有fd就绪，则令revents等于events，这样就可以通过revents filed判断哪些文件描述符可读写。
> - 通过pollfd结构体，可以区分描述符绑定的事件和返回事件，避免每一次poll调用都重建set集合。



## IO多路复用epoll

使用epoll分为三步：

- 通过epoll_create()函数在内核中创建白板
- 通过epoll_ctl()函数在白板中添加或删除文件描述符
- 通过epoll_wait()函数监听文件描述符事件



例子：

```C
struct epoll_event events[5];
  int epfd = epoll_create(10);
  ...
  ...
  for (i=0;i<5;i++) 
  {
    static struct epoll_event ev;
    memset(&client, 0, sizeof (client));
    addrlen = sizeof(client);
    ev.data.fd = accept(sockfd,(struct sockaddr*)&client, &addrlen);
    ev.events = EPOLLIN;
    epoll_ctl(epfd, EPOLL_CTL_ADD, ev.data.fd, &ev); 
  }
  
  while(1){
  	puts("round again");
  	nfds = epoll_wait(epfd, events, 5, 10000);
	
	for(i=0;i<nfds;i++) {
			memset(buffer,0,MAXBUF);
			read(events[i].data.fd, buffer, MAXBUF);
			puts(buffer);
	}
  }
```



### epoll实现原理

#### epoll的创建

要使用epoll需要调用`epoll_create()` 函数创建一个epoll句柄

```c
int epoll_create(int size);//size是历史遗留问题，现在无效了
```

当用户调用`epoll_create()` 函数时，会进入内核空间，并且调用`sys_epoll_create()` 内核函数来创建epoll句柄。

```c
asmlinkage long sys_epoll_create(int size)
{
    int error, fd = -1;
    struct eventpoll *ep;

    error = -EINVAL;
    if (size <= 0 || (error = ep_alloc(&ep)) < 0) {
        fd = error;
        goto error_return;
    }

    fd = anon_inode_getfd("[eventpoll]", &eventpoll_fops, ep);
    if (fd < 0)
        ep_free(ep);

error_return:
    return fd;
}
```

> `sys_epoll_create()` 内核函数主要做两件事情：
>
> 1. 调用 `ep_alloc()` 函数创建并初始化eventpoll对象
> 2. 调用 `anon_inode_getfd()` 函数把eventpoll对象映射到一个文件句柄，并返回这个文件句柄



**eventpoll对象**

```c
struct eventpoll {
    ...
    wait_queue_head_t wq;//等待队列，当调用epoll_wait(fd) 时，会把进程添加到eventpoll 对象的 wq 等待队列中
    ...
    struct list_head rdllist;//保存已经就绪的文件列表
    struct rb_root rbr;//使用红黑树来管理所有被监听的事件
    ...
};
```

先来说明一下 `eventpoll` 对象各个成员的作用：

1. `wq`: 等待队列，当调用 `epoll_wait(fd)` 时会把进程添加到 `eventpoll` 对象的 `wq` 等待队列中。
2. `rdllist`: 保存已经就绪的文件列表。
3. `rbr`: 使用红黑树来管理所有被监听的文件。

下图展示了 `eventpoll` 对象与被监听的文件关系：

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1gpc5c1gib3j30j30k5q4a.jpg" alt="img" style="zoom:50%;" />

#### 向epoll添加文件句柄

通过调用 `epoll_ctl()` 函数可以向epoll添加要监听的文件

```c
 long epoll_ctl(int epfd, int op, int fd,struct epoll_event *event);
```

下面说明一下各个参数的作用：

1. `epfd`: 通过调用 `epoll_create()` 函数返回的文件句柄。

2. `op`: 要进行的操作，有3个选项：

   - `EPOLL_CTL_ADD`：表示要进行添加操作。
   - `EPOLL_CTL_DEL`：表示要进行删除操作。
   - `EPOLL_CTL_MOD`：表示要进行修改操作。

3. `fd`: 要监听的文件句柄。

4. `event`: 告诉内核需要监听什么事。其定义如下：

   ```
   struct epoll_event {
       __uint32_t events;  /* Epoll events */
       epoll_data_t data;  /* User data variable */
   };
   Copy
   ```

   `events`可以是以下几个宏的集合：

- `EPOLLIN` ：表示对应的文件句柄可以读（包括对端SOCKET正常关闭）；
- `EPOLLOUT`：表示对应的文件句柄可以写；
- `EPOLLPRI`：表示对应的文件句柄有紧急的数据可读（这里应该表示有带外数据到来）；
- `EPOLLERR`：表示对应的文件句柄发生错误；
- `EPOLLHUP`：表示对应的文件句柄被挂断；
- `EPOLLET`：将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)来说的。
- `EPOLLONESHOT`：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里。

`data` 用来保存用户自定义数据。

`epoll_ctl()` 函数会调用 `sys_epoll_ctl()` 内核函数，`sys_epoll_ctl()` 内核函数的实现如下：

```c
asmlinkage long sys_epoll_ctl(int epfd, int op, 
    int fd, struct epoll_event __user *event)
{
    ...
    file = fget(epfd);
    tfile = fget(fd);
    ...
    ep = file->private_data;

    mutex_lock(&ep->mtx);

    epi = ep_find(ep, tfile, fd);

    error = -EINVAL;
    switch (op) {
    case EPOLL_CTL_ADD:
        if (!epi) {
            epds.events |= POLLERR | POLLHUP;
						
            error = ep_insert(ep, &epds, tfile, fd);
        } else
            error = -EEXIST;
        break;
    ...
    }
    mutex_unlock(&ep->mtx);

    ...
    return error;
}
```

`sys_epoll_ctl()` 函数会根据传入不同 `op` 的值来进行不同操作，比如传入 `EPOLL_CTL_ADD` 表示要进行添加操作，那么就调用 `ep_insert()` 函数来进行添加操作。

我们继续来分析添加操作 `ep_insert()` 函数的实现：

```c
static int ep_insert(struct eventpoll *ep, struct epoll_event *event,
             struct file *tfile, int fd)
{
    ...
    error = -ENOMEM;
    // 申请一个 epitem 对象
    if (!(epi = kmem_cache_alloc(epi_cache, GFP_KERNEL)))
        goto error_return;

    // 初始化 epitem 对象
    INIT_LIST_HEAD(&epi->rdllink);
    INIT_LIST_HEAD(&epi->fllink);
    INIT_LIST_HEAD(&epi->pwqlist);
    epi->ep = ep;
    ep_set_ffd(&epi->ffd, tfile, fd);
    epi->event = *event;
    epi->nwait = 0;
    epi->next = EP_UNACTIVE_PTR;

    epq.epi = epi;
    // 等价于: epq.pt->qproc = ep_ptable_queue_proc
    init_poll_funcptr(&epq.pt, ep_ptable_queue_proc);

    // 调用被监听文件的 poll 接口. 
    // 这个接口又各自文件系统实现, 如socket的话, 那么这个接口就是 tcp_poll().
    revents = tfile->f_op->poll(tfile, &epq.pt);
    ...
    ep_rbtree_insert(ep, epi); // 把 epitem 对象添加到epoll的红黑树中进行管理

    spin_lock_irqsave(&ep->lock, flags);

    // 如果被监听的文件已经可以进行对应的读写操作
    // 那么就把文件添加到epoll的就绪队列 rdllink 中, 并且唤醒调用 epoll_wait() 的进程.
    if ((revents & event->events) && !ep_is_linked(&epi->rdllink)) {
        list_add_tail(&epi->rdllink, &ep->rdllist);

        if (waitqueue_active(&ep->wq))
            wake_up_locked(&ep->wq);
        if (waitqueue_active(&ep->poll_wait))
            pwake++;
    }

    spin_unlock_irqrestore(&ep->lock, flags);
    ...
    return 0;
    ...
}
```

被监听的文件是通过 `epitem` 对象进行管理的，也就是说被监听的文件会被封装成 `epitem` 对象，然后会被添加到 `eventpoll` 对象的红黑树中进行管理（如上述代码中的 `ep_rbtree_insert(ep, epi)`）。

`tfile->f_op->poll(tfile, &epq.pt)` 这行代码的作用是调用被监听文件的 `poll()` 接口，如果被监听的文件是一个socket句柄，那么就会调用 `tcp_poll()`，我们来看看 `tcp_poll()` 做了什么操作：

```c
unsigned int tcp_poll(struct file *file, struct socket *sock, poll_table *wait)
{
    struct sock *sk = sock->sk;
    ...
    poll_wait(file, sk->sk_sleep, wait);
    ...
    return mask;
}
```

每个 `socket` 对象都有个等待队列（`waitqueue`）,用于存放等待 socket 状态更改的进程。

从上述代码可以知道，`tcp_poll()` 调用了 `poll_wait()` 函数，而 `poll_wait()` 最终会调用 `ep_ptable_queue_proc()` 函数，`ep_ptable_queue_proc()` 函数实现如下：

```c
static void ep_ptable_queue_proc(struct file *file, 
    wait_queue_head_t *whead, poll_table *pt)
{
    struct epitem *epi = ep_item_from_epqueue(pt);
    struct eppoll_entry *pwq;

    if (epi->nwait >= 0 && (pwq = kmem_cache_alloc(pwq_cache, GFP_KERNEL))) {
        init_waitqueue_func_entry(&pwq->wait, ep_poll_callback);
        pwq->whead = whead;
        pwq->base = epi;
        add_wait_queue(whead, &pwq->wait);
        list_add_tail(&pwq->llink, &epi->pwqlist);
        epi->nwait++;
    } else {
        epi->nwait = -1;
    }
}
```

`ep_ptable_queue_proc()` 函数主要工作是把当前 `epitem` 对象添加到 socket 对象的等待队列中，并且设置唤醒函数为 `ep_poll_callback()`，也就是说，当socket状态发生变化时，会触发调用 `ep_poll_callback()` 函数。`ep_poll_callback()` 函数实现如下：

```c
static int ep_poll_callback(wait_queue_t *wait, unsigned mode, int sync, void *key)
{
    ...
    // 把就绪的文件添加到就绪队列中
    list_add_tail(&epi->rdllink, &ep->rdllist);

is_linked:
    // 唤醒调用 epoll_wait() 而被阻塞的进程
    if (waitqueue_active(&ep->wq))
        wake_up_locked(&ep->wq);
    ...
    return 1;
}
```

`ep_poll_callback()` 函数的主要工作是把就绪的文件添加到 `eventepoll` 对象的就绪队列中，然后唤醒调用 `epoll_wait()` 被阻塞的进程。



#### 等待被监听的文件状态发生改变

把被监听的文件句柄添加到epoll后，就可以通过调用 `epoll_wait()` 等待被监听的文件状态发生改变。`epoll_wait()` 调用会阻塞当前进程，当被监听的文件状态发生改变时，`epoll_wait()` 调用便会返回。

`epoll_wait()` 系统调用的原型如下：

```c
long epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

各个参数的意义：

1. `epfd`: 调用 `epoll_create()` 函数创建的epoll句柄。
2. `events`: 用来存放就绪文件列表。
3. `maxevents`: `events` 数组的大小。
4. `timeout`: 设置等待的超时时间。

`epoll_wait()` 函数会调用 `sys_epoll_wait()` 内核函数，而 `sys_epoll_wait()` 函数最终会调用 `ep_poll()` 函数，我们来看看 `ep_poll()` 函数的实现：

```c
static int ep_poll(struct eventpoll *ep, 
    struct epoll_event __user *events, int maxevents, long timeout)
{
    ...
    // 如果就绪文件列表为空
    if (list_empty(&ep->rdllist)) {
        // 把当前进程添加到epoll的等待队列中
        init_waitqueue_entry(&wait, current);
        wait.flags |= WQ_FLAG_EXCLUSIVE;
        __add_wait_queue(&ep->wq, &wait);

        for (;;) {
            set_current_state(TASK_INTERRUPTIBLE); // 把当前进程设置为睡眠状态
            if (!list_empty(&ep->rdllist) || !jtimeout) // 如果有就绪文件或者超时, 退出循环
                break;
            if (signal_pending(current)) { // 接收到信号也要退出
                res = -EINTR;
                break;
            }

            spin_unlock_irqrestore(&ep->lock, flags);
            jtimeout = schedule_timeout(jtimeout); // 让出CPU, 切换到其他进程进行执行
            spin_lock_irqsave(&ep->lock, flags);
        }
        // 有3种情况会执行到这里:
        // 1. 被监听的文件集合中有就绪的文件
        // 2. 设置了超时时间并且超时了
        // 3. 接收到信号
        __remove_wait_queue(&ep->wq, &wait);

        set_current_state(TASK_RUNNING);
    }
    /* 是否有就绪的文件? */
    eavail = !list_empty(&ep->rdllist);

    spin_unlock_irqrestore(&ep->lock, flags);

    if (!res && eavail 
        && !(res = ep_send_events(ep, events, maxevents)) && jtimeout)
        goto retry;

    return res;
}
```

`ep_poll()` 函数主要做以下几件事：

1. 判断被监听的文件集合中是否有就绪的文件，如果有就返回。
2. 如果没有就把当前进程添加到epoll的等待队列中，并且进入睡眠。
3. 进程会一直睡眠直到有以下几种情况发生：
   1. 被监听的文件集合中有就绪的文件
   2. 设置了超时时间并且超时了
   3. 接收到信号
4. 如果有就绪的文件，那么就调用 `ep_send_events()` 函数把就绪文件复制到 `events` 参数中。
5. 返回就绪文件的个数。

最后，我们通过一张图来总结epoll的原理：

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/008eGmZEly1gpc5cdhrr0j310f0u0djf.jpg)

下面通过文字来描述一下这个过程：

1. 通过调用 `epoll_create()` 函数创建并初始化一个 `eventpoll` 对象。
2. 通过调用 `epoll_ctl()` 函数把被监听的文件句柄 (如socket句柄) 封装成 `epitem` 对象并且添加到 `eventpoll` 对象的红黑树中进行管理。
3. 通过调用 `epoll_wait()` 函数等待被监听的文件状态发生改变。
4. 当被监听的文件状态发生改变时（如socket接收到数据），会把文件句柄对应 `epitem` 对象添加到 `eventpoll` 对象的就绪队列 `rdllist` 中。并且把就绪队列的文件列表复制到 `epoll_wait()` 函数的 `events` 参数中。
5. 唤醒调用 `epoll_wait()` 函数被阻塞（睡眠）的进程。