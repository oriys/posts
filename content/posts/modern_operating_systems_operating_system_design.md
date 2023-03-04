---
title: "现代操作系统::操作系统设计"
date: 2021-07-23T02:11:01+08:00
draft: false
---

## 操作系统设计

- 设计问题的本质
  - 目标
    - 定义抽象概念
      - 进程、文件、线程、信号量
    - 提供基本操作
      - 每一个抽象概念可以通过具体数据结构的形式来实例化。用户可以创建进程、文件、信号量等。基本操作则处理这些数据结构。例如，用户可以读写文件。基本操作以系统调用的形式实现。从用户的观点来看，操作系统的核心是由抽象概念与其上的基本操作所构成的，而基本操作则可通过系统调用加以利用。
    - 确保隔离
      - 用户之间的隔离
      - 虚拟机之间的隔离
      - 进程之间的隔离
      - 故障隔离
      - 资源共享
    - 管理硬件
      - 提供一个框架，统一管理不同型号的硬件资源
  - 难点
    - 已经发展成了极其复杂的程序
    - 必须处理并发和并发导致竞争条件、死锁等问题。
    - 必须处理可能有敌意的用户
    - 需要提供在不同用户之间共享资源的能力
    - 设计人员需要思考未来的操作系统设计方向
    - 需要提供想当程度的通用性
    - 可移植性
    - 需要保持向后兼容
- 接口设计
  - 指导原则
    - 简单：一个简单的接口更加易于理解并且更加易于以无差错的方式实现
    - 完备：接口必须能够做用户需要做的一切事情，也就是说，它必须是完备的
    - 效率：如果一个功能特性或者系统调用不能够有效地实现，或许就不值得包含它。
  - 范型
    - 用户界面范型：不管选择什么范型，重要的是所有应用程序都要使用它。因此，系统设计者需要提供库和工具包给应用程序开发人员，使他们能够访问产生一致的外观与感觉的程序。没有工具，应用开发者做出来的东西可能完全不同。
    - 执行范型：
      - 算法范型：启动一个程序是为了执行某个功能，而该功能是事先知道的或者是从其参数获知的。
      - 事件驱动范型：在这里程序执行某种初始化（例如通过显示某个屏幕），然后等待操作系统告诉它第一个事件。事件经常是键盘敲击或鼠标移动。这一设计对干高度交互式的程序是十分有益的。
    - 数据范型：
      - 一切皆磁带：在早期的 FORTRAN 批处理系统中，所有一切都是作为连续的磁带来建立模型。用于读入的卡片组被看作输入磁带，用于穿孔的卡片组被看作输出磁带，井且打印机给出被看作输出磁带。磁盘文件也被看作磁带
      - 一切皆文件：UNIX 用”所有一切都是文件”的模型一步发展了这一思想。使用这一范型，所有 I/O 设备都被看作文件，井且可以像普通文件一样打开和操作。
      - 一切皆对象：Windows 试图使所有一切看起来像是一个对象。一旦一个进程获得了一个指向文件、进程、信号量、 邮箱或者其他内核对象的有效句柄，它就可以在其上执行操作。这一范型甚至比 UNIX 更加一般化，井且比 FORTRAN 要一般化得多。
      - 一切皆文档：Web 背后的范型是充满了文档的超空间，每一个文档具有一个 URL。通过键人一个 URL 或者点击被 URL 所支持的条目，你就可以得到该文档。
  - 系统调用接口
    - 操作系统应该提供恰好够用的系统调用，井且每个系统调用都应该尽可能简单
    - 在某些情况下，系统调用可能需要若干变体，但是通常比较好的实现是具有处理一般情况的一个系统调用，而由不同的库过程向程序员隐藏这一事实。
    - 添加更多的代码就是添加更多的程序错误
    - 不要隐藏能力，如果硬件具有极其高效的方法做某事，它就应该以简单的方法展露给程序员
    - 任何面向连接的机制与无连接的机制之间的权衡在于建立连接的机制（例如打开文件）要求的额外开销，
    - 与系统调用接口有关的另一个问题是接口的可见性。 POSIX 强制的系统调用列表很容易找到。Microsoft 从未将 Windows 系统调用列表公开。作为替代，Win API 和其他 API 被公开了，但是这些 API 包含大量的库调用（超过 10000 个），只有很少数是其正的系统调用
- 实现
  - 系统结构
    - 分层系统：对于一个新系统，选择走这一路线的设计人员应该首先非常仔细地选择各个层次，井且定义每个层次的功能。底层应该总是试图隐藏硬件最糟糕的特异性
    - 外内核：他们的观点基干端到端问题，某件事情必须由用户程序本身去完成，在一个较低的层次做同样的事情就是浪费。端到端问题可以扩展到几乎所有操作系统。它主张不要让操作系统做用户程序本身可以做的任何事情。
    - 在让操作系统做每件事情和让操作系统什么也不做之间的折衷是让操作系统做一点事情。这一设计导致微内核的出现，它让操作系统的大部分作为用户级的服务器进程而运行，在所有设计中这是最模块化和最灵活的。在灵活性上的极限是让每个设备驱动程序也作为一个用户进程而运行，从而完全保护内核和其他驱动程序，但是让设备驱动程序运行在内核会增加模块化程度。
    - 可扩展的系统：将更多的模块放到内核中，但是以一种“受保护的”方式。可扩展的系统自身井不是构造一个操作系统的方法。然而，通过以一个只是包含保护机制的 朵小系统为开端，然后每次将受保护的模块添加到内核中，直到达到期望的功能，对千手边的应用而言一个最小的系统就建立起来了 。
    - 内核线程：无论选择哪种结构模型，允许存在与任何用户进程相隔离的内核线程是很方便的。这些线程可以在后台运行，将脏页面写入磁盘，在内存和磁盘之间交换进程，如此等等。实际上，内核本身可以完全由这样的线程构成，所以当一个用户发出系统调用时，用户的线程并不是在内核模式中运行，而是阻塞井且将控制传给一个内核线程，该内核线程接管控制以完成工作。
  - 机制与策略
    - 另一个有助于体系结构一致性的原理是机制与策略的分离，该原理同时还有助于使系统保持小型和良好的结构。通过将机制放入操作系统而将策略留给用户进程，即使存在改变策略的需要，系统本身也可以保持不变。即使策略模块必须保留在内核中，它也应该尽可能地与机制相隔离，这样策略模块中的变化就不会影响机制模块。
    - 对于线程调度，机制负责从高优先级到低优先级调度线程，策略负责制定线程优先级
    - 对于内存分页，机制负责把页面在内存和磁盘之间换入换出，策略绝地交换是局部的还是全局的，是 LRU 还是 FIFO
  - 正交性
    - 良好的系统设计在于单独的概念可以独立地组合。
  - 命名
    - 操作系统大多数较长使用的数据结构都具有某种名字或标识符，通过这些名字或标识符就可以引用这些数据结构。显而易见的例子有注册名、文件名、设备名、进程 ID 等。
  - 绑定的时机
    - 操作系统使用多种类型的名字来引用对象。有时在名字和对象之间的映射是固定的，但是有时不是。在后一种情况下，何时将名字与对象绑定可能是很重要的。一般而言，早期绑定(early binding) 是简单的，但是不灵活，而晚期绑定(late binding) 则比较复杂，但是通常更加灵活。
    - 操作系统对大多数数据结构通常使用早期绑定，但是偶尔为了灵活性也使用晚期绑定。内存分配是一个相关的案例。在缺乏地址重定位硬件的机器上，早期的多道程序设计系统不得不在某个内存地址装载一个程序，井且对其重定位以便在此处运行。如果它曾经被交换出去，那么它就必须装回到相同的内存地址，否则就会出错。相反，页式虚拟内存是晚期绑定的一种形式。在页面被访问并且实际装入内存之前，与一个给定的虚拟地址相对应的实际物理地址是不知道的。
  - 静态与动态结构
    - 操作系统设计人员经常被迫在静态与动态数据结构之间进行选择。静态结构总是简单易懂，更加容易编程井且用起来更快，动态结构则更加灵活。一个显而易见的例子是进程表。早期的系统只是分配一个固定的数组，存放每个进程结构。如果进程表由 256 项组成，那么在任意时刻只能存在 256 个进程。试图创建第 257 个进程将会失败，因为缺乏表空间。类似的考虑对于打开的文件表以及许多其他内核表格也是有效的。
  - 自顶向下与自底向上的实现
    - 虽然最好是自顶向下地设计系统，但是在理论上系统可以自顶向下或者自底向上地实现。在自顶向下的实现中，实现者以系统调用处理程序为开端，并且探究需要什么机制和数据结构来支持它们。接着写这些过程等，直到触及硬件。
    - 这种方法的问题是，由于只有顶层过程可用，任何事情都难于测试。出于这样的原因，许多开发入员发现实际上自底向上地构建系统更加可行。这一方法需要首先编写隐藏底层硬件的代码。中断处理程序和时钟驱动程序也是早期就需要的。
  - 同步通信与异步通信
    - 对干系统设计者，在正确编程模型的决定上是一个艰难但是很重要的问题。这场论战没有冠军。像 apache 这样的 Web 服务器坚决拥护异步通信，但是 lighnpd 等其他服务器则基干事件驱动模式。两者都非常受欢迎。在我们看来，事件相比于线程更加容易理解和调试。只要没有多核并发的需要，事件很可能是一个好的选择。
  - 实用技术
    - 隐藏硬件：许多硬件是十分麻烦的，所以只好尽早将其隐藏起来
    - 引用：通过一个索引来找到另一个实体
    - 可重用性：在略微不同的上下文中重用相同的代码通常是可行的。
    - 重入：重入指的是代码同时被执行两次或多次的能力。
    - 蛮力法：在一些代码上不值得优化或优化代价很大，就不必优化
    - 首先检查错误：系统调用可能由千各种各样的原因而执行失败：要打开的文件属于他人、因为进程表满而创建进程失败、或者因为目标进程不存在而使信号不能被发送。操作系统在执行调用之前必须无微不至地检查每一个可能的错误。
- 性能
  - 操作系统为什么运行缓慢
    - 硬件检查、初始化、功能多
  - 什么应该优化
    - 唯一的优化应该是那些显而易见要成为不可避免的问题的事情
    - 性能一旦达到一个合理的水平，榨出最后一点百分比的努力和复杂性或许并不值得
  - 空间-时间的权衡
    - 改进性能的一种一般性的方法是权衡时间与空间。在一个使用很少内存但是速度比较慢的莽法与一个使用很多内存但是速度更快的算法之间进行选择，这在计算机科学中是经常发生的事情。在做出重要的优化时，值得寻找通过使用更多内存加快了速度的算法，或者反过来通过做更多的计算节省了宝贵的内存的算法。
  - 缓存
    - 用于改进性能的一项众所周知的技术是缓存。在任何相同的结果可能需要被获取多次的情况下，缓存都是适用的。一般的方法是首先做完整的工作，然后将结果保存在缓存中。对于后来的获取结果的工作，首先要检查缓存。如果结果在缓存中，就使用它。否则、再做完整的工作。
  - 线索
    - 缓存项总是正确的。缓存搜索可能失败，但是如果找到了一项，那么这一项保证是正确的井且无需再费周折就可以使用。在某些系统中，包含线索(hint)的表是十分便利的。这些线索是关于答案的提示，但是它们并不保证是正确的。调用者必须自行对结果进行验证。
  - 利用局部性
    - 进程和程序的行为井不是随机的，它们在时间上和空间上展现出相当程度的局部性，并且可以以各种方式利用该信息来改进性能。空间局部性的一个常见例子是：进程并不是在其地址空间内部随机地到处跳转的。在一个给定的时间间隔内．它们倾向于使用数目比较少的页面。进程正在有效地使用的页面可以被标记为它的工作集，井且操作系统能够确保当进程被允许运行时，它的工作集在内存中，这样就减少了缺页的次数。
    - 局部性起作用的另一个领域是多处理器系统中的线程调度，在多处理器上一种调度线程的方法是试图在最后一次用过的 CPU 上运行每个线程，期望它的某些内存块依然还在内存的缓存中。
  - 优化常见的情况
    - 区分最常见的情况和最坏可能的情况井且分别处理它们，这通常是一个好主意。针对这两者的代码常常是相当不同的。重要的是要使常见的情况速度快。对于最坏的情况，如果它很少发生，使其正确就足够了。
- 项目管理
  - 人月神话：
    - 工作不可能完全并行化
    - 为了完全利用数目众多的程序员，工作必须划分成数目众多的模块，这样每个人才能有事情做。由于每个模块可能潜在地与每个其他模块相互作用，需要将模块－模块相互作用的数目看成随着模块数目的平方而培长，
    - 调试工作是高度序列化的
  - 团队结构
    - 任何大型项目都需要组织成层次结构。底层是许多小的团队，每个团队由首席程序员领导。在下一层，必须由一名经理人对一组团队进行协调。经验表明，你所管理的每一个人将花费你 10% 的时间，所以每组 10 个团队需要一个全职经理。这些经理也必须被管理。
  - 经验的作用
    - 拥有丰富经验的设计人员对于一个操作系统项目来说至关重要。Brooks 指出，大多数错误不是在代码中，而是在设计中。程序员正确地做了吩咐他们要做的事情，而吩咐他们要做的事情是错误的。再多测试软件都无法弥补糟糕的设计说明书。
  - 没有银弹
    - 或许在下一个十年将会看到一颗银弹，或许我们将只好满足于逐步的、渐进的改进。
- 操作系统设计的趋势
  - 虚拟化与云
  - 众核芯片
  - 大型地址空间操作系统
  - 无缝的数据访问
  - 电池供电的计算机
  - 嵌入式系统