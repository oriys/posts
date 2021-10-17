---
title: "Java并发编程实战::结构化并发应用程序"
date: 2021-08-26T21:28:22+08:00
draft: true
---

## 任务执行

通过把应用程序的工作分解到多个任务中，可以简化程序的组织结构，提供一种自然的事务边界来优化错误恢复过程，以及提供一种自然的并行工作结构来提升并发性。

### 在线程中执行任务

在线程中执行任务

当围绕“任务执行”来设计应用程序结构时，第一步就是要找出清晰的任务边界。在理想情况下，各个任务之间是相互独立的：任务并不依赖于其他任务的状态、结果或边界效应。独立性有助于实现并发，因为如果存在足够多的处理资源，那么这些独立的任务都可以并行执行。为了在调度与负载均衡等过程中实现更高的灵活性，每项任务还应该表示应用程序的一小部分处理能力。

串行地执行任务，在服务器应用程序中，串行处理机制通常都无法提供高吞吐率或快速响应性。

```java
class SingleThreadWebServer {
    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            Socket connection = socket.accept();
            handleRequest(connection);
        }
    }
}
```

显式地为任务创建线程

```java
class ThreadPerTaskWebServer {
    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            final Socket connection = socket.accept();
            Runnable task = new Runnable() {
                public void run() {
                    handleRequest(connection);
                }
            };
            new Thread(task).start();
        }
    }
}
```

- 任务处理过程从主线程中分离出来，使得主循环能够更快地重新等待下一个到来的连接。这使得程序在完成前面的请求之前可以接受新的请求，从而提高响应性
- 任务可以并行处理，从而能同时服务多个请求。如果有多个处理器，或者任务由于某种原因被阻寒，例如等待 I/O 完成、获取锁或者资源可用性等，程序的吞吐量将得到提高。
- 任务处理代码必须是线程安全的，因为当有多个任务时会并发地调用这段代码。在正常负载情况下，“为每个任务分配一个线程”的方法能提升串行执行的性能。只要请求的到达速率不超出服务器的请求处理能力，那么这种方法可以同时带来更快的响应性和更高的吞吐率。

无限制创建线程的不足

- 线程生命周期的开销非常高。线程的创建与销毁并不是没有代价的。根据平台的不同，实际的开销也有所不同，但线程的创建过程都会需要时间，延迟处理的请求，并且需要 JVM 和操作系统提供一些辅助操作。如果请求的到达率非常高且请求的处理过程是轻量级的，例如大多数服务器应用程序就是这种情况，那么为每个请求创建一个新线程将消耗大量的计算资源。
- 资源消耗。活跃的线程会消耗系统资源，尤其是内存。如果可运行的线程数量多于可用处理器的数量，那么有些线程将闲置。大量空闲的线程会占用许多内存，给垃圾回收器带来压力，而目大量线程在竞年 CPU 资源时还将产生其他的性能开销。如果你已经拥有足够多的线程使所有 CPU 保持忙碌状态，那么再创建更多的线程反而会降低性能。
- 稳定性。在可创建线程的数量上存在一个限制。这个限制值将随着平台的不同而不同，并且受多个因素制约，包括 JVM 的启动参数、Thread 构造函数中请求的栈大小，以及底层操作系统对线程的限制等

### Executor 框架

Executor 基于生产者一消费者模式，提交任务的操作相当于生产者，执行任务的线程则相当于消费者。

基于 Executor 的 Web 服务器

```java
class TaskExecutionWebServer {
    private static final int NTHREADS = 100;
    private static final Executor exec = Executors.newFixedThreadPool(NTHREADS);
    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            final Socket connection = socket.accept();
            Runnable task = new Runnable() {
                public void run() {
                    handleRequest(connection);
                }
            };
            exec.execute(task);
        }
    }
}
```

通过使用 Executor，将请求处理任务的提交与任务的实际执行解耦开来，并且只需采用另一种不同的 Executor 实现，就可以改变服务器的行为。改变 Executor 实现或配置所带来的影响要远远小于改变任务提交方式带来的影响

执行策略

- 在执行策略中定义了任务执行的 'What、Where、When、How' 等方面
  - 在什么（What）线程中执行任务？
  - 任务按照什么(What）顺序执行 (FIFO、LIFO、优先级）？
  - 有多少个（How Many）任务能并发执行？
  - 在队列中有多少个（How Many）任务在等待执行？
  - 如果系统由于过载而需要拒绝一个任务，那么应该选择哪一个（Which）任务？另外，如何 (How)通知应用程序有任务被拒绝？
  - 在热行一个任务之前或之后，应该进行哪些（What）动作？

线程池

指管理一组同构工作线程的资源池。线程池是与工作队列（Work Oueue）密切相关的，其中在工作队列中保存了所有等待执行的任务。工作者线程（Worker Thread）的任务很简单：从工作队列中获取一个任务，执行任务，然后返回线程池并等待下一个任务。

“在线程池中执行任务”比“为每个任务分配一个线程”优势更多。通过重用现有的线程而不是创建新线程，可以在处理多个请求时分摊在线程创建和销毁过程中产生的巨大开销。另一个额外的好处是，当请求到达时，工作线程通常已经存在，因此不会由于等待创建线程而延任务的执行，从而提高了响应性。通过适当调整线程池的大小，可以创建足够多的线程以便使处理器保持忙碌状态，同时还可以防止过多线程相互竞争资源而使应用程序耗尽内存或失败、

Executors 的静态工厂方法

- newFixedThreadPool. newFixedThreadPool 将创建一个固定长度的线程池，每当提交个任务时就创建一个线程，直到达到线程池的最大数量，这时线程池的规模将不再变化（如果某个线程由于发生了未预期的 Exception 而结束，那么线程池会补充一个新的线程）。
- newCachedThreadPool. newCachedThreadPool 将创建一个可缓存的线程池，如果线程池的当前规模超过了处理需求时，那么将回收空闲的线程，而当需求增加时，则可以添加新的线程，线程池的规模不存在任何限制。
- newSingleThreadExecutor. newSingleThreadExecutor 是一个单线程的 Executor，它创建单个工作者线程来执行任务，如果这个线程异常结束，会创建另一个线程来替代。newSingleThreadExecutor 能确保依照任务在队列中的顺序来串行执行。
- newScheduledThreadPool. newScheduledThreadPool 创建了一个固定长度的线程池。而且以延迟或定时的方式来执行任务，类似于 Timer。

Executor 的生命周期

为了解决执行服务的生命周期问题，Executor 扩展了 ExecutorService 接口，添加了一些用于生命周期管理的方法

```java
public interface ExecutorService extends Executor {
    void shutdown();
    List<Runnable> shutdownNow();
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit)
            throws InterruptedException;
    // ... additional convenience methods for task submission
}
```

ExecutorService 的生命周期有 3 种状态：运行、关闭和已终止。ExecutorService 在初始创建时处于运行状态。shutdown 方法将执行平缓的关闭过程：不再接受新的任务，同时等待已经提交的任务执行完成一包括那些还末开始执行的任务。shutdownNow 方法将执行粗暴的关闭过程：它将尝试取消所有运行中的任务，并且不再启动队列中尚末开始执行的任务。

延迟任务和周期任务

Timer 存在一些缺陷，因此应该考虑使用 ScheduledThreadPoolExecutor 来代替它。可以通过 ScheduledThreadPoolExecutor 的构造函数或 newScheduledThreadPool 工厂方法来创建该类的对象。

如果要构建自己的调度服务，那么可以使用 DelayQueue，它实现了 BlockingQueue，并为 ScheduledThreadPoolExecutor 提供调度功能。DelayQueue 管理着一组 Delayed 对象。每个 Delayed 对象都有一个相应的延迟时间：在 DelayQueue 中，只有某个元索逾期后，才能从 DelayQueue 中执行 take 操作。从 DelayQueue 中返回的对象将根据它们的延迟时间进行排序。

### 超出可利用的并行性

