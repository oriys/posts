---
title: "八股文::操作系统"
date: 2021-08-21T03:28:49+08:00
draft: false
---

## 操作系统

### 进程和线程之间有什么区别？

### 进程间有哪些通信方式？

### 简述 select, poll, epoll 的使用场景以及区别，epoll 中水平触发以及边缘触发有什么不同？

### 简述 Linux 进程调度的算法

### 简述几个常用的 Linux 命令以及他们的功能

### 简述操作系统如何进行内存管理

### 线程有多少种状态，状态之间如何转换

### 简述操作系统中的缺页中断

### 线程间有哪些通信方式？

### 什么时候会由用户态陷入内核态？

### 进程有多少种状态？

### 简述自旋锁与互斥锁的使用场景

### Linux 下如何查看端口被哪个进程占用？

### 操作系统中，虚拟地址与物理地址之间如何映射？

### 进程通信中的管道实现原理是什么？

### Linux 下如何排查 CPU 以及 内存占用过多？

### 简述 Linux 系统态与用户态，什么时候会进入系统态？

### 多线程和多进程的区别是什么？

### 简述 Linux 虚拟内存的页面置换算法

### 创建线程有多少种方式？

### 简述 Linux 零拷贝的原理

### 简述同步与异步的区别，阻塞与非阻塞的区别

### Linux 中虚拟内存和物理内存有什么区别？有什么优点？

### BIO、NIO 有什么区别？怎么判断写文件时 Buffer 已经写满？简述 Linux 的 IO 模型

### 简述 mmap 的使用场景以及原理

### 两个线程交替打印一个共享变量

### 简述操作系统中 malloc 的实现原理

### Linux 下如何查看 CPU 荷载，正在运行的进程，某个端口对应的进程？

### LVS 的 NAT、TUN、DR 原理及区别

### 共享内存是如何实现的？

### 如何调试服务器内存占用过高的问题？

### 系统调用的过程是怎样的？操作系统是通过什么机制触发系统调用的？

### Linux 如何查看实时的滚动日志？

`tail -f log` 或者 `tail -F log`

### malloc 创建的对象在堆还是栈中？

```c
void foo()
{
    int *a_stack_pointer = (int*)malloc(10 * sizeof(int));
}
```

a_stack_pointer 在栈上，保存了它所指向的内容给的地址，用 malloc 分配的动态内存在堆上

### 为什么进程切换慢，线程切换快？

### 简述 CPU L1, L2, L3 多级缓存的基本作用

### 简述创建进程的流程

### 进程空间从高位到低位都有些什么？

### 什么情况下，进程会进行切换？

### 简述 Linux 的 I/O 模型

### 简述 traceroute 命令的原理

### Linux 页大小是多少？

| Architecture                         | Smallest page size | Larger page sizes                                         |
| ------------------------------------ | ------------------ | --------------------------------------------------------- |
| 32-bit x86                           | 4 KiB              | 4 MiB in PSE mode, 2 MiB in PAE mode                      |
| x86-64                               | 4 KiB              | 2 MiB, 1 GiB                                              |
| IA-64 (Itanium)                      | 4 KiB              | 8 KiB, 64 KiB, 256 KiB, 1 MiB, 4 MiB, 16 MiB, 256 MiB     |
| Power ISA                            | 4 KiB              | 64 KiB, 16 MiB, 16 GiB                                    |
| SPARC v8 with SPARC Reference MMU    | 4 KiB              | 256 KiB, 16 MiB                                           |
| UltraSPARC Architecture 2007 8 KiB 6 | 4 KiB              | 64KiB, 512 KiB , 4 MiB, 32 MiB , 256 MiB , 2 GiB , 16 GiB |
| ARMv7                                | 4 KiB              | 64 KiB, 1 MiB , 16 MiB                                    |

### 信号量是如何实现的？

```c
struct semaphore {
    raw_spinlock_t        lock;
    unsigned int        count;
    struct list_head    wait_list;
};

/**
 * down - acquire the semaphore
 * @sem: the semaphore to be acquired
 *
 * Acquires the semaphore.  If no more tasks are allowed to acquire the
 * semaphore, calling this function will put the task to sleep until the
 * semaphore is released.
 *
 * Use of this function is deprecated, please use down_interruptible() or
 * down_killable() instead.
 */
void down(struct semaphore *sem)
{
    unsigned long flags;

    raw_spin_lock_irqsave(&sem->lock, flags);
    if (likely(sem->count > 0))
        sem->count--;
    else
        __down(sem);
    raw_spin_unlock_irqrestore(&sem->lock, flags);
}

/**
 * down_interruptible - acquire the semaphore unless interrupted
 * @sem: the semaphore to be acquired
 *
 * Attempts to acquire the semaphore.  If no more tasks are allowed to
 * acquire the semaphore, calling this function will put the task to sleep.
 * If the sleep is interrupted by a signal, this function will return -EINTR.
 * If the semaphore is successfully acquired, this function returns 0.
 */
int down_interruptible(struct semaphore *sem)
{
    unsigned long flags;
    int result = 0;

    raw_spin_lock_irqsave(&sem->lock, flags);
    if (likely(sem->count > 0))
        sem->count--;
    else
        result = __down_interruptible(sem);
    raw_spin_unlock_irqrestore(&sem->lock, flags);

    return result;
}

/**
 * down_killable - acquire the semaphore unless killed
 * @sem: the semaphore to be acquired
 *
 * Attempts to acquire the semaphore.  If no more tasks are allowed to
 * acquire the semaphore, calling this function will put the task to sleep.
 * If the sleep is interrupted by a fatal signal, this function will return
 * -EINTR.  If the semaphore is successfully acquired, this function returns
 * 0.
 */
int down_killable(struct semaphore *sem)
{
    unsigned long flags;
    int result = 0;

    raw_spin_lock_irqsave(&sem->lock, flags);
    if (likely(sem->count > 0))
        sem->count--;
    else
        result = __down_killable(sem);
    raw_spin_unlock_irqrestore(&sem->lock, flags);

    return result;
}

/**
 * down_trylock - try to acquire the semaphore, without waiting
 * @sem: the semaphore to be acquired
 *
 * Try to acquire the semaphore atomically.  Returns 0 if the semaphore has
 * been acquired successfully or 1 if it cannot be acquired.
 *
 * NOTE: This return value is inverted from both spin_trylock and
 * mutex_trylock!  Be careful about this when converting code.
 *
 * Unlike mutex_trylock, this function can be used from interrupt context,
 * and the semaphore can be released by any task or interrupt.
 */
int down_trylock(struct semaphore *sem)
{
    unsigned long flags;
    int count;

    raw_spin_lock_irqsave(&sem->lock, flags);
    count = sem->count - 1;
    if (likely(count >= 0))
        sem->count = count;
    raw_spin_unlock_irqrestore(&sem->lock, flags);

    return (count < 0);
}

/**
 * down_timeout - acquire the semaphore within a specified time
 * @sem: the semaphore to be acquired
 * @timeout: how long to wait before failing
 *
 * Attempts to acquire the semaphore.  If no more tasks are allowed to
 * acquire the semaphore, calling this function will put the task to sleep.
 * If the semaphore is not released within the specified number of jiffies,
 * this function returns -ETIME.  It returns 0 if the semaphore was acquired.
 */
int down_timeout(struct semaphore *sem, long timeout)
{
    unsigned long flags;
    int result = 0;

    raw_spin_lock_irqsave(&sem->lock, flags);
    if (likely(sem->count > 0))
        sem->count--;
    else
        result = __down_timeout(sem, timeout);
    raw_spin_unlock_irqrestore(&sem->lock, flags);

    return result;
}

/**
 * up - release the semaphore
 * @sem: the semaphore to release
 *
 * Release the semaphore.  Unlike mutexes, up() may be called from any
 * context and even by tasks which have never called down().
 */
void up(struct semaphore *sem)
{
    unsigned long flags;

    raw_spin_lock_irqsave(&sem->lock, flags);
    if (likely(list_empty(&sem->wait_list)))
        sem->count++;
    else
        __up(sem);
    raw_spin_unlock_irqrestore(&sem->lock, flags);
}
```
