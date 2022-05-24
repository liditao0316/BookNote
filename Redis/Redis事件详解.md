# Redis-进阶：事件

> Redis采用事件驱动机制来处理大量的网络I/O

## 事件机制

> Redis中的事件驱动库只关注网络IO，以及定时器

该事件库处理下面两类事件：

- **文件事件**(file event)：用于处理Redis服务器和客户端之间的网络IO
- **时间事件**(time event)：Redis服务器中的一些操作（比如serverCron函数）需要在给定的时间点执行，而时间事件就是处理这类定时操作的。

事件驱动库的代码主要是在`src/ae.c`中实现的，其示意图如下所示：

<img src="https://pdai.tech/_images/db/redis/db-redis-event-1.png" alt="img" style="zoom:80%;" />

> aeEventLoop是整个事件驱动的核心，它管理着文件事件表和时间事件列表，不断地循环处理者就绪的文件事件和到期的时间事件



### 文件事件

> Redis基于**Reactor模式**开发了自己的网络事件处理器，也就是文件事件处理器。文件事件处理器使用**IO多路复用技术**，同时监听多个套接字，并为套接字关联不同的事件处理函数。当套接字的可读或者可写事件触发时，就会调用相应的事件处理函数

#### **1.为什么单线程的Redis能那么快？**

Redis的瓶颈主要是在IO而不是CPU，在6.0版本前是单线程模型；其次，Redis是单线程主要是指Redis的网络IO和键值对读写是由一个线程来完成的，这也是Redis对外提供键值存储服务的主要流程。（但Redis的其他功能，比如持久化、移步删除、集群数据同步等，其实是由额外的线程执行的）。

Redis采用了多路复用机制使其在网络IO操作中并发处理大量的客户端请求，实现高吞吐率。



#### 2.Redis事件响应框架ae_event及文件事件处理器

Redis使用的IO复用技术主要有：select、epoll、evport和kqueue等。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220319164015332.png" alt="image-20220319164015332" style="zoom:50%;" />

**文件事件处理器有四个组成部分：套接字、I/O多路复用程序、文件事件分派器以及事件处理器。文件事件处理器是在一个线程中执行的。**

<img src="https://pdai.tech/_images/db/redis/db-redis-event-3.png" alt="img" style="zoom:80%;" />

文件事件是对套接字的抽象（这种实现就要是通过回调的方式实现的），每当一个套接字准备好执行连接应答（accept）、写入、读取、关闭等操作时，就会产生一个文件事件。因为一个服务器通常会连接多个套接字，所以多个文件事件有可能会并发地出现。

I/O多路复用程序负责监听多个套接字，并向文件事件分派器传送哪些产生了事件的套接字。

尽管多个文件事件可能会并发地出现，但I/O多路复用程序总是会将所有产生的套接字都放到同一个队列（也就是aeEventLoop的fired就绪事件表）里面，然后文件事件处理器会以有序、同步、单个套接字的方式处理该队列中的套接字，也就是处理就绪的文件事件

```C
typedef struct aeEventLoop {

    // 目前已注册的最大描述符
    int maxfd;   /* highest file descriptor currently registered */
  
    // 目前已追踪的最大描述符
    int setsize; /* max number of file descriptors tracked */
    
  	// 用于生成时间事件 id
    long long timeEventNextId;

    // 最后一次执行时间事件的时间
    time_t lastTime;     /* Used to detect system clock skew */

    // 已注册的文件事件
    aeFileEvent *events; /* Registered events */

    // 已就绪的文件事件
    aeFiredEvent *fired; /* Fired events */

    // 时间事件
    aeTimeEvent *timeEventHead;

    // 事件处理器的开关
    int stop;

    // 多路复用库的私有数据
    void *apidata; /* This is used for polling API specific data */

    // 在处理事件前要执行的函数
    aeBeforeSleepProc *beforesleep;

} aeEventLoop;
```

<img src="https://pdai.tech/_images/db/redis/db-redis-event-4.png" alt="img" style="zoom:80%;" />

所以，一次Redis客户端与服务器进行连接并且发送命令的过程如上所示。



#### 3.Redis IO多路复用模型

在Redis只运行单线程的情况下，**该机制允许内核中，同时存在多个监听套接字和已连接套接字。**内核会一直监听这些套接字上的连接请求或数据请求。一旦有请求到达，就会交给Redis线程处理，这就实现了一个Redis线程处理多个IO流的效果。（**也就是说，处理IO还是Redis线程处理，只是将这些处理放在一个队列中，依次有序地执行**）

理解多路复用模型之前，我们先来看看客户端请求进来，是如何生成套接字的。

套接字执行流程中，函数调用情况如下所示

![image-20220320153043095](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220320153043095.png)

```c
/* Accept a connection and also make sure the socket is non-blocking, and CLOEXEC.
 * returns the new socket FD, or -1 on error. */
int anetTcpAccept(char *err, int serversock, char *ip, size_t ip_len, int *port) {
    int fd;
    struct sockaddr_storage sa;
    socklen_t salen = sizeof(sa);
    if ((fd = anetGenericAccept(err,serversock,(struct sockaddr*)&sa,&salen)) == ANET_ERR)
        return ANET_ERR;

    if (sa.ss_family == AF_INET) {
        struct sockaddr_in *s = (struct sockaddr_in *)&sa;
        if (ip) inet_ntop(AF_INET,(void*)&(s->sin_addr),ip,ip_len);
        if (port) *port = ntohs(s->sin_port);
    } else {
        struct sockaddr_in6 *s = (struct sockaddr_in6 *)&sa;
        if (ip) inet_ntop(AF_INET6,(void*)&(s->sin6_addr),ip,ip_len);
        if (port) *port = ntohs(s->sin6_port);
    }
    return fd;
}
```

```c
static int anetGenericAccept(char *err, int s, struct sockaddr *sa, socklen_t *len) {
    int fd;
    ...
    fd = accept(s,sa,len);
		...
    return fd;
}
```

>fd对应于文件事件中的fd，同时也对应于socket描述符fd

下图就是基于多路复用的Redis IO模型。图中的多个FD就是刚才所说的多个套接字。Redis网络框架调用epoll机制（或者kqueue、poll、select机制），让内核监听这些套接字。此时，Redis线程不会阻塞在一个特定的监听或者已连接套接字上，也就是说，不会阻塞在某一个特定的监听或已连接套接字上，也是说，不会阻塞在某一个特定的客户端请求处理上，正因为此，Redis可以同时和多个客户端连接并处理请求，从而提高并发性。

![img](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/db-redis-event-0.jpg)

基于多路复用的Redis高性能IO模型为了在请求到达时能通知到Redis线程，select/epoll提供基于事件的回调机制，即针对不同事件的发生，调用相应的处理函数。

**回调机制：**

其实，**select/epoll一旦检测到FD上有请求到达时，就会触发相应的事件**。这些事件会被放进一个事件队列，Redis单线程对该事件队列不断进行处理。这样一来，Redis无需一直轮询是否有请求实际发生，这就可以避免造成CPU资源浪费。同时，Redis在对事件队列进行处理，所以能够及时响应客户端请求，提高Redis的响应性能。

> 检测到FD上有请求到达时，就会触发相应的事件，这个事件是服务器一开始启动的时候创建的文件事件。
>
> ```c
> int main(int argc, char **argv) {
>   ...
> 	initServer();
> 	...
> }
> ```
>
> ```c
> void initServer(void) {
>   	....
> 		/* Create an event handler for accepting new connections in TCP and Unix
>      * domain sockets. */
>     if (createSocketAcceptHandler(&server.ipfd, acceptTcpHandler) != C_OK) {
>         serverPanic("Unrecoverable error creating TCP socket accept handler.");
>     }
>     if (createSocketAcceptHandler(&server.tlsfd, acceptTLSHandler) != C_OK) {
>         serverPanic("Unrecoverable error creating TLS socket accept handler.");
>     }
>     if (server.sofd > 0 && aeCreateFileEvent(server.el,server.sofd,AE_READABLE,
>         acceptUnixHandler,NULL) == AE_ERR) serverPanic("Unrecoverable error creating server.sofd file event.");
> 
> 
>     /* Register a readable event for the pipe used to awake the event loop
>      * from module threads. */
>     if (aeCreateFileEvent(server.el, server.module_pipe[0], AE_READABLE,
>         modulePipeReadable,NULL) == AE_ERR) {
>             serverPanic(
>                 "Error registering the readable event for the module pipe.");
>     }
>   	....
> }
> ```

> 每一个套接字FD只会创建一个对应的文件事件，也就是第一次连接服务器时创建一个文件事件，与服务器断开连接时会删除该文件事件。
>
> 每一个套接字FD创建文件事件，同时会将该文件事件添加到IO多路复用程序中监听
>
> 



#### 4. aeApiPoll--IO多路复用程序

<img src="https://pdai.tech/_images/db/redis/db-redis-event-6.png" alt="img" style="zoom:60%;" />

```c
static int aeApiPoll(aeEventLoop *eventLoop, struct timeval *tvp) 
{
    aeApiState *state = eventLoop->apidata;
    int retval, numevents = 0;
    // 调用epoll_wait函数，等待时间为最近达到时间事件的时间计算而来。
    retval = epoll_wait(state->epfd,state->events,eventLoop->setsize,
            tvp ? (tvp->tv_sec*1000 + tvp->tv_usec/1000) : -1);
    // 有至少一个事件就绪？
    if (retval > 0) 
    {
        int j;
        /*为已就绪事件设置相应的模式，并加入到 eventLoop 的 fired 数组中*/
        numevents = retval;
        for (j = 0; j < numevents; j++) 
    		{
            int mask = 0;
            struct epoll_event *e = state->events+j;
            if (e->events & EPOLLIN)
        mask |= AE_READABLE;
            if (e->events & EPOLLOUT)
        mask |= AE_WRITABLE;
            if (e->events & EPOLLERR) 
        mask |= AE_WRITABLE;
            if (e->events & EPOLLHUP)
        mask |= AE_WRITABLE;
            /* 设置就绪事件表元素 */
            eventLoop->fired[j].fd = e->data.fd;
            eventLoop->fired[j].mask = mask;
        }
    }
    
    // 返回已就绪事件个数
    return numevents;
}
```



### 时间事件

#### 1. 时间事件的分类

- 定时事件：让一段程序在指定的时间之后执行一次
- 周期性事件：让一段程序每隔指定时间就执行一次

#### 2. 时间事件的执行

​	ae.c/processTimeEvents函数是时间事件的执行器，这个函数会遍历所有已达到的时间事件，并调用这些事件的处理器。已达到事件指的是，时间事件的when属性记录的UNIX时间戳等于或小于当前事件的UNIX时间戳。

> 因此，不是说时间事件的时间戳到了就执行时间事件，而是等到线程调用processTimeEvents函数时才去执行时间事件



## Redis命令执行过程分析

> 一条命令执行完成并且返回数据一共涉及三部分：
>
> - **建立连接阶段**：响应socket的建立，并且创建client对象
> - **处理阶段**：从socket读取数据到输入缓冲区，然后解析并获得命令，执行命令并将返回值存储到输出缓冲区中
> - **数据返回阶段**：将返回值从输出缓冲区写到socket中，返回给客户端，最后关闭client

![image-20220320165514955](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220320165514955.png)

这三个阶段之间通过事件机制串联，在Redis启动阶段首先要注册socket连接建立事件处理器：

- 当客户端发来建立socket连接的请求时，用于建立连接的**文件事件**会被触发，然后其注册的方法将会被执行，然后注册socket读取事件处理器
- 当客户端发来命令时，socket读取事件处理器方法会被执行，对应处理阶段的相关逻辑将会被执行，然后注册socket写事件处理器
- 当写事件处理器被执行时，就会将返回值写回到socket中。

<img src="https://pdai.tech/_images/db/redis/db-redis-event-4.png" alt="img" style="zoom:80%;" />

### 建立连接阶段

#### 建立连接和Client

当客户端向Redis建立socket时，aeEventLoop会调用acceptTcpHandler处理函数，服务器会为每一个连接创建一个Client对象，并创建相应文件事件来监听socket的可读事件，并指定事件处理函数。

`acceptTcpHandler`函数会首先调`anetTcpAccept`方法，它底层会调用socket的accept方法，也就是接受客户端建立连接的请求，然后调用`accpet CommonHandler`方法，继续后续的逻辑处理。

```c
// 当客户端建立链接时进行的eventloop处理函数  networking.c
void acceptTcpHandler(aeEventLoop *el, int fd, void *privdata, int mask) {
    ....
    // 层层调用，最后在anet.c 中 anetGenericAccept 方法中调用 socket 的 accept 方法
    cfd = anetTcpAccept(server.neterr, fd, cip, sizeof(cip), &cport);
    if (cfd == ANET_ERR) {
        if (errno != EWOULDBLOCK)
            serverLog(LL_WARNING,
                "Accepting client connection: %s", server.neterr);
        return;
    }
    serverLog(LL_VERBOSE,"Accepted %s:%d", cip, cport);
    /**
     * 进行socket 建立连接后的处理
     */
    acceptCommonHandler(cfd,0,cip);
}
```

`acceptCommonHandler` 则首先调用 createClient 创建 client，接着判断当前 client 的数量是否超出了配置的 maxclients，如果超过，则给客户端发送错误信息，并且释放 client。

```java
static void acceptCommonHandler(int fd, int flags, char *ip) { //networking.c
    client *c;
    // 创建redisClient
    c = createClient(fd)
    // 当 maxClient 属性被设置，并且client数量已经超出时，给client发送error，然后释放连接
    if (listLength(server.clients) > server.maxclients) {
        char *err = "-ERR max number of clients reached\r\n";
        if (write(c->fd,err,strlen(err)) == -1) {
        }
        server.stat_rejected_conn++;
        freeClient(c);
        return;
    }
    .... // 处理为设置密码时默认保护状态的客户端连接
    // 统计连接数
    server.stat_numconnections++;
    c->flags |= flags;
}
```

`createClient`方法用于创建client，它代表连接到Redis客户端，每个客户端都有各自的输入缓冲区和输出缓冲区，输入缓冲区存储客户端通过socket发送过来的数据，输出缓冲区则存储着Redis对客户端的响应数据。

![image-20220320173409863](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220320173409863.png)

`createClient`方法除了创建client结构体并设置其属性值外，还会**对socket进行配置并注册读事件处理器**。(创建socket对应的文件事件)

```c
client *createClient(int fd) {
    client *c = zmalloc(sizeof(client));
    // fd 为 -1，表示其他特殊情况创建的client，redis在进行比如lua脚本执行之类的情况下也会创建client
    if (fd != -1) {
        // 配置socket为非阻塞、NO_DELAY不开启Nagle算法和SO_KEEPALIVE
        anetNonBlock(NULL,fd);
        anetEnableTcpNoDelay(NULL,fd);
        if (server.tcpkeepalive)
            anetKeepAlive(NULL,fd,server.tcpkeepalive);
        /**
         * 向 eventLoop 中注册了 readQueryFromClient。
         * readQueryFromClient 的作用就是从client中读取客户端的查询缓冲区内容。
         * 绑定读事件到事件 loop （开始接收命令请求）
         */
        if (aeCreateFileEvent(server.el,fd,AE_READABLE,
            readQueryFromClient, c) == AE_ERR)
        {
            close(fd);
            zfree(c);
            return NULL;
        }
    }
    // 默认选择数据库
    selectDb(c,0);
    uint64_t client_id;
    atomicGetIncr(server.next_client_id,client_id,1);
    c->id = client_id;
    c->fd = fd;
    .... // 设置client的属性,输入缓冲区 querybuf 和输出缓冲区 buf
    return c;
}
```



#### 读取socket数据到输入缓冲区

`readQueryFromClient`方法会调用read方法**从socket中读取数据到输入缓冲区**，然后判断大小是否大于系统设置的clientmaxquerybuf_len，如果大于，则向Redis发送错误信息，并关闭client

将数据读取到输入缓冲区后，readQueryFromClient方法会根据client的类型来做不同的处理：

- **普通客户端**：则直接调用processInputBuffer来处理；
- **主从客户端**：还需要将命令同步到自己的从服务器中，也就是说，Redis实例将主实例传来的命令执行后，继续将命令同步给自己的从实例。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220320183601887.png" alt="image-20220320183601887" style="zoom:50%;" />

```c
// 处理从client中读取客户端的输入缓冲区内容。
void readQueryFromClient(aeEventLoop *el, int fd, void *privdata, int mask) {
    client *c = (client*) privdata;
    ....
    if (c->querybuf_peak < qblen) c->querybuf_peak = qblen;
    c->querybuf = sdsMakeRoomFor(c->querybuf, readlen);
    // 从 fd 对应的socket中读取到 client 中的 querybuf 输入缓冲区
    nread = read(fd, c->querybuf+qblen, readlen);
    if (nread == -1) {
        .... // 出错释放 client
    } else if (nread == 0) {
        // 客户端主动关闭 connection
        serverLog(LL_VERBOSE, "Client closed connection");
        freeClient(c);
        return;
    } else if (c->flags & CLIENT_MASTER) { 
        /*
         * 当这个client代表主从的master节点时，将query buffer和 pending_querybuf结合
         * 用于主从复制中的命令传播？？？？
         */
        c->pending_querybuf = sdscatlen(c->pending_querybuf,
                                        c->querybuf+qblen,nread);
    }
    // 增加已经读取的字节数
    sdsIncrLen(c->querybuf,nread);
    c->lastinteraction = server.unixtime;
    if (c->flags & CLIENT_MASTER) c->read_reploff += nread;
    server.stat_net_input_bytes += nread;
    // 如果大于系统配置的最大客户端缓存区大小，也就是配置文件中的client-query-buffer-limit
    if (sdslen(c->querybuf) > server.client_max_querybuf_len) {
        sds ci = catClientInfoString(sdsempty(),c), bytes = sdsempty();
        // 返回错误信息，并且关闭client
        bytes = sdscatrepr(bytes,c->querybuf,64);
        serverLog(LL_WARNING,"Closing client that reached max query buffer length: %s (qbuf initial bytes: %s)", ci, bytes);
        sdsfree(ci);
        sdsfree(bytes);
        freeClient(c);
        return;
    }


    if (!(c->flags & CLIENT_MASTER)) {
        // processInputBuffer 处理输入缓冲区
        processInputBuffer(c);
    } else {
        // 如果client是master的连接
        size_t prev_offset = c->reploff;
        processInputBuffer(c);
        // 判断是否同步偏移量发生变化，则通知到后续的slave
        size_t applied = c->reploff - prev_offset;

        if (applied) {
            replicationFeedSlavesFromMasterStream(server.slaves,
                    c->pending_querybuf, applied);
            sdsrange(c->pending_querybuf,applied,-1);
        }
    }
}
```



#### 解析获取命令

`processInputBuffer`主要是将输入缓冲区中的数据解析对应的命令，根据命令类型是 PROTOREQMULTIBULK 还是 PROTOREQINLINE，来分别调用 processInlineBuffer 和 processMultibulkBuffer 方法来解析命令。

调用`processCommand`方法执行命令。执行成功后，如果是主从客户端，还需要更新同步偏移量reploff属性，然后重置client，让client可以接收一条命令。

```c
void processInputBuffer(client *c) { // networking.c
    server.current_client = c;
    /* 当缓冲区中还有数据时就一直处理 */
    while(sdslen(c->querybuf)) {
        .... // 处理 client 的各种状态
        /* 判断命令请求类型 telnet发送的命令和redis-cli发送的命令请求格式不同 */
        if (!c->reqtype) {
            if (c->querybuf[0] == '*') {
                c->reqtype = PROTO_REQ_MULTIBULK;
            } else {
                c->reqtype = PROTO_REQ_INLINE;
            }
        }
        /**
         * 从缓冲区解析命令
         */
        if (c->reqtype == PROTO_REQ_INLINE) {
            if (processInlineBuffer(c) != C_OK) break;
        } else if (c->reqtype == PROTO_REQ_MULTIBULK) {
            if (processMultibulkBuffer(c) != C_OK) break;
        } else {
            serverPanic("Unknown request type");
        }

        /* 参数个数为0时重置client，可以接受下一个命令 */
        if (c->argc == 0) {
            resetClient(c);
        } else {
            // 执行命令
            if (processCommand(c) == C_OK) {
                if (c->flags & CLIENT_MASTER && !(c->flags & CLIENT_MULTI)) {
                    // 如果是master的client发来的命令，则 更新 reploff
                    c->reploff = c->read_reploff - sdslen(c->querybuf);
                }

                // 如果不是阻塞状态，则重置client，可以接受下一个命令
                if (!(c->flags & CLIENT_BLOCKED) || c->btype != BLOCKED_MODULE)
                    resetClient(c);
            }
        }
    }
    server.current_client = NULL;
}
```



#### 执行命令

`processCommand`方法会处理很多逻辑，不过大致可以分为三个部分：首先是调用lookupCommand方法获取对应的redisCommand；接着是检测当前Redis是否可以执行该命令；最后调用call方法真正执行命令。

```c
int processCommand(client *c) {
    // 1 处理 quit 命令
    if (!strcasecmp(c->argv[0]->ptr,"quit")) {
        addReply(c,shared.ok);
        c->flags |= CLIENT_CLOSE_AFTER_REPLY;
        return C_ERR;
    }

    /**
     * 根据 argv[0] 查找对应的 command
     * 2 命令字典查找指定命令；所有的命令都存储在命令字典中 struct redisCommand redisCommandTable[]={}
     */
    c->cmd = c->lastcmd = lookupCommand(c->argv[0]->ptr);
    if (!c->cmd) {
        // 处理未知命令
    } else if ((c->cmd->arity > 0 && c->cmd->arity != c->argc) ||
               (c->argc < -c->cmd->arity)) {
        // 处理参数错误
    }
    // 3 检查用户验证
    if (server.requirepass && !c->authenticated && c->cmd->proc != authCommand)
    {
        flagTransaction(c);
        addReply(c,shared.noautherr);
        return C_OK;
    }

    /**
     * 4 如果是集群模式，处理集群重定向。当命令发送者是master或者 命令没有任何key的参数时可以不重定向
     */
    if (server.cluster_enabled &&
        !(c->flags & CLIENT_MASTER) &&
        !(c->flags & CLIENT_LUA &&
          server.lua_caller->flags & CLIENT_MASTER) &&
        !(c->cmd->getkeys_proc == NULL && c->cmd->firstkey == 0 &&
          c->cmd->proc != execCommand))
    {
        int hashslot;
        int error_code;
        // 查询可以执行的node信息
        clusterNode *n = getNodeByQuery(c,c->cmd,c->argv,c->argc,
                                        &hashslot,&error_code);
        if (n == NULL || n != server.cluster->myself) {
            if (c->cmd->proc == execCommand) {
                discardTransaction(c);
            } else {
                flagTransaction(c);
            }
            clusterRedirectClient(c,n,hashslot,error_code);
            return C_OK;
        }
    }

    // 5 处理maxmemory请求，先尝试回收一下，如果不行，则返回异常
    if (server.maxmemory) {
        int retval = freeMemoryIfNeeded();
        ....
    }

    /**
     * 6 当此服务器是master时：aof持久化失败时，或上一次bgsave执行错误，
     * 且配置bgsave参数和stop_writes_on_bgsave_err；禁止执行写命令
     */
    if (((server.stop_writes_on_bgsave_err &&
          server.saveparamslen > 0 &&
          server.lastbgsave_status == C_ERR) ||
          server.aof_last_write_status == C_ERR) &&
        server.masterhost == NULL &&
        (c->cmd->flags & CMD_WRITE ||
         c->cmd->proc == pingCommand)) { .... }

    /**
     * 7 当此服务器时master时：如果配置了repl_min_slaves_to_write，
     * 当slave数目小于时，禁止执行写命令
     */
    if (server.masterhost == NULL &&
        server.repl_min_slaves_to_write &&
        server.repl_min_slaves_max_lag &&
        c->cmd->flags & CMD_WRITE &&
        server.repl_good_slaves_count < server.repl_min_slaves_to_write) { .... }

    /**
     * 8 当时只读slave时，除了master的不接受其他写命令
     */
    if (server.masterhost && server.repl_slave_ro &&
        !(c->flags & CLIENT_MASTER) &&
        c->cmd->flags & CMD_WRITE) { .... }

    /**
     * 9 当客户端正在订阅频道时，只会执行以下命令
     */
    if (c->flags & CLIENT_PUBSUB &&
        c->cmd->proc != pingCommand &&
        c->cmd->proc != subscribeCommand &&
        c->cmd->proc != unsubscribeCommand &&
        c->cmd->proc != psubscribeCommand &&
        c->cmd->proc != punsubscribeCommand) { .... }
    /**
     * 10 服务器为slave，但没有正确连接master时，只会执行带有CMD_STALE标志的命令，如info等
     */
    if (server.masterhost && server.repl_state != REPL_STATE_CONNECTED &&
        server.repl_serve_stale_data == 0 &&
        !(c->cmd->flags & CMD_STALE)) {...}
    /**
     * 11 正在加载数据库时，只会执行带有CMD_LOADING标志的命令，其余都会被拒绝
     */
    if (server.loading && !(c->cmd->flags & CMD_LOADING)) { .... }
    /**
     * 12 当服务器因为执行lua脚本阻塞时，只会执行以下几个命令，其余都会拒绝
     */
    if (server.lua_timedout &&
          c->cmd->proc != authCommand &&
          c->cmd->proc != replconfCommand &&
        !(c->cmd->proc == shutdownCommand &&
          c->argc == 2 &&
          tolower(((char*)c->argv[1]->ptr)[0]) == 'n') &&
        !(c->cmd->proc == scriptCommand &&
          c->argc == 2 &&
          tolower(((char*)c->argv[1]->ptr)[0]) == 'k')) {....}

    /**
     * 13 开始执行命令
     */
    if (c->flags & CLIENT_MULTI &&
        c->cmd->proc != execCommand && c->cmd->proc != discardCommand &&
        c->cmd->proc != multiCommand && c->cmd->proc != watchCommand)
    {
        /**
         * 开启了事务，命令只会入队列
         */
        queueMultiCommand(c);
        addReply(c,shared.queued);
    } else {
        /**
         * 直接执行命令
         */
        call(c,CMD_CALL_FULL);
        c->woff = server.master_repl_offset;
        if (listLength(server.ready_keys))
            handleClientsBlockedOnLists();
    }
    return C_OK;
}


struct redisCommand redisCommandTable[] = {
    {"get",getCommand,2,"rF",0,NULL,1,1,1,0,0},
    {"set",setCommand,-3,"wm",0,NULL,1,1,1,0,0},
    {"hmset",hsetCommand,-4,"wmF",0,NULL,1,1,1,0,0},
    .... // 所有的 redis 命令都有
}
```

call方法是Redis中执行命令的通用方法，它会处理通用的命令的前置和后续操作。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220320185120385.png" alt="image-20220320185120385" style="zoom:50%;" />

- 如果有监视器 monitor，则需要将命令发送给监视器。
- 调用 redisCommand 的proc 方法，执行对应具体的命令逻辑。
- 如果开启了 CMDCALLSLOWLOG，则需要记录慢查询日志
- 如果开启了 CMDCALLSTATS，则需要记录一些统计信息
- 如果开启了 CMDCALLPROPAGATE，则当 dirty大于0时，需要调用 propagate 方法来进行命令传播。

<img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220320185751273.png" alt="image-20220320185751273" style="zoom:50%;" />

命令传播就是将写命令写入到repl-backlog-buffer缓冲中，并发送给各个从服务器中

```c
// 执行client中持有的 redisCommand 命令
void call(client *c, int flags) {
    /**
     * dirty记录数据库修改次数；start记录命令开始执行时间us；duration记录命令执行花费时间
     */
    long long dirty, start, duration;
    int client_old_flags = c->flags;

    /**
     * 有监视器的话，需要将不是从AOF获取的命令会发送给监视器。当然，这里会消耗时间
     */
    if (listLength(server.monitors) &&
        !server.loading &&
        !(c->cmd->flags & (CMD_SKIP_MONITOR|CMD_ADMIN)))
    {
        replicationFeedMonitors(c,server.monitors,c->db->id,c->argv,c->argc);
    }
    ....
    /* Call the command. */
    dirty = server.dirty;
    start = ustime();
    // 处理命令，调用命令处理函数
    c->cmd->proc(c);
    duration = ustime()-start;
    dirty = server.dirty-dirty;
    if (dirty < 0) dirty = 0;

    .... // Lua 脚本的一些特殊处理

    /**
     * CMD_CALL_SLOWLOG 表示要记录慢查询日志
     */
    if (flags & CMD_CALL_SLOWLOG && c->cmd->proc != execCommand) {
        char *latency_event = (c->cmd->flags & CMD_FAST) ?
                              "fast-command" : "command";
        latencyAddSampleIfNeeded(latency_event,duration/1000);
        slowlogPushEntryIfNeeded(c,c->argv,c->argc,duration);
    }
    /**
     * CMD_CALL_STATS 表示要统计
     */
    if (flags & CMD_CALL_STATS) {
        c->lastcmd->microseconds += duration;
        c->lastcmd->calls++;
    }
    /**
     * CMD_CALL_PROPAGATE表示要进行广播命令
     */
    if (flags & CMD_CALL_PROPAGATE &&
        (c->flags & CLIENT_PREVENT_PROP) != CLIENT_PREVENT_PROP)
    {
        int propagate_flags = PROPAGATE_NONE;
        /**
         * dirty大于0时，需要广播命令给slave和aof
         */
        if (dirty) propagate_flags |= (PROPAGATE_AOF|PROPAGATE_REPL);
        .... 
        /**
         * 广播命令，写如aof，发送命令到slave
         * 也就是传说中的传播命令
         */
        if (propagate_flags != PROPAGATE_NONE && !(c->cmd->flags & CMD_MODULE))
            propagate(c->cmd,c->db->id,c->argv,c->argc,propagate_flags);
    }
    ....
}
```



#### 将命令结果写入输出缓存区

在所有的redisComman执行的最后，一般都会调用addReply方法进行结果返回。

`addReply`方法做了两件事情：

- `prepareClientToWrite`判断是否需要返回数据，并且将当前client添加到等待写返回数据队列中。
- 调用`_addReplyToBuffer`或者` _addReplyObjectToList`方法将返回值写入到输出缓冲区中，等待写入socket.

```c
void addReply(client *c, robj *obj) {
    if (prepareClientToWrite(c) != C_OK) return;
    if (sdsEncodedObject(obj)) {
        // 需要将响应内容添加到output buffer中。总体思路是，先尝试向固定buffer添加，添加失败的话，在尝试添加到响应链表
        if (_addReplyToBuffer(c,obj->ptr,sdslen(obj->ptr)) != C_OK)
            _addReplyObjectToList(c,obj);
    } else if (obj->encoding == OBJ_ENCODING_INT) {
        .... // 特殊情况的优化
    } else {
        serverPanic("Wrong obj->encoding in addReply()");
    }
}
```

`prepareClientToWrite`首先判断当前client是否需要返回数据；接着如果这个client还未加入到clients_pending_write队列中，则将其加入Redis的等待写入返回值客户端队列中。

```java
int prepareClientToWrite(client *c) {
    // 如果是 lua client 则直接OK
    if (c->flags & (CLIENT_LUA|CLIENT_MODULE)) return C_OK;
    // 客户端发来过 REPLY OFF 或者 SKIP 命令，不需要发送返回值
    if (c->flags & (CLIENT_REPLY_OFF|CLIENT_REPLY_SKIP)) return C_ERR;

    // master 作为client 向 slave 发送命令，不需要接收返回值
    if ((c->flags & CLIENT_MASTER) &&
        !(c->flags & CLIENT_MASTER_FORCE_REPLY)) return C_ERR;
    // AOF loading 时的假client 不需要返回值
    if (c->fd <= 0) return C_ERR; 

    // 将client加入到等待写入返回值队列中，下次事件周期会进行返回值写入。
    if (!clientHasPendingReplies(c) &&
        !(c->flags & CLIENT_PENDING_WRITE) &&
        (c->replstate == REPL_STATE_NONE ||
         (c->replstate == SLAVE_STATE_ONLINE && !c->repl_put_online_on_ack)))
    {
        // 设置标志位并且将client加入到 clients_pending_write 队列中
        c->flags |= CLIENT_PENDING_WRITE;
        listAddNodeHead(server.clients_pending_write,c);
    }
    // 表示已经在排队，进行返回数据
    return C_OK;
}
```

Redis将输出缓冲区分为两部分：一个是固定大小的buffer和一个响应内容数据的链表。固定buffer和响应链表，整体上构成了一个队列。这样组织的好处是，既可以节省空间，预先不需要分配大量的内存空间，并且可以抛弃频繁分配和回收空间。

- 在链表为空并且buffer有足够空间时，则将响应添加到buffer中

- 如果buffer满了则创建一个结点追加到链表上

  <img src="https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220320200007985.png" alt="image-20220320200007985" style="zoom:50%;" />



#### 将命令返回值从输出缓冲区写入socket

Redis在两次事件循环之间都会调用`beforesleep`方法处理一些事情，对于clients_pending_write队列的处理也在其中

```c
int aeProcessEvents(aeEventLoop *eventLoop, int flags)
{
    ...
		//如果需要在在事件处理前执行的函数，那么执行它
    if (eventLoop->beforesleep != NULL && flags & AE_CALL_BEFORE_SLEEP)
        eventLoop->beforesleep(eventLoop);

  /* Call the multiplexing API, will return only on timeout or when
         * some event fires. */
  		numevents = aeApiPoll(eventLoop, tvp);
}

```

`beforeSleep` 函数会调用 handleClientsWithPendingWrites 函数来处理 clientspendingwrite 列表。

![image-20220320204057769](https://ldt-typora.oss-cn-shenzhen.aliyuncs.com/img/image-20220320204057769.png)

`handleClientsWithPendingWrites`方法会遍历clientspendingwrite列表，对于每个client都会先调用`writeToClient`方法尝试将返回数据从输出缓冲区写入到socket中，**如果还未写完，则只能调用aeCreateFileEvent方法来注册一个写数据事件处理器`sendReplyToCilent`**，等待Redis事件机制的再次调用。

这样的好处是对于返回数据较少的客户端，不需要麻烦的注册写事件处理器，等待事件触发再写到socket中，而是在下一次事件循环周期就直接将数据写到socket中，加快了数据返回的响应速度。

但是从这里会发现，如果clientspendingwrite队列过长，则处理事件也会很久，阻塞正常的事件响应处理，导致Redis后续命令延时增加。

```c
// 直接将返回值写到client的输出缓冲区中，不需要进行系统调用，也不需要注册写事件处理器
int handleClientsWithPendingWrites(void) {
    listIter li;
    listNode *ln;
    // 获取系统延迟写队列的长度
    int processed = listLength(server.clients_pending_write);

    listRewind(server.clients_pending_write,&li);
    // 依次处理
    while((ln = listNext(&li))) {
        client *c = listNodeValue(ln);
        c->flags &= ~CLIENT_PENDING_WRITE;
        listDelNode(server.clients_pending_write,ln);

        // 将缓冲值写入client的socket中，如果写完，则跳过之后的操作。
        if (writeToClient(c->fd,c,0) == C_ERR) continue;

        // 还有数据未写入，只能注册写事件处理器了
        if (clientHasPendingReplies(c)) {
            int ae_flags = AE_WRITABLE;
            if (server.aof_state == AOF_ON &&
                server.aof_fsync == AOF_FSYNC_ALWAYS)
            {
                ae_flags |= AE_BARRIER;
            }
            // 注册写事件处理器 sendReplyToClient，等待执行
            if (aeCreateFileEvent(server.el, c->fd, ae_flags,
                sendReplyToClient, c) == AE_ERR)
            {
                    freeClientAsync(c);
            }
        }
    }
    return processed;
}
```

sendReplyToClient 方法其实也会调用 writeToClient 方法，该方法就是将输出缓冲区中的 buf 和 reply 列表中的数据都尽可能多的写入到对应的 socket中。

```c
// 将输出缓冲区中的数据写入socket，如果还有数据未处理则返回C_OK
int writeToClient(int fd, client *c, int handler_installed) {
    ssize_t nwritten = 0, totwritten = 0;
    size_t objlen;
    sds o;
    // 仍然有数据未写入
    while(clientHasPendingReplies(c)) {
        // 如果缓冲区有数据
        if (c->bufpos > 0) {
            // 写入到 fd 代表的 socket 中
            nwritten = write(fd,c->buf+c->sentlen,c->bufpos-c->sentlen);
            if (nwritten <= 0) break;
            c->sentlen += nwritten;
            // 统计本次一共输出了多少子节
            totwritten += nwritten;

            // buffer中的数据已经发送，则重置标志位，让响应的后续数据写入buffer
            if ((int)c->sentlen == c->bufpos) {
                c->bufpos = 0;
                c->sentlen = 0;
            }
        } else {
            // 缓冲区没有数据，从reply队列中拿
            o = listNodeValue(listFirst(c->reply));
            objlen = sdslen(o);

            if (objlen == 0) {
                listDelNode(c->reply,listFirst(c->reply));
                continue;
            }
            // 将队列中的数据写入 socket
            nwritten = write(fd, o + c->sentlen, objlen - c->sentlen);
            if (nwritten <= 0) break;
            c->sentlen += nwritten;
            totwritten += nwritten;
            // 如果写入成功，则删除队列
            if (c->sentlen == objlen) {
                listDelNode(c->reply,listFirst(c->reply));
                c->sentlen = 0;
                c->reply_bytes -= objlen;
                if (listLength(c->reply) == 0)
                    serverAssert(c->reply_bytes == 0);
            }
        }
        // 如果输出的字节数量已经超过NET_MAX_WRITES_PER_EVENT限制，break
        if (totwritten > NET_MAX_WRITES_PER_EVENT &&
            (server.maxmemory == 0 ||
             zmalloc_used_memory() < server.maxmemory) &&
            !(c->flags & CLIENT_SLAVE)) break;
    }
    server.stat_net_output_bytes += totwritten;
    if (nwritten == -1) {
        if (errno == EAGAIN) {
            nwritten = 0;
        } else {
            serverLog(LL_VERBOSE,
                "Error writing to client: %s", strerror(errno));
            freeClient(c);
            return C_ERR;
        }
    }
    if (!clientHasPendingReplies(c)) {
        c->sentlen = 0;
        //如果内容已经全部输出，删除事件处理器
        if (handler_installed) aeDeleteFileEvent(server.el,c->fd,AE_WRITABLE);
        // 数据全部返回，则关闭client和连接
        if (c->flags & CLIENT_CLOSE_AFTER_REPLY) {
            freeClient(c);
            return C_ERR;
        }
    }
    return C_OK;
}
```



## 总结

### IO多路复用

> 每一个客户端请求进来，Redis都会给客户端创建一个socket，每个socket都有一个唯一的编号，同时创建一个Client，并且会为client绑定一个读文件事件，用于将请求信息从socket写入Client的输入缓冲期。操作系统内核监听socket，一旦socket有请求进来，IO多路复用程序会通知主线程，让主线程执行相应的文件事件。而这个通知就是通过socket的编号来实现的，也就是告诉主线程，与这个套接字绑定的读事件可以执行了。
