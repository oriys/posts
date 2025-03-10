---
title: "现代操作系统::多处理器系统"
date: 2021-07-21T02:11:01+08:00
draft: false
keywords: ["多处理器系统", "UMA", "NUMA", "多核芯片", "群调度"]
---

## 多处理器系统

- 多处理机
  - 共享存储器多处理机是这样一种计算机系统，其两个或更多的 CPU 全部共享访问一个公用的 RAM。
  - CPU 可对存储器的某个字写入某个值，然后读回该字，并得到一个不同的值。在进行恰当组织时，这种性质构成了处理器间通信的基础:一个 CPU 向存储器写人某些数据而另一个读取这些数据。
  - 多处理器硬件
    - 所有的多处理机都具有每个 CPUi 可访问全部存储器的性质
    - UMA 多处理器读出每个存储器字的速度是一样快的，NUMA 没有这种特性
    - [基于总线的 UMA 多处理器结构](https://imgur.com/fotq9F5.png)
      - 两个或更多的 CPU 以及一个或多个存储器模块都使用同一个总线进行通信
      - 当一个 CPU 需要读一个存储器字 (memory word) 时，它首先检查总线忙否。 如果总线空闲，该 CPU 把所需字的地址放到总线上，发出若干控制信号，然后等待存储器把所需的字放到总线上。当 CPU 核数上升时，效率下降严重。
      - 解决方案是为每个 CPU 添加一个高速缓存
        - 高速缓存可以位于 CPU 芯片的内部、 CPU 附近、在处理器板上或所有这三种方式的组合
        - 高速缓存 不以单个字为基础，而是以 32 字节或 64 字节块为基础
        - 高速缓存一致性协议
          - CPU 试图在一个或多个远程高速缓存中写入一个字，总线硬件检测到写，并把一个信号放到总线上通知所有其他的高速缓存。如果其他高速缓存有个“干净"的副本，也就是同存储器内容完全一样的副本，那么它们可以丢弃该副本井让写者在修改之前从存储器取出高速缓存块。如果某些其他高速缓存有"脏"(被修改过)副本，它必须在处理写之前把数据写回存储器或者把它通过总线直接传送到写者上。
    - [基于交叉开关的 UMA 多处理机](https://i.imgur.com/hzKkxlq.png)
      - 连接 n 个 CPU 到 K 个存储器的最简单的电路是交叉开关
      - 水平线(进线)和垂直线(出线)的每个相交位咒上是一个交叉点 (crosspoint)。交叉点是一个小 的电子开关，具体取决千水平线和垂直线是否偕要连接。
      - 非阻塞网络
    - [使用多级交换网络的 UMA 多处理机](https://i.imgur.com/pKoIru0.png)
      - 有一种完全不同的、基于简单 2 x 2 开关的多处理机设计。 这个开关有两个输入和两个输出。到达任意一个输入线的消息可以被交换至任意一个输出线上。就我们的目标而言，消息可由四个部分组成。Module (模块)域指明使用哪个存储器。Address (地址)域指定在模块中的地址。Opcode (操作码)给定了操作，如 READ 或 WRITE。最后，在可选的 Value (值)域中可包含一个操作数，比如一个要被 WRITE 写入的 32 位字。该开关检查 Module 域并利用它确定消息是应该送给 X 还是发送给 Y。
      - 这个 2 x 2 开关可有多种使用方式，用以构建大型的多级交换网络。 有一种是简单经济的 omega 网络，
      - Omega 网络的接线模式常被称作全混洗(perfect shuffle)。
      - 接着看看 Omega 网络是如何工作的，假设 CPU 011 打算从存储器模块 110 读取一个字。CPU 发送 READ 消息给开关 1D, 它在 Module 域包含 110。ID 开关取 110 的首位(最左位)并用它进行路由处理。0 路由到上端输出，而 1 的路由到下端，由于该位为 1, 所以消息通过低端输出被路由到 2D。
      - 所有的第二级开关，包括 2D,取用第二个比特位进行路由。这.位还是 1，所以消息通过低端输出转发到 3D。在这里对第三位进行测试，结果发现是 0。于是，消息送往上端输出，并达到所期望的存储器 110。
      - 在消息通过交换网络之后，模块号的左端的位就不再需要了。它们可以有很好的用途，可以用来记录入线编号，这样，应答消息可以找到返回路径。对于路径 a，入线编号分别是 0 (向上输入到 1D)、1(低输入到 2D)，1(低输入到 3D)、使用 011 作为应答路由，只要从右向左读出每位即可。
      - 阻塞网络
    - [NUMA 多处理机](https://i.imgur.com/rGiHg3S.png)
      - 所有 NUMA 机器都具有以下三种关键特性，它们是 NUMA 与其他多处理机的主要区别:
        - 具有对所有 CPU 都可见的单个地址空间。
        - 通过 LOAD 和 STORE 指令访问远程存储器。
        - 访问远程存储器慢于访问本地存储器。
      - 在对远程存储器的访问时间不被隐藏时(因为没有高速缓存)，系统被称为 NC-NUMA (No Cache NUMA,无高速缓存 NUMA)。在有一致性高速缓存时，系统被称为 CC-NUMA (Cache Coherent NUMA,高速缓存一致 NUMA)。
      - 目前构造大型 CC-NUMA 多处理机最常见的方法是基于目录的多处理机(directory-based multiprocessor)。其基本思想是，维护个数据库来记录高速缓存行的位置及其状态。当一个高速缓存行被引用时，就查询数据库找出高速缓存行的位置以及它是“干净”的还是“脏”的。由于每条访问存储器的指令都必须查询这个数据库，所以它必须配有极高速的专用硬件。
    - 多核芯片
      - CPU 拥有多于一个内核
      - CPU 可能共享高速缓存或者不共享，但是它们都共享内存
      - 特殊的硬件电路可以确保在一个字同时出现在两个或者多个的高速缓存中的情况下，当其中某个 CPU 修改了该字，所有其他高速缓存中的该字都会被自动地并且原子性地删除来确保一致性。这个过程称为窥探
    - 众核芯片
      - 众核芯片是指包括几十、几百甚至成千上万个核心的多核处理器
      - 保持缓存一致性的机制会变得非常复杂和昂贵，一致性壁垒
      - 一些工程师认为，唯一已证明可适用于众核的编程模型是采用消息传递和分布式内存实现的。
      - 成千上万的核心数现在已不再那么少见了，图形处理单元(GPU)作为当今最为常见的众核。
      - 于几乎任何一台非嵌入式并且有显示器的计算机系统中。GPU 是一个拥有专用内存和成千上万个微小核的处理器则。因而它们十分擅长进行像图形程序渲染多边形这样的大量并行的小规模计算，而不太擅长串行任务，同时也很难对它们编程。
    - 异构多核
      - 一些芯片会把一个 GPU 和一些通用处理器核封装在一起，许多片上系统在通用处理器核之外还包括一个或多个特殊用途的处理器。在一块芯片上封装了不同类型的处理器的系统袚统称为异构多核处理器
    - 在多核上编程
      - 当前的编程语言井不适合编写高度并行的程序，好的编译器和调试工具也很少见，程序员很少有井行编程的经验
      - 同步、消除资源竞争条件和避免死锁等问题就像噩梦一般
      - 应用场景
  - 多处理机操作系统类型
    - 每个 CPU 有自己的操作系统
      - 允许所有的 CPU 共享操作系统的代码，而且只需要提供数据的私有副本
      - 四个潜在的问题
        - 首先，在一个进程进行系统调用时，该系统调用是在本机的 CPU 上被捕获并处理的，并使用操作系统表中的数据结构。
        - 其次，因为每个操作系统都有自己的表，那么它也有自己的进程集合，通过自身调度这些进程。
        - 第三，没有共享物理页面。会出现如下的情形:在 CPU2 不断地进行页面调度时 CPU1 却有多余的页面。由于内存分配是固定的，所以 CPU 2 无法向 CPU 1 借用页面。
        - 第四，也是最坏的情形，如果操作系统维护近期使用过的磁盘块的缓冲区高速缓存，每个操作系统都独自进行这种维护工作，因此，可能出现某一修改过的磁盘块同时存在于多个缓冲区高速缓存的情况，这将会导致不一致性的结果。避免这一问题的唯一途径是，取消缓冲区高速缓存。
    - 主从多机处理机
      - 在这种模型中，操作系统的一个副本及其数据表都在 CPU 1 上，而不是在其他所有 CPU 上。为了在该 CPU 1 上进行处理，所有的系统调用都重定向到 CPU 1 上。如果有剩余的 CPU 时间，还可以在 CPU 1 上运行用户进程。这种模型称为主从模型( master-slave), 因为 CPU 1 是主 CPU，而其他的都是从属 CPU。
      - 主从模型解决了在第一种模型中的多数问题。有单一的数据结构(如一个链表或者一组优先级链表)用来记录就绪进程。当某个 CPU 空闲下来时，它向 CPU 1 上的操作系统请求一个进程运行，并被分配一个进程。这样，就不会出现一个 CPU 空闲而另一个过载的情形。类似地，可在所有的进程中动态地分配页面，而且只有一个缓冲区高速缓存，所以决不会出现不一致的情形。
      - 这个模型的问题是，如果有很多的 CPU, 主 CPU 会变成一个瓶颈。
    - 对称对处理机
      - 消除了上述的不对称性。在存储器中有操作系统的一个副本，但任何 CPU 都可以运行它。在有系统调用时，进行系统调用的 CPU 陷入内核并处理系统调用
      - 任何 CPU 都可以运行操作系统，但在任一时刻只有一个 CPU 可运行操作系统
      - 可以把操作系统分割成互不影响的临界区。每个临界区由其互斥信号量保护，所以一次只有一个 CPU 可执行它。采用这种方式，可以实现更多的井行操作。
  - 多处理机同步
    - 如果一个进程在单处理机(仅含一个 CPU)中需要访问一些内核临界表的系统调用，那么内核代码在接触该表之前可以先禁止中断。然后它继续工作，在相关工作完成之前，不会有任何其他的进程溜进来访问该表。
    - 在多处理机中，禁止中断的操作只影响到完成禁止中断操作的这个 CPU，其他的 CPU 继续运行并且可以访问临界表。因此，必须采用一种合适的互斥信号量协议，而且所有的 CPU 都遵守该协议以保证互斥工作的进行。
    - 任何实用的互斥信号量协议的核心都是一条特殊指令， 该指令允许检测一个存储器字并以一种不可见的操作设置。指令 TSL (Test and Set Lock 做的是，读出一个存储器字并把它存储在一个寄存器中。同时，它对该存储器字写入一个 1 (或某些非零值)。当然，这需要两个总线周期来完成存储器的读写。在单处理机中，只要该指令不被中途中断，TSL 指令就始终照常工作。
    - 为了阻止多个 CPU 同时访问同一个存储器字，SL 指令必须首先锁住总线，阻止其他的 CPU 访问它，然后进行存储器的读写访问，再解锁总线。
    - 如果正确地实现和使用 TSL，它能够保证互斥机制正常工作。但是这种互斥方法使用了自旋锁(spin lock), 因为请求的 CPU 只是在原地尽可能快地对锁进行循环测试。这样做不仅完全浪费了提出请求的各个 CPU 的时间，而且还给总线或存储器增加了大量的负载，严重地降低了所有其他 CPU 从事正常工作的速度。
    - 在获取锁之前先观察锁是否空闲，可以减少竞争，避免高速缓存颠簸。
    - 另一个减少总线流量的方式是使用著名的以太网二进制指数回退算法
    - 一个更好的想法是，让每个打算获得互斥信号量的 CPU 都拥有各自用于测试的私有锁变且
    - 自旋与同步
      - 假设一些 CPU 是空闲的，需要访问共享的就绪链表(ready list) 以便选择一个进程运行。如果就绪链表被锁住了，CPU 必须保持等待直到能够访问该就绪链表。
      - 然而，在另外一些情形中，却存在着别的选择。例如，如果在一个 CPU 中的某些线程需要访问文件系统缓冲区高速缓存，而该文件系统缓冲区高速缓存正好锁住了，那么 CPU 可以决定切换至另外一个线程而不是等待。
      - 假设自旋和进行线程切换都是可行的选择，则可进行如下的权衡。自旋直接浪费了 CPU 周期。重复地测试锁并不是高效的工作。不过，切换也浪费了 CPU 周期，因为必须保存当前线程的状态，必须获得保护就绪链表的锁，还必须选择一个线程，必须装入其状态，并且使其开始运行。花费在这两个线程间来回切换和所有高速缓存未命中的周期时间都浪费了。
      - 有一种设计是总是进行自旋。 第二种设计 方案则总是进行切换。 而第三种设计方案是每当遇到一个锁住的互斥信号量时，就单独做出决定。在必须做出决定的时刻，并不知道自旋和切换哪种方案更好，但是对于任何给定的系统，有可能对其所有的有关活动进行跟踪，并且随后进行离线分析。然后就可以确定哪个决定最好及在最好情形下所浪费的时间。
      - 得出来了这样一个模型: 一个未能获得互斥信号量的线程自旋一段时间。 如果时间超过某个阈值，则进行切换。在某些情形下，该阈值是一个定值，典型值是切换至另一个线程再切换回来的开销。在另一些情形下，该阈值是动态变化的，它取决于所观察到的等待互斥信号量的历史信息。
  - 多处理机调度
    - 线程是内核线程还是用户线程至关重要。如果线程是由用户空间库维护的，而对内核不可见，那么调度一如既往的基于单个进程。如果内核并不知道线程的存在，它就不能调度线程。
    - 对内核线程来说，情况有所不同。在这种情况下所有线程均是内核可见的，内核可以选择一个进程的任一线程。
    - 在单处理机中，调度是一维的。唯一必须(不断重复地)回答的问题是:“接下来运行的线程应该是哪一个?”而在多处理机中，调度是二维的。调度程序必须决定哪一个进程运行以及在哪一个 CPU 上运行。这个在多处理机中增加的维数大大增加了调度的复杂性。
    - 另一个造成复杂性的因素是，在有些系统中所有的线程是不相关的，它们属于不同的进程，彼此无关。而在另外一些系统中它们是成组的，同属于同一个应用并且协同工作。前一种情形的例子是服务器系统，其中独立的用户运行相互独立的进程。这些不同进程的线程之间没有关系，后一种情形通常出现在程序开发环境中。
    - 分时
      - 让我们首先讨论调度独立线程的情况。处理独立线程的最简单算法是，为就绪线程维护一个系统级的数据结构，它可能只是一个链表，但更多的情况下可能是对应不同优先级一个链表集合。
      - 由所有 CPU 使用的单个调度数据结构分时共享这些 CPU，正如它们在一个单处理机系统中那样。它还支持自动负载平衡，因为决不会出现一个 CPU 空闲而其他 CPU 过载的情况。不过这一方法有两个缺点，一个是随着 CPU 数量增加所引起的对调度数据结构的潜在竞争，二是当线程由于 I/O 阻塞时所引起上下文切换的开销(overhead)。
      - 为了避免这种异常情况，一些系统采用智能调度(smartscheduling)的方法，其中，获得了自旋锁的线程设置一个进程范围内的标志以表示它目前拥有了一个自旋锁。 当它释放该自旋锁时，就清除这个标志。这样调度程序就不会停止持有自旋锁的线程，相反，调度程序会给予稍微多一些的时间让该线程完成临界区内的工作并释放自旋锁。
      - 有些多处理机考虑了这一因素，并使用了所谓亲和调度。其基本思想是，尽量使一个线程在它前一次运行过的同一个 CPU 上运行。创建这种亲和力(affinity)的一种途径是采用一种两级调度算法(two-level scheduling algorithm)。
      - 在一个线程创建时，两级调度算法有三个优点。
        - 它把负载大致平均地分配在可用的 CPU 上;
        - 它尽可能发挥了高速缓存亲和力的优势
        - 通过为每个 CPU 提供一个私有的就绪线程链表，使得对就绪线程链表的竞争减到了最小，因为试图使用另一个 CPU 的就绪线程链表的机会相对较小。
    - 空间共享
      - 在多个 CPU 上同时调度多个线程称为空间共享(space sharing)。
      - 最简单的空间共享算法是这样工作的。假设一组相关的线程是一次性创建的。在其创建的时刻，调度程序检查是否有同线程数量一样多的空闲 CPU 存在。如果有，每个线程获得各自专用的 CPU(非多道程序处理)并且都开始运行。如果没有足够的 CPU,就没有线程开始运行，直到有足够的 CPU 时为止。
      - 在单处理机系统中，最短作业优先是批处理调度中知名的算法。在多处理机系统中类似的算法是，选择需要最少的 CPU 周期数的线程，也就是其 CPU 周期数\*运行时间最小的线程为候选线程。然而，在实际中，这一信息很难得到，因此该算法难以实现。事实上，研究表明，要胜过先来先服务算法是非常困难的。
      - 在这个简单的分区模型中，一个线程请求一定数量的 CPU，然后或者全部得到它们或者一直等到有足够数量的 CPU 可用为止。
      - 另一种处理方式是主动地管理线程的并行度。管理并行度的一种途径是使用一个中心服务器，用它跟踪哪些线程正在运行，哪些线程希望运行以及所需 CPU 的最小和最大数量。 每个应用程序周期性地询问中心服务器有多少个 CPU 可用。然后它调整线程的数量以符合可用的数量。
    - 群调度
      - 空间共享的一个明显优点是消除了多道程序设计，从而消除了上下文切换的开销。但是，一个同样明显的缺点是当 CPU 被阻塞或根本无事可做时时间被浪费了，只有等到其再次就绪。于是，人们寻找既可以调度时间又可以调度空间的算法，特别是对于要创建多个线程而这些线程通常需要彼此通信的线程。
      - 这一问题的解决方案是群调度(gang scheduling), 它是协同调度的发展产物。群调度由三个部分组成:
        - 把一组相关线程作为一个单位，即一个群(gang)， 一起调度。
        - 一个群中的所有成员在不同的分时 CPU 上同时运行。
        - 群中的所有成员共同开始和结束其时间片。
      - 使群调度正确工作的关键是，同步调度所有的 CPU。这意味着把时间划分为离散的时间片。

## 多计算机

- 多计算机是紧耦合 CPU,不共享存储器。每台计算机有自己的存储器。众所周知，这些系统有各种其他的名称，如机群计算机以及工作站机群。云计环服务都是建立在多计算机上，因为它们需要大的计算能力。
- 多计算机硬件
  - 一个多计算机系统的基本节点包括一个 CPU、存储器、一个网络接口，有时还有一个硬盘。节点可以封装在标准的 PC 机箱中，不过通常没有图像适配卡、显示器、键盘和鼠标等。有时这种配置被称为无主工作站、因为没有用户。有用户的工作站逻辑上应该对应地被叫作有主工作站
  - 互联技术
    - 在每个节点上有一块网卡，带有一根或两根从网卡上接出的电缆(或光纤)。这些电缆或者连到其他的节点上，或者连到交换机上
    - 在多计算机中可采用两种交换机制
      - 在存储转发包交换,由源节点的网络接口卡注入第一个交换机的包组成。比特串一次进来一位，当整个包到达一个输入缓冲区时，它被复制到沿着其路径通向下一个交换机的队列当中。当数据包到达目标节点所连接的交换机时，该数据包被复制到目标节点的网络接口卡，并最终到达其 RAM.
      - 另一种交换机制是电路交换,它包括由第一个交换机建立的，通过所有交换机而到达目标交换机的一条路径。一且该路径建立起来，比特流就从源到目的地通过整个路径不断地尽快输送。在所涉及的交换机中，没有中间缓冲。电路交换需要有一个建立阶段，它需要一点时间，但是一且建立完成，速度就很快。在包发送完毕之后，该路径必须被拆除。电路交换的一种变种称为虫孔路由,它把每个包拆成子包，井允许第一个子包在整个路径还没有完全建立之前就开始流动。
  - 网络接口
    - 接口板上可以有一个或多个 DMA 通道，甚至在板上有一个完整的 CPU (乃至多个 CPU)。通过请求在系统总线上的块传送, DMA 通道可以在接口板和主 RAM 之间以非常高的速率复制包，因而可以一次性传送若干字而不需要为每个字分别请求总线
    - 很多接口板上有一个完整的 CPU, 可能另外还有一个或多个 DMA 通道。它们被称为网络处理器, 并且其功能日趋强大。这种设计意味着主 CPU 将一些工作分给了网卡，诸如处理可靠的传送、多播、压缩/解压缩、加密/解密以及在多进程系统中处理安全事务等。但是，有两个 CPU 则意味着它们必须同步，以避免竞争条件的发生，这将增加额外的开销，井且对千操作系统来说意味着要承担更多的工作。
    - 跨层复制数据是安全的，但不一定高效
  - 低层通信软件
    - 在多计算机系统中高性能通信的敌人是对包的过度复制
    - 为了避免这种对性能的影响，不少多计算机把接口板映射到用户空间，井允许用户进程直接把包送到卡上，而不需要内核的参与
    - 如果在节点上有若于个进程运行而且需要访问网络以发送包，一个解决方案是，把接口板映射到所有需要它的进程中去，但是这样做就需要有一个机制用以避免竞争
    - 内核本身会经常需要访问互连网络，最简单的设计是使用两块网络接口板，一块映射到用户空间供应用程序使用，另一块映射到内核空间供操作系统使用。许多多计算机就正是这样做的.
    - 另一方面，较新的网络接口通常是多队列的，这意味若它们有多个缓冲区可以有效地支持多个用户.例如, Intel 1350 系列网卡具有 8 个发送和 8 个接收队列，可虚拟化为许多虚拟端口
    - 节点至网络接口通信
      - 将包送到接口板上。最快的方法是使用板上的 DMA 芯片直接将它们从 RAM 复制到板上
      - 这种方式的问题是，DMA 可以使用物理地址而不是虚拟地址，井且独立干 CPU 运行，除非存大 I/0 MMU。
      - 另外，如果操作系统决定替换一个页面.而 DMA 芯片正在从该页面复制一个包，就会传送错误的数据。然而更加糟糕的是，如果操作系统在替换某一个页面的同时 DMA 芯片正在把一个包复制进该页面，结果不仅进来的包会丢失，无辜的存储器页面也会被毁坏，这可能会带来灾难性的后果。
      - 可采用一类将页面钉住和释放的系统调用，把有关页面标记成暂时不可交换的。
    - 远程直接内存访间
      - 降低数据的复制品都需要很大代价。为了应对这一问题，一些网络接口支持远程直接内存访问 (RMDA) 技术，允许一台机器直接访问另一台机器的内存。RMDA 不需要操作系统的参与，直接从应用的内存空间中读取或写人数据.
  - 用户层通信软件
    - 在多计算机中，不同 CPU 上的进程通过互相发送消息实现通信。在最简单的情况下，这种消息传送是暴露给用户进程的。换句话说，操作系统提供了一种发送和接收消息的途径，而库过程使得这些低层的调用对用户进程可用。在较复杂的情形下，通过使得远程通信看起来像过程调用的办法，将实际的消息传递对用户隐藏起来
  - 远程过程调用
    - 允许程序调用位于其他 CPU 中的过程。当机器 1 的进程调用机器 2 的过程时，在机器 1 中的调用进程被挂起，在机器 2 中被调用的过程执行。可以在参数中传递从调用者到被调用者的信息，并且可在过程的处理结果中返回信息。根本不存在对程序员可见的消息传递或 I/O。 这种技术即是所谓的远程过程调用,并且已经成为大多计算机的软件的基础。习惯上，称发出调用的过程为客户机，而称被调用的过程为服务器
  - 分布式共享存储器
    - 每台机器有其自己的虚拟内存和页表。当一个 CPU 在一个它并不拥有的页面上进行 LOAD 和 STORE 时，会陷入到操作系统当中。然后操作系统对该页面进行定位，并请求当前持有该页面的 CPU 解除对该页面的映射并通过互连网络发送该页面。在该页面到达时，页面被映射进来，于是出错指令重新启动。事实上，操作系统只是从远程 RAM 中而不是从本地磁盘中满足了这个缺页异常。对用户而言，机器看起来拥有共享存储器。
  - 多计算机调度
    - 维护就绪进程的一个中心链表，每个进程只能在其当前所在的 CPU 上运行。不过，当创建一个新进程时，存在着一个决定将其放在哪里的选择，例如，从平衡负载的考虑出发。
  - 负载平衡
    - 图论确定算法：有一类被广泛研究的算法用干下面这样一个系统，该系统包含已知 CPU 和存储器需求的进程，以及给出每对进程之间平均流量的已知矩阵。如果进程的数众大于 CPU 的数量 k, 则必须把若干个进程分配给每个 CPU。其想法是以最小的网络流量完成这个分配工作。
    - 发送者发起的分布式启发算法：当进程创建时，它就运行在创建它的节点上，除非该节点过载了。过载节点的度量可能涉及太多的进程，过大的工作集，或者其他度量。如果过载了，该节点随机机选择另一个节点井询问它的负载情况。如果被探查的节点负载低于某个阈值，就将新的进程送到该节点上。如果不是，则选择另一个机器探查。探查工作并不会永远进行下去。在 N 次探查之内，如果没有找到合适的主机，算法就终止，且进程继续在原有的机器上运行。整个算法的思想是负载较重的节点试图甩掉超额的工作。应该看到在负载重的条件下，所有的机器都会持续地对其他机器进行探查，徒劳地试配找到一台愿意接收更多工作的机器。几乎没有进程能够被卸载，可是这样的尝试会带来巨大的开销 。
    - 接收者发起的分布式启发算法：在这个算法中，只要有一个进程结束，系统就检查是否有足够的工作可做。如果不是，它随机选择某台机器井要求它提供工作。如果该台机器没有可提供的工作，会接若询问第二台，然后是第三台机器。如果在 N 次探查之后，还是没有找到工作，该节点暂时停止询问，去做任何已经安排好的工作，而在下一个进程结束之后机器会再次进行询问。如果没有可做的工作，机器就开始空闲。在经过固定的时间间隔之后，它又开始探查。

## 分布式系统

- 这些系统与多计算机类似，每个节点都有自己的私有存储器，整个系统中没有共享的物理存储器。但是，分布式系统与多计算机相比，耦合度更加松散。
- 分布式系统添加在其底层网络上的是一些通用范型(模型)，它们提供了一种统一的方法来观察整个系统。分布式系统想要做的是，将松散连接的大量机器转化为基于一种概念的一致系统。这些范型有的比较简单，而有的是很复杂的，但是其思想则总是提供某些东西用来统一整个系统。
- 分布式系统面对不同硬件和操作系统实现某种统一性的途径是，在操作系统的顶部添加一层软件。这层软件称为中间件。这层软件提供了一些特定的数据结构和操作，从而允许散布的机器上的进程和用户用一致的方式互操作。
- 网络硬件
  - 以太网
    - 经典的以太网，在 IEEE802.3 标准中有具体描述，由用来连接若干计算机的同轴电缆组成
    - 许多计算机连接到同一根电缆上，需要一个协议来防止混乱。要在以太网上发送包，计算机首先要监听电缆，看看是否有其他的计算机正在进行传输。如果没有，这台计算机便开始传送一个包，其中有一个短包头，随后是 0 到 1500 字节的有效信息载荷 (payload)。如果电缆正在使用中，计算机只是等待直到当前的传输结束，接着该台计算机开始发送。
    - 如果两台计算机同时开始发送，就会导致冲突发生，两台机器都做检测。采用二进制指数回退算法。
    - 为了避免碰撞问题，现代以太网使用交换机。每个交换机有若干个端口. 一个端口用于连接一台计算机 一个以太网或另一个交换机
  - 因特网
    - Internet 包括了两类计算机，主机和路由器。主机 (host) 有 PC、笔记本计算机、掌上电脑、服务器、大型计算机以及其他那些个人或公司所有且希望与 Internet 连接的计算机。
    - 地区网络和 ISP 的路由器通过中等速度的光纤连接到主于网上。依次，每个配备路由器的公司以太网连接到地区网络的路由器上。而 ISP 的路由器则被连接到供 ISP 客户们使用的调制解调器汇集器上。按照这种方式，在 Internet 上的每台主机至少拥有通往其他主机的一条路径.而且每台经常拥有多条通往其他主机的路径。
    - 在 Internet 上的所有通信都以包的形式传送
- 网络服务和协议
  - 所有的计算机网络都为其用户(主机和进程)提供一定的服务，这种服务通过某些关干合法消息交换的规则加以实现
  - 计算机网络为使用网络的主机和进程提供服务。 包括面向连接的服务和无连接服务。
  - 每种服务可以用服务质量 (quality of service) 表征。有些服务就其从来不丢失数据而言是可靠的
  - 网络协议
    - 所有网络都有高度专门化的规则，用以说明什么消息可以发送以及如何响应这些消息
    - 所有的现代网络都使用所谓的 协议栈 (protocol stack) 把不同的协议一层一层叠加起来。每一层解决不同的问题
    - 大多数分布式系统都使用 Internet 作为基础，因此这些系统使用的关键协议是两种主要的 Internet 协议: IP 和 TCP.
    - DNS 用来寻找 IP 地址
- 基于文档的中间件
  - WWW
- 基于文件系统的中间件
  - 使一个分布式系统看起来像一个大型文件系统
- 基于对象的中间件
  - 对象是变量的集合，这些变昼与一套称为方法的访问过程绑定在一起。进程不允许直接访间这些变量。相反，要求它们调用方法来访问。
- 基于协作的中间件
  - Linda： 耶鲁大学研发的用干通信和同步的新系统。在 Linda 系统中，相互独立的进程之间通过一个抽象的元组空间进行通信。对整个系统而言，元组空间是全局性的，在任何机器上的进程都可以把元组插入或移出元组空间，而不用考虑它们是如何存放的以及存放在何处。对于用户而言，元组空间像一个巨大的全局共享存储器
  - 发布/订阅：它由大量通过广播网网络互联的进程组成。每个进程可以是一个信息生产者、信息消费者或两者都是。

## 有关多处理机系统的研究

- 重新设计一个专门针对多核硬件的操作系统
- 分布式一致性
- 何使大型应用可以在多核和多处理器的环境下进行扩展
- 调试并行应用
- 降低多处理器系统功耗的工作
