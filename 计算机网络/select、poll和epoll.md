## select、poll和epoll

>select、poll和epoll都是**IO多路复用机制**，即可以监听多个**文件描述符**，一旦某个文件描述的就绪（读或写就绪），能够通知相应进程进行读写操作。但select、poll和epoll本质上都是**同步IO**，因为他们都需要在读写就绪后自己读写数据，也就是说这个读写过程是阻塞的。而**异步IO**则无需自己负责进行读写操作，异步IO的实现会负责把内容从内核空间拷贝到用户空间。



## 实现基础

### 文件抽象与poll操作

#### 文件抽象

在Linux内核里，文件是一个抽象，设备是个文件，网络套接字也是个文件。文件抽象必须支持的能力定义在 `file_operations` 结构体里。

在Linux里，一个打开的文件对应一个文件描述度FD，FD其实是一个整数，内核把进程打开的文件维护在一个数组里，FD对应的是数组的下标。

文件抽象能力的定义：

```c
// 源码位置：include/linux/fs.h
struct file_operations {
    struct module *owner;
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    ssize_t (*read_iter) (struct kiocb *, struct iov_iter *);
    ssize_t (*write_iter) (struct kiocb *, struct iov_iter *);
    int (*iterate) (struct file *, struct dir_context *);
    int (*iterate_shared) (struct file *, struct dir_context *);

    // 对于 select/poll/epoll 最重要的实现基础
    // 非阻塞的轮询文件状态的函数
    unsigned int (*poll) (struct file *, struct poll_table_struct *);

    // 省略其他函数指针
} __randomize_layout;


// 源码位置：include/linux/poll.h
typedef struct poll_table_struct {
    // 文件的 file_operations.poll 实现一定可调用的队列处理函数
    poll_queue_proc _qproc;

    // poll 操作敢兴趣的事件
    unsigned long _key;
} poll_table;

// poll 队列处理函数
typedef void (*poll_queue_proc)(struct file *, wait_queue_head_t *, struct poll_table_struct *);
```

#### 文件poll操作

poll函数的原型：

## select的实现

`__user` 是一个宏定义，表示用户空间的，内核不能直接使用，需要使用函数 `copy_from_user/copy_to_user` 进行处理。

### 源码分析

#### 1. 进程打开的文件

```C
// 源码位置：include/linux/sched.h
struct task_struct {
    // ....省略其他属性

    /* 文件系统信息: */
    struct fs_struct        *fs;

    /* 打开的文件信息: */
    struct files_struct     *files;

    // ....省略其他属性
}
```

**进程维护打开的文件的数据结构**：

```c
// 源码位置：include/linux/fdtable.h
struct files_struct {
  /*
   * read mostly part
   */
    atomic_t count;
    bool resize_in_progress;
    wait_queue_head_t resize_wait;

    struct fdtable __rcu *fdt;
    struct fdtable fdtab;
  /*
   * written part on a separate cache line in SMP
   */
    spinlock_t file_lock ____cacheline_aligned_in_smp;
    unsigned int next_fd;
    unsigned long close_on_exec_init[1];
    unsigned long open_fds_init[1];
    unsigned long full_fds_bits_init[1];
    struct file __rcu * fd_array[NR_OPEN_DEFAULT];
};

struct fdtable {
    // 进程能打开的最大文件数
    unsigned int max_fds;
    struct file __rcu **fd;         /* current fd array */
    unsigned long *close_on_exec;

    // 当前打开的一组文件
    unsigned long *open_fds; 
    unsigned long *full_fds_bits;
    struct rcu_head rcu;
};

static inline bool fd_is_open(unsigned int fd, const struct fdtable *fdt)
{
    return test_bit(fd, fdt->open_fds);
}
```

**小结**：进程打开的文件维护在位图 `fdtable.open_fds`里，对应的比特位为1表示文件已打开，为0是关闭。

select传递事件也借鉴了这种思想，通过位图传递，FD对应的比特位为1表示对事件有兴趣或有事件发生。

#### 2. 基本数据结构

```c
// include/uapi/linux/posix_types.h
#define __FD_SETSIZE    1024
typedef struct {
    unsigned long fds_bits[__FD_SETSIZE / (8 * sizeof(long))];
} __kernel_fd_set;

// 源码位置： include/linux/types.h
typedef __kernel_fd_set     fd_set;

// 源码位置： fs/select.c
// 用于传递 select 的输入事件、输出结果，是 fd_set 的扩展版。
typedef struct {
    unsigned long *in, *out, *ex;
    unsigned long *res_in, *res_out, *res_ex;
} fd_set_bits;
```

**小结**：从上述定义可以看到，fd_set就是一个位图，限制select最多能够 poll 1024个文件，如果需要poll更多文件需要修改码源，重新编译。

`fd_set_bits` 里的6个变量是6个指针，指向不同的位图起始地址，见下文里的注释说明。

#### 3. select主逻辑

`select`调用利用`fd_set`这个位图来传递输入的要监听的事件和输出的结果

```c
// 源码位置： fs/select.c

// select 系统调用原型
SYSCALL_DEFINE5(select, int, n, fd_set __user *, inp, fd_set __user *, outp,
        fd_set __user *, exp, struct timeval __user *, tvp)
{
    struct timespec64 end_time, *to = NULL;
    struct timeval tv;
    int ret;

    if (tvp) {
        if (copy_from_user(&tv, tvp, sizeof(tv)))
            return -EFAULT;

        to = &end_time;
        if (poll_select_set_timeout(to,
                tv.tv_sec + (tv.tv_usec / USEC_PER_SEC),
                (tv.tv_usec % USEC_PER_SEC) * NSEC_PER_USEC))
            return -EINVAL;
    }

    ret = core_sys_select(n, inp, outp, exp, to);
    ret = poll_select_copy_remaining(&end_time, tvp, 1, ret);

    return ret;
}

int core_sys_select(int n, fd_set __user *inp, fd_set __user *outp,
               fd_set __user *exp, struct timespec64 *end_time)
{
    fd_set_bits fds;
    void *bits;
    int ret, max_fds;
    size_t size, alloc_size;
    struct fdtable *fdt;
    /* Allocate small arguments on the stack to save memory and be faster */
    // 在栈上分配的小段参数，用于节省内存和提升速度
    // SELECT_STACK_ALLOC=256
    long stack_fds[SELECT_STACK_ALLOC/sizeof(long)];

    ret = -EINVAL;
    if (n < 0)
        goto out_nofds;

    /* max_fds can increase, so grab it once to avoid race */
    rcu_read_lock();
    fdt = files_fdtable(current->files);
    max_fds = fdt->max_fds;
    rcu_read_unlock();

    // poll 的最大 FD 不能超过 进程打开的最大FD
    if (n > max_fds)
        n = max_fds;


    // 每个文件有3种输入、3种输出，因此每个文件需要6个位图来表示事件
    /*
     * We need 6 bitmaps (in/out/ex for both incoming and outgoing),
     * since we used fdset we need to allocate memory in units of
     * long-words. 
     */
    // n 个文件需要的字节数，也是每份位图的大小
    size = FDS_BYTES(n);
    bits = stack_fds;

    // sizeof(stack_fds) / 6 是把栈上分配的内存块划分为 6 份做位图
    if (size > sizeof(stack_fds) / 6 ) {
        /* Not enough space in on-stack array; must use kmalloc */
        // 栈上分配的空间不够，要使用 kmalloc 
        ret = -ENOMEM;
        if (size > (SIZE_MAX / 6))
            goto out_nofds;

        alloc_size = 6 * size;
        bits = kvmalloc(alloc_size, GFP_KERNEL);
        if (!bits)
            goto out_nofds;
    }

    // 把 fds 里的指针指向不同位图的起始地址
    fds.in      = bits;
    fds.out     = bits +   size;
    fds.ex      = bits + 2*size;
    fds.res_in  = bits + 3*size;
    fds.res_out = bits + 4*size;
    fds.res_ex  = bits + 5*size;

    // 把用户空间的事件拷贝到内核空间
    if ((ret = get_fd_set(n, inp, fds.in)) ||
        (ret = get_fd_set(n, outp, fds.out)) ||
        (ret = get_fd_set(n, exp, fds.ex)))
        goto out;

    // 清零输出结果
    zero_fd_set(n, fds.res_in);
    zero_fd_set(n, fds.res_out);
    zero_fd_set(n, fds.res_ex);

    ret = do_select(n, &fds, end_time);

    if (ret < 0)
        goto out;
    if (!ret) {
        ret = -ERESTARTNOHAND;
        if (signal_pending(current))
            goto out;
        ret = 0;
    }

    // 通过 __copy_to_user 拷贝结果到用户空间
    if (set_fd_set(n, inp, fds.res_in) ||
        set_fd_set(n, outp, fds.res_out) ||
        set_fd_set(n, exp, fds.res_ex))
        ret = -EFAULT;

out:
    if (bits != stack_fds)
        kvfree(bits);
out_nofds:
    return ret;
}


static int do_select(int n, fd_set_bits *fds, struct timespec64 *end_time)
{
    ktime_t expire, *to = NULL;
    // 构建一个等待队列，该队列维护着对所有添加到文件的等待队列的节点的指针
    struct poll_wqueues table;

    // 等待节点的数据原型，主要用于传递参数
    poll_table *wait;
    int retval, i, timed_out = 0;
    u64 slack = 0;
    unsigned int busy_flag = net_busy_loop_on() ? POLL_BUSY_LOOP : 0;
    unsigned long busy_start = 0;

    rcu_read_lock();
    retval = max_select_fd(n, fds);
    rcu_read_unlock();

    if (retval < 0)
        return retval;
    n = retval;

    // 设置 wait._qproc = __pollwait
    poll_initwait(&table);
    wait = &table.pt;
    if (end_time && !end_time->tv_sec && !end_time->tv_nsec) {
        wait->_qproc = NULL;
        timed_out = 1;
    }

    if (end_time && !timed_out)
        slack = select_estimate_accuracy(end_time);

    retval = 0;
    for (;;) {
        unsigned long *rinp, *routp, *rexp, *inp, *outp, *exp;
        bool can_busy_loop = false;

        inp = fds->in; outp = fds->out; exp = fds->ex;
        rinp = fds->res_in; routp = fds->res_out; rexp = fds->res_ex;

        // 分批轮询
        for (i = 0; i < n; ++rinp, ++routp, ++rexp) {
            unsigned long in, out, ex, all_bits, bit = 1, mask, j;
            unsigned long res_in = 0, res_out = 0, res_ex = 0;

            in = *inp++; out = *outp++; ex = *exp++;
            all_bits = in | out | ex;           // 
            if (all_bits == 0) {
                // 没有敢兴趣的事件，跳过 BITS_PER_LONG 个文件
                i += BITS_PER_LONG;
                continue;
            }

            // 批次内逐个轮询
            for (j = 0; j < BITS_PER_LONG; ++j, ++i, bit <<= 1) {   // bit 左移是为了给正确的文件设置事件结果
                struct fd f;
                if (i >= n)
                    break;
                if (!(bit & all_bits))
                    continue;
                f = fdget(i);
                if (f.file) {
                    // 找到了文件
                    const struct file_operations *f_op;
                    f_op = f.file->f_op;
                    mask = DEFAULT_POLLMASK;
                    if (f_op->poll) {
                        wait_key_set(wait, in, out,
                                 bit, busy_flag);

                        // 调用文件的 poll 函数，最终会调用到  __pollwait 函数
                        // __pollwait 
                        mask = (*f_op->poll)(f.file, wait);
                    }
                    fdput(f);

                    // 下面的 if 语句块内，是已经检测到事件发生了，进程不需要进行等待和唤醒
                    // 把 _qproc 设置为 NULL 是为了避免往后续 poll 未就绪的文件时被加入等待队列
                    // 这样可以避免无效的唤醒
                    if ((mask & POLLIN_SET) && (in & bit)) {
                        res_in |= bit;
                        retval++;
                        wait->_qproc = NULL;
                    }
                    if ((mask & POLLOUT_SET) && (out & bit)) {
                        res_out |= bit;
                        retval++;
                        wait->_qproc = NULL;
                    }
                    if ((mask & POLLEX_SET) && (ex & bit)) {
                        res_ex |= bit;
                        retval++;
                        wait->_qproc = NULL;
                    }

                    // 
                    /* got something, stop busy polling */
                    if (retval) {
                        can_busy_loop = false;
                        busy_flag = 0;

                    /*
                     * only remember a returned
                     * POLL_BUSY_LOOP if we asked for it
                     */
                    } else if (busy_flag & mask)
                        can_busy_loop = true;

                }
            }

            // 小批次轮询完，把结果记录下来
            if (res_in)
                *rinp = res_in;
            if (res_out)
                *routp = res_out;
            if (res_ex)
                *rexp = res_ex;

            //  进入睡眠，等待超时或唤醒
            cond_resched();
        }

        // 所有文件都轮询了一遍，要加入文件等待队列的都已经加了，避免下次轮询重复添加
        wait->_qproc = NULL;

        // 有事件、或超时、或有信号要处理
        if (retval || timed_out || signal_pending(current))
            break;
        if (table.error) {
            retval = table.error;
            break;
        }

        /* only if found POLL_BUSY_LOOP sockets && not out of time */
        if (can_busy_loop && !need_resched()) {
            if (!busy_start) {
                busy_start = busy_loop_current_time();
                continue;
            }
            if (!busy_loop_timeout(busy_start))
                continue;
        }
        busy_flag = 0;

        /*
         * If this is the first loop and we have a timeout
         * given, then we convert to ktime_t and set the to
         * pointer to the expiry value.
         */
        if (end_time && !to) {
            expire = timespec64_to_ktime(*end_time);
            to = &expire;
        }

        // 进程状态设置为 TASK_INTERRUPTIBLE，进入睡眠直到超时
        if (!poll_schedule_timeout(&table, TASK_INTERRUPTIBLE,
                       to, slack))
            timed_out = 1;
    }

    // 释放等待节点，重点是把等待节点从文件的等待队列删除掉
    poll_freewait(&table);

    return retval;
}
```

