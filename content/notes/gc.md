---
title: "垃圾回收的算法与实现笔记"
date: 2021-03-20T23:19:56+08:00
draft: true
tags: ["GC"]
---

# 垃圾回收的算法与实现

## 学习 GC 之前

- GC 是 Garbage Collection 的简称，中文称为“垃圾回收”
- GC 把程序不用的内存空间视为垃圾
- GC 要做的有两件事
  - 找到内存空间里的垃圾
  - 回收垃圾，让程序员能再次利用这部分空间
- 没有 GC 的世界
  - 如果忘记释放内存空间，该内存空间就会发生内存泄露
  - 如果忘记初始化指向释放对象的内存空间的指针，导致“悬垂指针”(dangling pointer)
  - 错误释放了使用中的内存空间
- 有 GC 的世界
  - 省去了管理内存的烦恼，把精力集中在更本质的编程工作上
- GC 的历史
  - 最初的 GC 算法是 John McCarthy 在 1960 年发布的
  - GC 标记-清除算法
  - 引用计数法
  - GC 复制算法
  - 50 年来，GC 的根本都没有改变
- 对象/头/域
  - 对象表示的是“通过应用程序利用的数据的集合”
  - 一般来说，对象由头(header)和域(field)构成
    - 头
      - 对象的大小
      - 对象的种类
      - 头中事先存有运行 GC 所需的信息，根据 GC 算法的不同，信息也不同
    - 域
      - 对象使用者在对象中可访问的部分
      - 指针，指向内存空间中某块区域的值
      - 非指针，直接使用值本身，数值、字符以及真假值都是非指针
- 指针
  - 语言处理程序是否能判别指针和非指针
  - 要判别指针和非指针需要花费一定的功夫
  - 在大多数语言处理程序中，指针都默认指向对象的首地址
- mutator
  - 应用程序实体
  - 生成对象
  - 更新指针
- 堆
  - 动态存放对象的内存空间
- 活动对象 / 非活动对象
  - 分配到内存空间中的对象中那些能通过 mutator 引用的对象称为“活动对象”
  - 分配到堆中那些不能通过程序引用的对象称为“非活动对象”
  - 死掉的对象(即非活动对象)我们就称为“垃圾”
- 分配
  - 在内存空间中分配对象
  - GC 无法分配可用空间
    - 销毁至今为止的所有计算结果，输出错误信息
    - 扩大堆，分配可用空间
  - 在内存空间大小没有特殊限制的情况下，应该扩大堆
- 分块
  - 为利用对象而事先准备出来的空间
  - 初始状态下，堆被一个大的分块所占据
  - 程序会根据 mutator 的要求把这个分块分割成合适的大小，作为对象使用
  - 活动对象不久后会转化为垃圾被回收
  - 被回收的内存空间再次成为分块
  - 内存里的各个区块都重复着分块 → 活动对象 → 垃圾 → 分块 → ...... 这样的过程
- 根
  - 根是指向对象的指针的“起点”部分
- GC 性能评价标准
  - 吞吐量
    - 在单位时间内的处理能力
  - 最大暂停时间
    - 因执行 GC 而暂停执行 mutator 的最长时间
  - 堆使用效率
    - 堆中堆放的信息越多，GC 的效率也就越高，吞吐量也就随之得到改善
    - 为了执行 GC，需要把在头中堆放的信息控制在最小限度
    - 根据堆的用法，堆使用效率也会出现巨大的差异
      - GC 复制算法中将堆二等分，每次只使用一半，交替进行，因此总是只能利用堆的一半
      - GC 标记-清除算法和引用计数法就能利用整个堆
    - 堆使用效率和吞吐量，以及最大暂停时间不可兼得
  - 访问的局部性
    - 容量：辅助存储器 > 内存 > 缓存 > 寄存器
    - 速度：寄存器 > 缓存 > 内存 > 辅助存储器
    - 具有引用关系的对象安排在堆中较近的位置，就能提高在缓存中读取到想利用的数据的概率

## GC 标记-清除算法

- 定义
  - GC 标记-清除算法由标记阶段和清除阶段构成。标记阶段是把所有活动对象都做上标记的阶段。清除阶段是把那些没有标记的对象，也就是非活动对象回收的阶段。通过这两个阶段，就可以令不能利用的内存空间重新得到利用
- GC 标记-清除算法
  - 标记阶段
    - 我们首先要标记通过根直接引用的对象，然后递归地标记通过指针数组能访问到的对象。
    - 标记的标志在对象头中，为真表示被标记过，反之没有
    - 标记的时间与活动对象的数量正比
    - 深度优先标记更能压低内存使用量
  - 清除阶段
    - 在清除阶段中，collector 会遍历整个堆，回收没有打上标记的对象(即垃圾)，将其添加到空闲链表中，使其能再次得到利用
  - 分配
    - 分配是指将回收的垃圾进行再利用，搜索空闲链表并寻找大小合适的分块，如果此函数没有找到合适的分块，则会返回 NULL
    - 分配策略
      - First-fit
        - 初发现大于等于 size 的分块时就会立即返回该分块
      - Best-fit
        - 遍历空闲链表，返回大于等于 size 的最小分块
      - Worst-fit
        - 找出空闲链表中最大的分块，将其分割成 mutator 申请的大小和分割后剩余的大小，目的是将分割后剩余的分块最大化
  - 合并
    - 在清除阶段合并空闲链表中连续的分块，合成一个大分块
- 优点
  - 实现简单
  - 与保守式 GC 算法兼容
- 缺点
  - 碎片化
    - 标记清除算法在清除阶段会出现大量碎片
    - 碎片化增加 mutator 的执行负担
  - 分配速度
    - 分块不连续，每次都需要完整遍历空闲链表
  - 与 COW 不兼容
    - GC 会设置所有活动对象的标志位，这样就会频繁发生本不应该发生的复制，压迫到内存空间。
- 多个空闲链表
  - 创建分块大小不同的空闲链表，这样一来，只要按照 mutator 所申请的分块大小选择空闲链表，就能在短时间内找到符合条件的分块了
- BiBOP 法
  - 把堆分割成固定大小的块，让每个块只能配置同样大小的对象
  - 并不能完全消除碎片化
- 位图标记
  - 只收集各个对象的标志位并表格化，不跟对象一起管理。在标记的时候，不在对象的头里置位，而是在这个表格中的特定场所置位
  - 位图表格的实现方法有多种，例如散列表和树形结构
  - 在位图标记中重要的是，位图表格中位的位置要和堆里的各个对象切实对应
  - 优点
    - 与写时复制技术兼容
      - 使用位图标记是不会对对象设置标志位的，所以也不会发生无谓的复制
      - 充血位图表格会发生复制。不过，因为位图表格非常小，所以即使被复制也不会有什么大的影响。
    - 清除操作更高效
      - 用了位图表格的清除操作则把所有对象的标志位集合到一处，所以可以快速消去标志位
  - 注意点
    - 在堆为多个的情况下，一般会为每个堆都准备一个位图表格
- 延迟清除法
  - 延迟清除法(Lazy Sweep)是缩减因清除操作而导致的 mutator 最大暂停时间的方法
  - 在标记操作结束后，不一并进行清除操作，直到无法返回合适的分块再执行延迟清扫来分配块
  - 延迟清除法不是一下遍历整个堆，它只在分配时执行必要的遍历
  - 当垃圾堆和活动对象堆格子形成一种邻接的状态，程序在活动对象堆附近一时难以获得合适的块，导致效果不平衡

## 引用计数法

## GC 复制算法

## GC 标记-压缩算法

## 保守式 GC

## 分代垃圾回收

## 增量式垃圾回收

## RC Immix 算法

## Python 的垃圾回收

## V8 的垃圾回收
