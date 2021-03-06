---
title: "MIT::6.824::Introduction"
date: 2021-09-16T09:06:25+08:00
draft: false
---

## 介绍

- 什么是分布式系统
  - 多台通过网络交互的主机
  - 大型网站的存储、MapReduce、点对点共享等
  - 许多关键基础设施是分布式的
- 为什么要构建分布式系统
  - 连接在物理上隔离的主机，实现数据共享、文件共享
  - 通过并行增加容量
  - 通过复制加强容错
  - 通过隔离实现数据安全(对外提供一个有限的接口)
- 历史背景
  - 局域网(1980s)
    - DNS，email，AFS
  - 数据中心、大型网站(1990s)
    - 网页搜索、购物
  - 云计算(2000s)
    - 大数据、机器学习
  - 依然是活跃的领域(当前)
- 挑战
  - 并发部分多，交互复杂
  - 必须应对部分失败
  - 难以实现性能潜力
- 为什么要上 6.824
  - 有趣 —— 难题，强大的解决方案
  - 被真实系统使用 —— 由大型网站的兴起驱动
  - 活跃的研究领域 —— 重要的未解决问题
  - 动手实践 —— 在实验中构建真正的系统
- 焦点：这是一门关于应用程序基础设施的课程。
  - 存储
  - 计算
  - 通信
- 重要目标
  - 利用抽象隐匿分布式系统的复杂性
- 容错
  - 可用性:尽管失败，应用程序仍能取得进展
    - 复制
  - 故障恢复:恢复故障后应用程序将正常运行
    - 日志记录
    - 故障转移
    - 持久化存储
- 一致性：通用基础设施需要明确定义的行为
  - 写后读一致性
  - 副本之间的数据延迟
- 性能：可扩展的吞吐量
  - N x 服务器 -> 通过并行 CPU、磁盘、网络的 N x 总吞吐量。
  - 随着 N 的增长，扩展越来越难
  - 长尾效应
- 容错、一致性和性能是敌人
  - 强大的容错能力需要通信
    - 例如，将数据发送到备份
  - 强一致性需要通信
    - 例如，Get() 必须检查最近的 Put()。
  - 许多设计仅提供弱一致性以提高速度。
    - 例如 Get() 不返回最新的 Put()
  - 许多设计点在一致性/性能范围内是可能的！
- MapReduce
  - Google 经常需要在 TB 数据集上的进行几个小时的计算，例如建立搜索索引，或排序，或分析网络结构
  - 方便非专家编写程序构建分布式系统
  - 程序员只是定义了 Map 和 Reduce 函数 ，编写通常相当简单的顺序代码
  - MR 负责并隐藏分发的各个方面
- MapReduce 的抽象视图
  - 输入(已经)分成 M 个文件
  - ```txt
    Input1 -> Map -> a,1 b,1
    Input2 -> Map -> b,1
    Input3 -> Map -> a,1 c,1
    | | |
    | | -> Reduce -> c,1
    | -----> Reduce -> b,2
    ---------> Reduce -> a,2
    ```
  - MR 为每个输入文件调用 Map()，生成一组 k2,v2 “中间”数据，每个 Map() 调用都是一个“任务”，MR 收集给定 k2 的所有中间 v2，并将每个键 + 值传递给 Reduce 调用最终输出是一组来自 Reduce()s 的 \<k2,v3> 对
  - 统计单词
    - 输入是数千个文本文件
    - Map(k, v)
      - 把 v 拆分成单词 word
      - 对每个 word w
        - emit(w,"1")
    - Reduce(k，v)
      - emit(len(v))
- MapReduce 扩展良好：
  - N 台'worker'计算机提供 N x 吞吐量。
  - Map()s 可以并行运行，因为它们不交互。
  - Reduce()s 也是如此。
  - 因此，您可以通过购买更多计算机来获得更多吞吐量。
- MapReduce 隐藏了很多细节
  - 将应用程序代码发送到服务器
  - 跟踪完成了哪些任务
  - 将数据从 Maps 移动到 Reduces
  - 平衡服务器负载
  - 从故障中恢复
- MapReduce 的限制
  - 没有交互或状态(除了通过中间输出)
  - 没有迭代，没有多阶段流水线
  - 没有实时或流处理。
- 输入和输出存储在 GFS 集群文件系统上
  - MR 需要巨大的并行输入和输出吞吐量。
  - GFS 将文件拆分到多个服务器上，大小为 64 MB
    - Map()并行读取
    - Reduce()并行写入
  - GFS 还会在 2 或 3 个服务器上复制每个文件
  - 拥有 GFS 是 MapReduce 的一大胜利
- 什么可能会限制性能？
  - 我们关心，因为这是优化的事情。
  - 中央处理器？记忆？磁盘？网络？
  - 2004 年，作者受到网络容量的限制。
    - MR 通过网络发送什么？
      - Map 从 GFS 读取输入。
      - Reduce 读取 Map 输出。
  - 可以和输入一样大，例如用于排序。
  - reduce 将输出文件写入 GFS。
  - 在 MR 的 all-to-all shuffle 中，一半的流量通过根交换机。
  - 论文中的根交换机：100 到 200 吉比特/秒，总计
    - 1800 台机器，所以 55 兆/秒/机器。
    - 55 很小，例如远低于磁盘或 RAM 速度。
  - 今天：网络和根交换机相对于 CPU/磁盘要快得多。
- 一些细节
- 一个 coordinator，负责向 worker 分发任务并记住进度。
- coordinator 将 Map 任务交给 worker，直到所有 Maps 完成
  - 将输出(中间数据)映射到本地磁盘
  - 将每个 Reduce 任务的输出按散列映射到一个文件中
- 所有 Maps 完成后，coordinator 下发 Reduce 任务
  - 每个 Reduce 从(所有)Map worker 获取其中间输出
  - 每个 Reduce 任务在 GFS 上写入一个单独的输出文件
- MR 如何最大限度地 reduce 网络使用？
  - Coordinator 尝试在存储其输入的 GFS 服务器上运行每个 Map 任务。
    - 所有计算机都运行 GFS 和 MR 工作线程
    - 所以输入是从本地磁盘(通过 GFS)读取的，而不是通过网络读取。
  - 中间数据仅通过网络一次。
    - Map worker 写入本地磁盘。
    - Reduce worker 直接从 Map workers 读取，而不是通过 GFS。
  - 将中间数据分区为包含许多键的文件。
    - R 远小于键的数量。
    - 大型网络传输效率更高。
- MR 如何获得良好的负载平衡？
  - 如果 N-1 个服务器必须等待 1 个慢速服务器完成，则既浪费又慢。
  - 但有些任务可能需要比其他任务更长的时间。
  - 解决方案：任务比 worker 多得多。
    - coordinator 向完成先前任务的 worker 分发新任务。
    - 因此，没有任何任务如此之大，以至于会影响完成时间(希望如此)。
    - 因此，较快的服务器比较慢的服务器完成更多的任务，同时完成。
- 容错呢
  - 即如果 worker 在 MR 工作期间崩溃怎么办？
  - 我们希望对应用程序员完全隐藏故障！
  - MR 是否必须从头开始重新运行整个工作？
    - 为什么不？
  - MR 只重新运行失败的 Map()s 和 Reduce()s。
    - 假设 MR 运行 Map 两次，其中一个 Reduce 看到第一次运行的输出，
      - 另一个 Reduce 看到第二次运行的输出？
    - 正确性需要重新执行以产生完全相同的输出。
    - 所以 Map 和 Reduce 必须是纯函数：
      - 他们只被允许查看他们的参数
      - 无状态，无文件 I/O，无交互，无外部通信。
  - 如果你想允许非功能性的 Map 或 Reduce 怎么办？
    - worker 失败将需要重新执行整个工作，
      - 或者您需要创建同步的全局检查点。
- worker 崩溃恢复的详细信息
  - Map worker 崩溃
    coordinator 通知 worker 不再响应 ping
    coordinator 知道它在那个 worker 上运行了哪些 Map 任务
    这些任务的中间输出现在丢失了，必须重新创建
    coordinator 告诉其他 worker 运行这些任务
    如果 Reduces 已经获取了中间数据，则可以省略重新运行
  - reduce worker 崩溃。
    完成的任务没问题——存储在 GFS 中，带有副本。
    coordinator 重新启动其他 worker 的 worker 未完成的任务。
  - 其他故障/问题：
    - 如果 coordinator 给两个 worker 相同的 Map() 任务怎么办？
      也许 coordinator 错误地认为一名 worker 死亡。
      它只会告诉 Reduce worker 其中之一。
    - 如果 coordinator 给两个 worker 相同的 Reduce() 任务怎么办？
      他们都会尝试在 GFS 上写入相同的输出文件！
      原子 GFS 重命名可防止混合；一个完整的文件将可见。
    - 如果一个 worker 很慢——一个“落后者”怎么办？
      也许是由于 flakey 硬件。
      coordinator 启动最后几个任务的第二个副本。
    - 如果 worker 由于硬件或软件损坏而计算出错误的输出怎么办？
      太糟糕了！MR 假设“故障停止”CPU 和软件。
    - 如果 coordinator 崩溃怎么办？
      - 重跑整个任务
- 当前状态
  - 极具影响力(Hadoop、Spark 等)。
  - 可能不再在 Google 使用。
  - 被 Flume / FlumeJava 取代(参见 Chambers 等人的论文)。
  - GFS 替换为 Colossus(没有很好的描述)和 BigTable。
- 结论
  - MapReduce 一手让大集群计算流行起来。
  - 不是最有效或最灵活的。
  - 缩放良好。
  - 易于编程——隐藏故障和数据移动。
