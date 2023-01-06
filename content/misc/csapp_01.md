---
title: "深入理解计算机系统::计算机系统漫游"
date: 2021-04-15T00:17:05+08:00
draft: true
---

## 计算机系统漫游

### 信息就是位 + 上下文

- 文件分为文本文件和二进制文件。
- 8 个比特被组织成一组，称为字节。
- 系统中所有的信息：包括磁盘文件，内存中的程序、内存中存放的用户数据以及网络上传送的数据，都是由一串比特表示的。区分不同数据对象的唯一方法就是我们读到这些数据对象时的上下文呢。在不同的上下文中，一个同样的字节序列可能表示一个整数、浮点数、字符串或者机器指令。

### 程序被其他程序翻译成不同的格式

- 预处理阶段：预处理器根据以字符#开头的命令，修改原始的 C 程序。
- 编译阶段：编译器将预处理后的文本文件翻译成包含汇编语言程序的文本文件。
- 汇编阶段：汇编器将包含低汇编语言程序的文本文件翻译成机器语言指令，并将结果保存在目标文件中，目标文件是一个二进制文件。
- 链接阶段：链接器以某种方式合并程序中调用的函数预编译目标中间，最终得到一个可执行文件，可以被加载到内存中，有系统执行。

### 了解编译系统如何工作是大有益处的

- 优化程序性能
- 理解链接时出现的错误
- 避免安全漏洞

### 处理器读并解释储存在内存中的指令

#### 系统的硬件组成

- 总线
  - 贯穿整个系统的一组电子管道
  - 负责携带信息字节并负责在各个部件中传递。
  - 通常总线被设计成传递定长的字节块，称为字(word)，不同机器上字长不一致，需要具体的上下文判断。
- I/O 设备
  - I/O 设备是系统与外部世界的联系通道。
  - 每个 I/O 设备通过一个控制器或适配器与 I/O 总线相连。
    - 控制器是 I/O 设备本身或者系统的主印刷电路板上的芯片组。
    - 适配器是一块插在主板插槽上的卡。
- 主存
  - 主存是一个临时存储设备，在处理器执行程序时，用来存放程序和程序所处理的数据。
  - 从物理角度来说，主存是由一组 DRAM 芯片组成的。
  - 从逻辑上来说，存储器是一个线性的字节数据，每个字节都有其唯一的地址。
  - 组成程序的每条机器指令有不同数量的字节构成。
- 处理器
  - CPU，解释或者执行存储在内存中指令的引擎。
  - 程序计数器 PC，在任何时候，PC 都指向主存中的某条机器语言指令。
  - 寄存器文件，由一些单个字长的寄存器组成，每个寄存器都有唯一的名字。
  - ALU，算数逻辑单元，负责做算术运算。

#### 运行程序

- shell 等待输入。
- 输入命令。
- shell 读取命令，并将字符串放到内存中。
- shell 执行一系列指令来加载可执行文件，并将目标文件中的代码和数据从磁盘复制到主存。
- 处理器开始执行程序中 main 程序中的机器语言指令。
- 程序中的数据从主存加载到寄存器文件中。
- 从寄存器文件中复制到显示设备，最终显示到屏幕上。

### 高速缓存至关重要

- 处理器处理数据的速度远远超出从主存读取数据的速度，系统设计者引入了高速缓存存储器作为但是的集结区域，存放处理器近期可能会需要的信息。
- 高速缓存的局部性原理，程序具有访问局部区域里的数据和代码的趋势。

### 存储设备形成层次结构

![一个存储器层次结构的示例](https://i.imgur.com/LomKAzN.png)

### 操作系统管理硬件

- 操作系统的两个基本功能
  - 防止硬件被时空的应用程序滥用
  - 向应用程序提供简单一致的机制来控制复杂而又通常不大相同的低级硬件设备。
- 操作系统通过几个基本的抽象概念（进程、虚拟内存和文件）来实现以上功能。
  - 文件是对 I/O 设备的抽象
  - 虚拟内存是对主存和磁盘 I/O 设备的抽象表示
  - 进程是对处理器、主存和 I/O 设备的抽象表示
- 进程
  - 进程是操作系统对一个正在运行的程序的一种抽象。
  - 并发，不同的进程的指令是交错执行的。
  - 多核处理器可以并行执行多个进程。
  - 任何一个时刻，但处理器都只能执行一个进程的代码。
  - 不同进程切换交错执行的机制称为上下文切换，上下文是操作系统保持跟踪进程运行所需的所有状态信息。
- 线程
  - 一个进程由多个称为线程的执行单元组成，每个线程运行在进程的上下文中，共享一部分代码和数据。
- 虚拟内存
  - 提供一种每个进程都独占整片主存的假象。称为虚拟地址空间。
- 文件
  - 文件解释字节序列。
  - 每个 I/O 设备，包括磁盘、键盘、显示器、甚至网络都可以看成是文件。
  - 系统的所有输入输出都是通过使用一小组称为 Unix I/O 的系统函数调用读写文件来实现的。

### 系统之间利用网络通信

- 现代系统通过网络和其他系统连接在一起
- 网络可视为一个 I/O 设备

### 重要主题

- Amdahl 定律：当我们对系统的某个部分加速时，其对系统的影响取决于该部分的重要性和加速程度。
- 并发与并行
  - 线程级并行
    - 多任务快速切换
    - 多核处理器
    - 超线程技术
  - 指令级并行
    - 现代处理器可以同时执行多条指令
    - 流水线
    - 超标量操作
  - 单指令、多数据并行 SIMD
    - 一条执行产生多个可以并行执行的操作
- 抽象的重要性
  - 虚拟机