---
title: "深入理解Java虚拟机::垃圾收集器与内存分配策略"
date: 2021-04-02T22:10:56+08:00
draft: false
---


## 垃圾收集器与内存分配策略

- 垃圾回收需要完成的三件事
  - 那些内存需要回收
  - 什么时候回收
  - 如何回收
- 当垃圾回收成为性能瓶颈时，对垃圾回收进行必要的监控和调整
- 需要回收的区域
  - 方法区
  - 内存堆
- 对象已死
  - 引用计数法
    - 在对象中添加一个引用计数器
    - 每当有一个地方引用它时，计数器值就加一
    - 当引用失效时，计数器值就减一
    - 任何时刻计数器为零的对象就是不可能再被使用的
    - 优点
      - 原理简单，判定效率也很高
    - 缺点
      - 有很多例外情况要考虑，必须要配合大量额外处理才能保证正确地工作
      - 如单纯的引用计数就很难解决对象之间相互循环引用的问题
  - 可达性分析
    - 通过一系列称为“GC Roots”的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为“引用链”（Reference Chain），如果某个对象到 GC Roots 间没有任何引用链相连，或者用图论的话来说就是从 GC Roots 到这个对象不可达时，则证明此对象是不可能再被使用的
    - GC Roots
      - 在虚拟机栈（栈帧中的本地变量表）中引用的对象
      - 在方法区中类静态属性引用的对象，譬如 Java 类的引用类型静态变量
      - 方法区中常量引用的对象，譬如字符串常量池（String Table）里的引用
      - 在本地方法栈中 JNI（即通常所说的 Native 方法）引用的对象
      - Java 虚拟机内部的引用，如基本数据类型对应的 Class 对象，一些常驻的异常对象（比如 NullPointExcepiton、OutOfMemoryError）等，还有系统类加载器
      - 所有被同步锁（synchronized 关键字）持有的对象
      - 反映 Java 虚拟机内部情况的 JMXBean、JVMTI 中注册的回调、本地代码缓存等。
      - 其他临时性加入的对象
    - 引用
      - 强引用是程序代码之中普遍存在的引用赋值
      - 软引用是用来描述一些还有用，但非必须的对象
      - 弱引用也是用来描述那些非必须对象，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生为止
      - 虚引用也称为“幽灵引用”或者“幻影引用”，它是最弱的一种引用关系。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知。
    - 回收方法区
      - 废弃的常量和不再使用的类型
      - 判断一个类不再被使用
        - 该类所有的实例都已经被回收
        - 加载该类的类加载器已经被回收
        - 该类对应的 java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法
- 垃圾回收算法
  - 分代收集理论
    - 绝大多数对象都是朝生夕灭的
    - 熬过越多次垃圾收集过程的对象就越难以消亡
    - 跨代引用相对于同代引用来说仅占极少数
  - 标记清除算法
    - 首先标记出所有需要回收的对象，在标记完成后，统一回收掉所有被标记的对象，也可以反过来，标记存活的对象，统一回收所有未被标记的对象
    - 主要缺点
      - 执行效率不稳定，标记和清除两个过程的执行效率都随对象数量增长而降低
      - 标记、清除之后会产生大量不连续的内存碎片
  - 标记复制算法
    - 它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块
    - 当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉
    - 空间浪费
    - 优化
      - 新生代中的对象有 98%熬不过第一轮收集
      - 把新生代分为一块较大的 Eden 空间和两块较小的 Survivor 空间，每次分配内存只使用 Eden 和其中一块 Survivor。发生垃圾搜集时，将 Eden 和 Survivor 中仍然存活的对象一次性复制到另外一块 Survivor 空间上，然后直接清理掉 Eden 和已用过的那块 Survivor 空间
      - HotSpot 虚拟机默认 Eden 和 Survivor 的大小比例是 8∶1
      - 当 Survivor 空间不足以容纳一次 Minor GC 之后存活的对象时，就需要依赖其他内存区域（实际上大多就是老年代）进行分配担保
  - 标记整理算法
    - 用于老年代回收
    - 标记过程仍然与标记清除算法一样
    - 整理时让所有存活的对象都向内存空间一端移动
    - STW 的影响
    - 是否移动对象需要考虑延迟和吞吐
    - 优化
      - 平时采用标记清除，当内存碎片化到影响对象分配的时候采用标记整理
- HotSpot 的算法实现细节
  - 根节点枚举
    - 根节点枚举始终必须在一个能保障一致性的快照中进行
    - 枚举根节点时是必须要停顿的, STW
    - 准确式垃圾收集
    - 用 OopMap 的数据结构来直接得到哪些地方存放着对象引用的
    - 一旦类加载动作完成，HotSpot 会把对象偏移量上的数据类型计算出来
  - 安全点
    - 只是在“特定的位置”记录了这些信息
    - 强制要求必须执行到达安全点后才能够暂停
    - 安全点位置的选取基本上是以“是否具有让程序长时间执行的特征”为标准进行选定的
    - 抢先式中断
      - 在垃圾收集发生时，系统首先把所有用户线程全部中断，如果发现有用户线程中断的地方不在安全点上，就恢复这条线程执行，让它一会再重新中断，直到跑到安全点上
    - 主动式中断
      - 当垃圾收集需要中断线程的时候，不直接对线程操作，仅仅简单地设置一个标志位，各个线程执行过程时会不停地主动去轮询这个标志，一旦发现中断标志为真时就自己在最近的安全点上主动中断挂起
  - 安全区域
    - 安全区域是指能够确保在某一段代码片段之中，引用关系不会发生变化，因此，在这个区域中任意地方开始垃圾收集都是安全的。我们也可以把安全区域看作被扩展拉伸了的安全点
    - 当用户线程执行到安全区域里面的代码时，首先会标识自己已经进入了安全区域，那样当这段时间里虚拟机要发起垃圾收集时就不必去管这些已声明自己在安全区域内的线程了。当线程要离开安全区域时，它要检查虚拟机是否已经完成了根节点枚举(或者垃圾收集过程中其他需要暂停用户线程的阶段)，如果完成了，那线程就当作没事发生过，继续执行;否则它就必须一直等待，直到收到可以离开安全区域的信号为止。
  - 记忆集与卡表
    - 解决对象跨代引用所带来的问题
    - 记忆集是一种用于记录从非收集区域指向收集区域的指针集合的抽象数据结构
    - 卡表
      - 字节数组 CARD_TABLE 的每一个元素都对应着其标识的内存区域中一块特定大小的内存块，这个内存块被称作“卡页”(Card Page)
      - 一个卡页的内存中通常包含不止一个对象，只要卡页内有一个(或更多)对象的字段存在着跨代指针，那就将对应卡表的数组元素的值标识为 1，称为这个元素变脏(Dirty)，没有则标识为 0。在垃圾收集发生时，只要筛选出卡表中变脏的元素，就能轻易得出哪些卡页内存块中包含跨代指针，把它们加入 GC Roots 中一并扫描。
  - 写屏障
    - 解决卡表元素如何维护的问题
    - 写屏障可以看作在虚拟机层面对“引用类型字段赋值”这个动作的 AOP 切面，在引用对象赋值时会产生一个环形(Around)通知，供程序执行额外的动作，也就是说赋值的前后都在写屏障的覆盖范畴内
    - 应用写屏障后，虚拟机就会为所有赋值操作生成相应的指令
    - 伪共享问题
  - 并发的可达性问题
    - 从 GC Roots 再继续往下遍历对象图，这一步骤的停顿时间就必定会与 Java 堆容量直接成正比例关系了
    - 三色标记
      - 白色:表示对象尚未被垃圾收集器访问过。显然在可达性分析刚刚开始的阶段，所有的对象都是白色的，若在分析结束的阶段，仍然是白色的对象，即代表不可达。
      - 黑色:表示对象已经被垃圾收集器访问过，且这个对象的所有引用都已经扫描过。黑色的对象代表已经扫描过，它是安全存活的，如果有其他对象引用指向了黑色对象，无须重新扫描一遍。黑色对象不可能直接(不经过灰色对象)指向某个白色对象。
      - 灰色:表示对象已经被垃圾收集器访问过，但这个对象上至少存在一个引用还没有被扫描过
    - 对象消失的问题
      - 赋值器插入了一条或多条从黑色对象到白色对象的新引用
      - 赋值器删除了全部从灰色对象到该白色对象的直接或间接引用
    - 增量更新
      - 当黑色对象插入新的指向白色对象的引用关系时，就将这个新插入的引用记录下来，等并发扫描结束之后，再将这些记录过的引用关系中的黑色对象为根，重新扫描一次。这可以简化理解为，黑色对象一旦新插入了指向白色对象的引用之后，它就变回灰色对象了。
    - 原始快照
      - 当灰色对象要删除指向白色对象的引用关系时，就将这个要删除的引用记录下来，在并发扫描结束之后，再将这些记录过的引用关系中的灰色对象为根，重新扫描一次。这也可以简化理解为，无论引用关系删除与否，都会按照刚刚开始扫描那一刻的对象图快照来进行搜索。
- 经典垃圾回收器
  - Serial 收集器
    - 使用一个处理器或一条收集线程去完成垃圾收集工作
    - 额外内存消耗最小
    - 简单而高效
    - Serial 收集器对于运行在客户端模式下的虚拟机来说是一个很好的选择
  - ParNew 收集器
    - Serial 收集器的多线程并行版本
    - Parallel Scavenge 收集器及 G1 收集器等都没有使用 HotSpot 中原本设计的垃圾收集器的分代框架，而选择另外独立实现,CMS 只能和 ParNew 搭配使用
  - Parallel Scavenge 收集器
    - 基于标记-复制算法实现的收集器
    - 达到一个可控制的吞吐量(Throughput)。所谓吞吐量就是处理器用于运行用户代码的时间与处理器总消耗时间的比值
    - 最高效率地利用处理器资源
  - Serial Old 收集器
    - Serial Old 是 Serial 收集器的老年代版本，它同样是一个单线程收集器
    - 使用标记-整理算法
    - 供客户端模式下的 HotSpot 虚拟机使用
  - Parallel Old 收集器
    - Parallel Old 是 Parallel Scavenge 收集器的老年代版本，支持多线程并发收集
    - 基于标记-整理算法实现
    - 注重吞吐量或者处理器资源较为稀缺的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器这个组合
  - CMS 收集器
    - 获取最短回收停顿时间为目标的收集器
    - 标记-清除算法
      - 初始标记(CMS initial mark)
        - 仅仅只是标记一下 GC Roots 能直接关联到的对象
      - 并发标记(CMS concurrent mark)
        - 从 GC Roots 的直接关联对象开始遍历整个对象图的过程，这个过程耗时较长但是不需要停顿用户线程
      - 重新标记(CMS remark)
        - 修正并发标记期间，因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录
      - 并发清除(CMS concurrent sweep)
        - 清理删除掉标记阶段判断的已经死亡的对象，由于不需要移动存活对象，所以这个阶段也是可以与用户线程同时并发的
    - 其中初始标记、重新标记这两个步骤仍然需要“Stop The World”
    - 缺点
      - 对处理器资源非常敏感
        - CMS 默认启动的回收线程数是(处理器核心数量 +3)/4
      - CMS 收集器无法处理“浮动垃圾”
        - CMS 的并发标记和并发清理阶段，用户线程是还在继续运行的，程序在运行自然就还会伴随有新的垃圾对象不断产生，但这一部分垃圾对象是出现在标记过程结束以后，CMS 无法在当次收集中处理掉它们，只好留待下一次垃圾收集时再清理掉
      - 空间碎片
  - G1 收集器
    - 开创了收集器面向局部收集的设计思路和基于 Region 的内存布局形式
    - 提供并发的类卸载的支持
    - 主要面向服务端应用的垃圾收集器
    - JDK9 的默认收集器
    - 面向堆内存任何部分来组成回收集(Collection Set，一般简称 CSet)进行回收，衡量标准不再是它属于哪个分代，而是哪块内存中存放的垃圾数量最多，回收收益最大，这就是 G1 收集器的 Mixed GC 模式
    - 基于 Region 的堆内存布局
    - G1 不再坚持固定大小以及固定数量的分代区域划分，而是把连续的 Java 堆划分为多个大小相等的独立区域(Region)，每一个 Region 都可以根据需要，扮演新生代的 Eden 空间、Survivor 空间，或者老年代空间。收集器能够对扮演不同角色的 Region 采用不同的策略去处理，这样无论是新创建的对象还是已经存活了一段时间、熬过多次收集的旧对象都能获取很好的收集效果
    - Region 中还有一类特殊的 Humongous 区域，专门用来存储大对象。G1 认为只要大小超过了一个 Region 容量一半的对象即可判定为大对象
    - 对于那些超过了整个 Region 容量的超级大对象， 将会被存放在 N 个连续的 Humongous Region 之中
    - 使用 Region 划分内存空间，以及具有优先级的区域回收方式，保证了 G1 收集器在有限的时间内获取尽可能高的收集效率
    - Region 里面存在的跨 Region 引用对象如何解决
      - 使用记忆集避免全堆作为 GC Roots 扫描，但在 G1 收集器上记忆集的应用其实要复杂很多，它的每个 Region 都维护有自己的记忆集，这些记忆集会记录下别的 Region 指向自己的指针，并标记这些指针分别在哪些卡页的范围之内
      - 根据经验，G1 至少要耗费大约相当于 Java 堆容量 10%至 20%的额外内存来维持收集器工作
    - 并发标记阶段如何保证收集线程与用户线程互不干扰地运行
      - 原始快照算法
      - 如果内存回收的速度赶不上内存分配的速度， G1 收集器也要被迫冻结用户线程执行，导致 Full GC 而产生长时间“Stop The World”
    - 怎样建立起可靠的停顿预测模型
      - G1 收集器的停顿预测模型是以衰减均值(Decaying Average)为理论基础来实现的，在垃圾收集过程中，G1 收集器会记录每个 Region 的回收耗时、每个 Region 记忆集里的脏卡数量等各个可测量的步骤花费的成本，并分析得出平均值、标准偏差、置信度等统计信息
    - 收集过程
      - 初始标记(Initial Marking):仅仅只是标记一下 GC Roots 能直接关联到的对象，并且修改 TAMS 指针的值，让下一阶段用户线程并发运行时，能正确地在可用的 Region 中分配新对象。这个阶段需要停顿线程，但耗时很短，而且是借用进行 Minor GC 的时候同步完成的，所以 G1 收集器在这个阶段实际并没有额外的停顿。
      - 并发标记(Concurrent Marking):从 GC Root 开始对堆中对象进行可达性分析，递归扫描整个堆里的对象图，找出要回收的对象，这阶段耗时较长，但可与用户程序并发执行。当对象图扫描完成以后，还要重新处理 SATB 记录下的在并发时有引用变动的对象。
      - 最终标记(Final Marking):对用户线程做另一个短暂的暂停，用于处理并发阶段结束后仍遗留下来的最后那少量的 SATB 记录。
      - 筛选回收(Live Data Counting and Evacuation):负责更新 Region 的统计数据，对各个 Region 的回收价值和成本进行排序，根据用户所期望的停顿时间来制定回收计划，可以自由选择任意多个 Region 构成回收集，然后把决定回收的那一部分 Region 的存活对象复制到空的 Region 中，再清理掉整个旧 Region 的全部空间。这里的操作涉及存活对象的移动，是必须暂停用户线程，由多条收集器线程并行完成的。
    - G1 收集器除了并发标记外，其余阶段也是要完全暂停用户线程的
    - 最先进的垃圾收集器的设计导向都不约而同地变为追求能够应付应用的内存分配速率 (Allocation Rate)，而不追求一次把整个 Java 堆全部清理干净
    - G1 垃圾收集产生的内存占用(Footprint)和程序运行时的额外执行负载 (Overload)都要比 CMS 要高
      - G1 的记忆集(和其他内存消耗)可能会占整个堆容量的 20%乃至更多的内存空间
      - G1 对写屏障的复杂操作要比 CMS 消耗更多的运算资源
      - 目前在小内存应用上 CMS 的表现大概率仍然要会优于 G1，而在大内存应用上 G1 则大多能发挥其优势，这个优劣势的 Java 堆容量平衡点通常在 6GB 至 8GB 之间
- 低延迟垃圾收集器
  - ZGC 收集器
    - 在尽可能对吞吐量影响不太大的前提下，把垃圾收集的停顿时间限制在十毫秒以内的低延迟
    - ZGC 收集器是一款基于 Region 内存布局的，(暂时) 不设分代的，使用了读屏障、染色指针和内存多重映射等技术来实现可并发的标记-整理算法的，以低延迟为首要目标的一款垃圾收集器
    - ZGC 的 Region 具有动态性——动态创建和销毁，以及动态的区域容量大小
      - 小型 Region(Small Region):容量固定为 2MB，用于放置小于 256KB 的小对象。
      - 中型 Region(Medium Region):容量固定为 32MB，用于放置大于等于 256KB 但小于 4MB 的对象。
      - 大型 Region(Large Region):容量不固定，可以动态变化，但必须为 2MB 的整数倍，用于放置 4MB 或以上的大对象
      - ZGC 收集器有一个标志性的设计是它采用的染色指针技术
        - 染色指针是一种直接将少量额外的信息存储在指针上的技术
        - ZGC 用地址的高 4 位提取出来存储四个标志信息
          - 其引用对象的三色标记状态
          - 是否进入了重分配集(即被移动过)
          - 是否只能通过 finalize()方法才能被访问到
        - 染色指针可以大幅减少在垃圾收集过程中内存屏障的使用数量，设置内存屏障，尤其是写屏障的目的通常是为了记录对象引用的变动情况，如果将这些信息直接维护在指针中，显然就可以省去一些专门的记录操作
        - ZGC 只使用了读屏障
        - ZGC 使用了多重映射(Multi-Mapping)将多个不同的虚拟内存地址映射到同一个物理内存地址上
    - 收集过程
      - 并发标记：标记阶段会更新染色指针中的 Marked0、Marked1 标志位
      - 并发预备重分配: 根据特定的查询条件统计得出本次收集过程要清理哪些 Region，将这些 Region 组成重分配集(Relocation Set)
      - 并发重分配(Concurrent Relocate):重分配是 ZGC 执行过程中的核心阶段，这个过程要把重分配集中的存活对象复制到新的 Region 上，并为重分配集中的每个 Region 维护一个转发表(Forward Table)，记录从旧对象到新对象的转向关系
      - 并发重映射(Concurrent Remap):重映射所做的就是修正整个堆中指向重分配集中旧对象的所有引用
- 选择合适的垃圾收集器
  - 如果是数据分析、科学计算类的任务，目标是能尽快算出结果，那吞吐量就是主要关注点
  - 如果是 SLA 应用，那停顿时间直接影响服务质量，严重的甚至会导致事务超时，这样延迟就是主要关注点
  - 如果是客户端应用或者嵌入式应用，那垃圾收集的内存占用则是不可忽视的