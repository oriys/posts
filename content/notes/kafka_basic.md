---
title: "Kafka::基础"
date: 2021-08-11T03:52:05+08:00
draft: true
---

## 入门

- 定义
  - Apache Kafka 是一款开源的消息引擎系统。根据维基百科的定义，消息引擎系统是一组规范。企业利用这组规范在不同系统之间传递语义准确的消息，实现松耦合的异步式数据传递。
- 特点
  - 消息引擎传输的对象是消息
  - 传输消息属于消息引擎设计机制的一部分
- 消息编码格式
  - CSV
  - XML
  - JSON
  - Google 的 Protocol Buffer
  - Facebook 的 Thrift
  - 结构化的二进制字节序列
- 消息传输协议
  - 点对点模型
  - 发布/订阅模型
- 功能
  - 削峰填谷：缓冲上下游瞬时突发流量，使其更平滑过度给下游
  - 发送方和接收方的松耦合：简化应用开发，减少了系统不必要的交互
- 术语
  - 消息：Record。Kafka 是消息引擎嘛，这里的消息就是指 Kafka 处理的主要对象。
  - 主题：Topic。主题是承载消息的逻辑容器，在实际使用中多用来区分具体的业务。
  - 分区：Partition。一个有序不变的消息序列。每个主题下可以有多个分区。
  - 消息位移：Offset。表示分区中每条消息的位置信息，是一个单调递增且不变的值。
  - 副本：Replica。Kafka 中同一条消息能够被拷贝到多个地方以提供数据冗余，这些地方就是所谓的副本。副本还分为领导者副本和追随者副本，各自有不同的角色划分。副本是在分区层级下的，即每个分区可配置多个副本实现高可用。
  - 生产者：Producer。向主题发布新消息的应用程序。
  - 消费者：Consumer。从主题订阅新消息的应用程序。
  - 消费者位移：Consumer Offset。表征消费者消费进度，每个消费者都有自己的消费者位移。
  - 消费者组：Consumer Group。多个消费者实例共同组成的一个组，同时消费多个分区以实现高吞吐。
  - 重平衡：Rebalance。消费者组内某个消费者实例挂掉后，其他消费者实例自动重新分配订阅主题分区的过程。Rebalance 是 Kafka 消费者端实现高可用的重要手段。
- 分布式流处理平台
  - 研发背景
    - 数据正确性不足，Polling 导致数据偏差
    - 系统高度定制化，维护成本高
  - 特性
    - 提供一套 API 实现生产者和消费者
    - 降低网络传输和磁盘存储开销
    - 实现高伸缩性架构
  - 优势
    - 更容易实现端到端的正确性，Kafka 可以实现端到端的精确一次处理语义
    - 更开放的流式计算定位，Kafka Streams 是一个用于搭建实时流处理的客户端库而非是一个完整的功能系统
    - 用作分布式存储
- 版本选择
  - Apache Kafka，也称社区版 Kafka。优势在于迭代速度快，社区响应度高，使用它可以让你有更高的把控度；缺陷在于仅提供基础核心组件，缺失一些高级的特性。
  - Confluent Kafka，Confluent 公司提供的 Kafka。优势在于集成了很多高级特性且由 Kafka 原班人马打造，质量上有保证；缺陷在于相关文档资料不全，普及率较低，没有太多可供参考的范例。
  - CDH/HDP Kafka，大数据云公司提供的 Kafka，内嵌 Apache Kafka。优势在于操作简单，节省运维成本；缺陷在于把控度低，演进速度较慢。

## 基本使用

- 部署环境
  - 操作系统
    - Linux：高效的 I/O 性能、Linux 实现了零拷贝的数据传输、Kafka 更重视 Linux 社区
  - 磁盘
    - 追求性价比的公司可以不搭建 RAID，使用普通磁盘组成存储空间即可
    - 使用机械磁盘完全能够胜任 Kafka 线上环境
  - 磁盘容量
    - 新增消息数
    - 消息留存时间
    - 平均消息大小
    - 备份数
    - 是否启用压缩
  - 带宽
    - 预留三分一空闲带宽
- 重要的配置
  - Broker 端参数
    - log.dirs: Broker 需要使用的若干个文件目录路径，逗号分隔的多个路径
    - log.dir: 表示单个路径，它是补充上一个参数用的
    - zookeeper.connect: 负责协调管理并保存 Kafka 集群的所有元数据信息，逗号分隔，chroot 只写一次，添加到最后
    - listeners：学名叫监听器，其实就是告诉外部连接者要通过什么协议访问指定主机名和端口开放的 Kafka 服务。
    - advertised.listeners：和 listeners 相比多了个 advertised。Advertised 的含义表示宣称的、公布的，就是说这组监听器是 Broker 用于对外发布的。
    - listener.security.protocol.map：自定义协议声明
    - auto.create.topics.enable：是否允许自动创建 Topic。
    - unclean.leader.election.enable：是否允许 Unclean Leader 选举。
    - auto.leader.rebalance.enable：是否允许定期进行 Leader 选举。
    - log.retention.{hour|minutes|ms}：控制一条消息数据被保存多长时间。从优先级上来说 ms 设置最高、minutes 次之、hour 最低。
    - log.retention.bytes：这是指定 Broker 为消息保存的总磁盘容量大小，-1 表示不限制。
    - message.max.bytes：控制 Broker 能够接收的最大消息大小。
  - Topic 端参数
    - retention.ms：规定了该 Topic 消息被保存的时长
    - retention.bytes：规定了要为该 Topic 预留多大的磁盘空间
  - JVM 参数
    - -XX:+UseParallelGC：启用 CMS 收集器
    - KAFKA_HEAP_OPTS：指定堆大小。
    - KAFKA_JVM_PERFORMANCE_OPTS：指定 GC 参数。
  - 操作系统参数
    - 文件描述符限制：ulimit -n，每次连接会打开一个文件，可以设置大一点
    - 文件系统选择：XFS 性能强于 ext3、ext4
    - SWAP：预留一些以免耗尽内存触发系统 OOM killer
    - 调整 Flush 落盘时间：已经有了副本冗余，可以延长一点，换取性能

## 实践与原理

- 生产者消息分区原理
  - 主题下的每条消息只会保存在某一个分区中，而不会在多个分区中被保存多份。
  - 分区的作用就是提供负载均衡的能力，实现系统的高伸缩性
  - 分区策略
    - 轮询策略：轮询策略有非常优秀的负载均衡表现，它总是能保证消息最大限度地被平均分配到所有分区上，故默认情况下它是最合理的分区策略
    - 随机策略：从实际表现来看，它要逊于轮询策略，所以如果追求数据的均匀分布，还是使用轮询策略比较好
    - Key-ordering 策略：Kafka 允许为每条消息定义消息键，简称为 Key。这个 Key 的作用非常大，它可以是一个有着明确业务含义的字符串，比如客户代码、部门编号或是业务 ID 等；也可以用来表征消息元数据。特别是在 Kafka 不支持时间戳的年代，在一些场景中，工程师们都是直接将消息创建时间封装进 Key 里面的。一旦消息被定义了 Key，那么你就可以保证同一个 Key 的所有消息都进入到相同的分区里面，由于每个分区下的消息处理都是有顺序的，故这个策略被称为 Key-Ordering 保序策略
    - 其他分区策略：基于地理位置的分区等
- 生产者压缩算法
  - V2 版本的 Kafka 提取了消息中的公共部分到外面消息集合里，对整个消息集合进行压缩，V1 版本压缩消息队列后再放入消息集合中，获得了更好的压缩效果。
  - 何时压缩
    - 在生产端和 Broker 端都可以压缩，一般在生产端压缩
    - Broker 端压缩的场景：
      - Broker 端指定了和 Producer 端不同的压缩算法
      - Broker 端发生了消息格式转换：出于兼容不同版本消费者程序，丧失了零拷贝抽象
  - 何时解压缩
    - 在 Consumer 和 Broker 都可以解压缩，每个压缩过的消息集合在 Broker 端写入时都要发生解压缩操作，目的就是为了对消息执行各种验证
  - 压缩算法
    - 根据压缩比和压缩/解压吞吐量来选择压缩算法
    - 如果 Producer 端服务器 CPU 资源充足，可以在 Producer 端开启压缩
    - 尽量避免解压缩操作
- 无消息丢失配置
  - Kafka 只对“已提交”的消息（committed message）做有限度的持久化保证。
    - 当 Kafka 的若干个 Broker 成功地接收到一条消息并写入到日志文件后，它们会告诉生产者程序这条消息已成功提交。此时，这条消息在 Kafka 看来就正式变为“已提交”消息了。
    - 消息保存在 N 个 Kafka Broker 上，那么这个前提条件就是这 N 个 Broker 中至少有 1 个存活。只要这个条件成立，Kafka 就能保证你的这条消息永远不会丢失。
  - 消息丢失的场景
    - 异步发送消息：丢包，队列阻塞，Broker 因为格式错误不接受，改正方法采用 callback
    - 消费者丢失数据：先更新位移再消费，中间中断了消费，重新消费时从位移点开始，丢失了一部分数据。维持先消费消息（阅读），再更新位移（书签）的顺序即可。这样就能最大限度地保证消息不丢失。
    - 多线程消费更新位移丢失消息。如果是多线程异步处理消费消息，Consumer 程序不要开启自动提交位移，而是要应用程序手动提交位移。
  - 最佳实践
    - 不要使用 producer.send(msg)，而要使用 producer.send(msg, callback)。记住，一定要使用带有回调通知的 send 方法。
    - 设置 acks = all。acks 是 Producer 的一个参数，代表了你对“已提交”消息的定义。如果设置成 all，则表明所有副本 Broker 都要接收到消息，该消息才算是“已提交”。这是最高等级的“已提交”定义。
    - 设置 retries 为一个较大的值。这里的 retries 同样是 Producer 的参数，对应前面提到的 Producer 自动重试。当出现网络的瞬时抖动时，消息发送可能会失败，此时配置了 retries > 0 的 Producer 能够自动重试消息发送，避免消息丢失。
    - 设置 unclean.leader.election.enable = false。这是 Broker 端的参数，它控制的是哪些 Broker 有资格竞选分区的 Leader。如果一个 Broker 落后原先的 Leader 太多，那么它一旦成为新的 Leader，必然会造成消息的丢失。故一般都要将该参数设置成 false，即不允许这种情况的发生。
    - 设置 replication.factor >= 3。这也是 Broker 端的参数。其实这里想表述的是，最好将消息多保存几份，毕竟目前防止消息丢失的主要机制就是冗余。
    - 设置 min.insync.replicas > 1。这依然是 Broker 端参数，控制的是消息至少要被写入到多少个副本才算是“已提交”。设置成大于 1 可以提升消息持久性。在实际环境中千万不要使用默认值 1。
    - 确保 replication.factor > min.insync.replicas。如果两者相等，那么只要有一个副本挂机，整个分区就无法正常工作了。我们不仅要改善消息的持久性，防止数据丢失，还要在不降低可用性的基础上完成。推荐设置成 replication.factor = min.insync.replicas + 1。
    - 确保消息消费完成再提交。Consumer 端有个参数 enable.auto.commit，最好把它设置成 false，并采用手动提交位移的方式。就像前面说的，这对于单 Consumer 多线程处理的场景而言是至关重要的。
- Kafka 拦截器
  - 可以在消息处理的前后多个时点动态植入不同的处理逻辑，比如在消息发送前或者在消息被消费后。
  - Kafka 拦截器分为生产者拦截器和消费者拦截器。生产者拦截器允许你在发送消息前以及消息提交成功后植入你的拦截器逻辑；而消费者拦截器支持在消费消息前以及提交位移后编写特定逻辑。
  - 生产者拦截器继承`org.apache.kafka.clients.producer.ProducerInterceptor`
    - `onSend`：该方法会在消息发送之前被调用
    - `onAcknowledgement`：该方法会在消息成功提交或发送失败之后被调用
  - 消费者拦截器继承`org.apache.kafka.clients.consumer.ConsumerInterceptor`
    - `onConsume` ：该方法在消息返回给 Consumer 程序之前调用。
    - `onCommit`：Consumer 在提交位移之后调用该方法。
  - 场景
    - 客户端监控
    - 端到端系统性能检测
    - 消息审计
- 生产者 TCP 连接管理
  - KafkaProducer 实例创建时启动 Sender 线程，从而创建与 bootstrap.servers 中所有 Broker 的 TCP 连接。
  - KafkaProducer 实例首次更新元数据信息之后，还会再次创建与集群中所有 Broker 的 TCP 连接。
  - 如果 Producer 端发送消息到某台 Broker 时发现没有与该 Broker 的 TCP 连接，那么也会立即创建连接。
  - 如果设置 Producer 端 connections.max.idle.ms 参数大于 0，则步骤 1 中创建的 TCP 连接会被自动关闭；如果设置该参数 =-1，那么步骤 1 中创建的 TCP 连接将无法被关闭，从而成为“僵尸”连接。
- 可靠性保障
  - 处理消息常见承诺
    - 最多一次（at most once）：消息可能会丢失，但绝不会被重复发送。
    - 至少一次（at least once）：消息不会丢失，但有可能被重复发送。
    - 精确一次（exactly once）：消息不会丢失，也不会被重复发送。
  - 幂等 Producer:enable.idempotence 被设置成 true 后，Producer 自动升级成幂等性 Producer，其他所有的代码逻辑都不需要改变。Kafka 自动帮你做消息的重复去重。底层具体的原理很简单，就是经典的用空间去换时间的优化思路，即在 Broker 端多保存一些字段。当 Producer 发送了具有相同字段值的消息后，Broker 能够自动知晓这些消息已经重复了，于是可以在后台默默地把它们“丢弃”掉。
  - 事务 Producer：和普通 Producer 代码相比，事务型 Producer 的显著特点是调用了一些事务 API，如 initTransaction、beginTransaction、commitTransaction 和 abortTransaction，它们分别对应事务的初始化、事务开始、事务提交以及事务终止。在 Consumer 端设置事务级别 read_committed，它只处理事务型 Producer 写入的消息。
  - 幂等性 Producer 只能保证单分区、单会话上的消息幂等性；而事务能够保证跨分区、跨会话间的幂等性。从交付语义上来看，自然是事务型 Producer 能做的更多，但是性能更差。
- 消费者组
  - Consumer Group 是 Kafka 提供的可扩展且具有容错性的消费者机制
  - Consumer Group 下可以有一个或多个 Consumer 实例。这里的实例可以是一个单独的进程，也可以是同一进程下的线程。在实际场景中，使用进程更为常见一些。
  - Group ID 是一个字符串，在一个 Kafka 集群中，它标识唯一的一个 Consumer Group。
  - Consumer Group 下所有实例订阅的主题的单个分区，只能分配给组内的某个 Consumer 实例消费。这个分区当然也可以被其他的 Group 消费。
  - 理想情况下，Consumer 实例的数量应该等于该 Group 订阅主题的分区总数。
- 位移主题
  - 新版本 Consumer 的位移管理机制将 Consumer 的位移数据作为一条条普通的 Kafka 消息，提交到 \_\_consumer_offsets 中。可以这么说，\_\_consumer_offsets 的主要作用是保存 Kafka 消费者的位移信息。
  - 位移主题的消息格式却是 Kafka 自己定义的
  - 位移主题的 Key 中应该保存 3 部分内容：\<Group ID，主题名，分区号\>
  - 当 Kafka 集群中的第一个 Consumer 程序启动时，Kafka 会自动创建位移主题。
  - 如果位移主题是 Kafka 自动创建的，那么该主题的分区数是 50，副本数是 3。
  - Consumer 提交位移的方式有两种：自动提交位移和手动提交位移。
  - Kafka 使用 Compact 策略来删除位移主题中的过期消息，避免该主题无限期膨胀。
  - Kafka 提供了专门的后台线程定期地巡检待 Compact 的主题，看看是否存在满足条件的可删除数据。这个后台线程叫 Log Cleaner。
- 重平衡
  - Rebalance 就是让一个 Consumer Group 下所有的 Consumer 实例就如何消费订阅主题的所有分区达成共识的过程。
  - 在 Rebalance 过程中，所有 Consumer 实例共同参与，在协调者组件的帮助下，完成订阅主题分区的分配。但是，在整个过程中，所有实例都不能消费任何消息，因此它对 Consumer 的 TPS 影响很大。
  - 缺点
    - Rebalance 影响 Consumer 端 TPS。
    - Rebalance 很慢。
    - Rebalance 效率不高。
  - Rebalance 发生时机
    - 组成员数量发生变化
    - 订阅主题数量发生变化
    - 订阅主题的分区数发生变化
  - 避免 Rebalance
    - 第一类非必要 Rebalance 是因为未能及时发送心跳，导致 Consumer 被“踢出”Group 而引发的
    - 第二类非必要 Rebalance 是 Consumer 消费时间过长导致的
- 位移提交
  - Consumer 需要向 Kafka 汇报自己的位移数据，这个汇报过程被称为提交位移
  - Consumer 需要为分配给它的每个分区提交各自的位移数据。
  - 位移提交的语义保障是由开发者来负责的，Kafka 只会“无脑”地接受你提交的位移
  - 从用户的角度来说，位移提交分为自动提交和手动提交；从 Consumer 端的角度来说，位移提交分为同步提交和异步提交。
  - 手动提交也无法完全避免重复消费
- CommitFailedException
  - 所谓 CommitFailedException，顾名思义就是 Consumer 客户端在提交位移时出现了错误或异常，而且还是那种不可恢复的严重异常
  - 优化方法
    - 缩短单条消息处理的时间
    - 增加 Consumer 端允许下游系统消费一批消息的最大时长
    - 减少下游系统一次性消费的消息总数
    - 下游系统使用多线程来加速消费
- 消费者 TCP 管理
  - 构建 KafkaConsumer 实例时是不会创建任何 TCP 连接的
  - TCP 连接是在调用 KafkaConsumer.poll 方法时被创建的
    - 发起 FindCoordinator 请求时
    - 连接协调者时
    - 消费数据时
  - 3 类 TCP 连接
    - 确定协调者和获取集群元数据。
    - 连接协调者，令其执行组成员管理操作。
    - 执行实际的消息获取。
- 消费进度监控
  - 对于 Kafka 消费者来说，最重要的事情就是监控它们的消费进度了，或者说是监控它们消费的滞后程度。这个滞后程度有个专门的名称：消费者 Lag 或 Consumer Lag。
  - 在实际业务场景中必须时刻关注消费者的消费进度。一旦出现 Lag 逐步增加的趋势，一定要定位问题，及时处理，避免造成业务损失。
  - 几种监控方式
    - 使用 Kafka 自带的命令行工具 kafka-consumer-groups 脚本。
    - 使用 Kafka Java Consumer API 编程。
    - 使用 Kafka 自带的 JMX 监控指标。

## Kafka 内核

- 副本机制
  - 好处
    - 提供数据冗余。即使系统部分组件失效，系统依然能够继续运转，因而增加了整体可用性以及数据持久性。
    - 提供高伸缩性。支持横向扩展，能够通过增加机器的方式来提升读性能，进而提高读操作吞吐量。
    - 改善数据局部性。允许将数据放入与用户地理位置相近的地方，从而降低系统延时。
  - 定义
    - 所谓副本（Replica），本质就是一个只能追加写消息的提交日志
  - 数据一致性
    - 在 Kafka 中，副本分成两类：领导者副本（Leader Replica）和追随者副本（Follower Replica）。每个分区在创建时都要选举一个副本，称为领导者副本，其余的副本自动称为追随者副本。
    - 追随者副本是不对外提供服务，它唯一的任务就是从领导者副本异步拉取消息，并写入到自己的提交日志中，从而实现与领导者副本的同步。
    - 当领导者副本挂掉了，或者说领导者副本所在的 Broker 宕机时，Kafka 依托于 ZooKeeper 提供的监控功能能够实时感知到，并立即开启新一轮的领导者选举，从追随者副本中选一个作为新的领导者。老 Leader 副本重启回来后，只能作为追随者副本加入到集群中。
  - In-sync Replicas（ISR）
    - ISR 副本集合。ISR 中的副本都是与 Leader 同步的副本，相反，不在 ISR 中的追随者副本就被认为是与 Leader 不同步的
    - Leader 副本天然就在 ISR 中。也就是说，ISR 不只是追随者副本集合，它必然包括 Leader 副本。甚至在某些情况下，ISR 只有 Leader 这一个副本。
    - 允许 Follower 副本在一段时间内落后 Leader 副本
  - Unclean 领导者选举
    - Kafka 把所有不在 ISR 中的存活副本都称为非同步副本。通常来说，非同步副本落后 Leader 太多，因此，如果选择这些副本作为新 Leader，就可能出现数据的丢失。毕竟，这些副本中保存的消息远远落后于老 Leader 中的消息。在 Kafka 中，选举这种副本的过程称为 Unclean 领导者选举。Broker 端参数 unclean.leader.election.enable 控制是否允许 Unclean 领导者选举。
- 请求处理
  - 所有的请求都是通过 TCP 网络以 Socket 的方式进行通讯的。
  - Reactor 模式
    - Reactor 模式是事件驱动架构的一种实现方式，特别适合应用于处理多个客户端并发向服务器端发送请求的场景
    - Acceptor，它会将不同的请求下发到多个工作线程中处理。只是用于请求分发，不涉及具体的逻辑处理，非常得轻量级，因此有很高的吞吐量表现。而这些工作线程可以根据实际业务处理需要任意增减，从而动态调节系统负载能力。
    - 网络线程池：专门处理客户端发送的请求。Acceptor 线程采用轮询的方式将入站请求公平地发到所有网络线程中，因此，在实际使用过程中，这些线程通常都有相同的几率被分配到待处理请求。
    - IO 线程池：当网络线程拿到请求后，它不是自己处理，而是将请求放入到一个共享请求队列中。Broker 端还有个 IO 线程池，负责从该队列中取出请求，执行真正的处理。如果是 PRODUCE 生产请求，则将消息写入到底层的磁盘日志中；如果是 FETCH 请求，则从磁盘或页缓存中读取消息。
    - Purgatory 的组件：缓存一时未满足条件不能立刻处理的请求
- 消费者重平衡流程
  - 触发与通知
    - 组成员数量发生变化。
    - 订阅主题数量发生变化。
    - 订阅主题的分区数发生变化。
  - 重平衡的通知机制正是通过心跳线程来完成的。当协调者决定开启新一轮重平衡后，它会将“REBALANCE_IN_PROGRESS”封装进心跳请求的响应中，发还给消费者实例。当消费者实例发现心跳响应中包含了“REBALANCE_IN_PROGRESS”，就能立马知道重平衡又开始了，这就是重平衡的通知机制。
  - 重平衡一旦开启，Broker 端的协调者组件就要开始忙了，主要涉及到控制消费者组的状态流转。当前，Kafka 设计了一套消费者组状态机（State Machine），来帮助协调者完成整个重平衡流程。
  - Kafka 为消费者组定义了 5 种状态，它们分别是：Empty、Dead、PreparingRebalance、CompletingRebalance 和 Stable。
    - ![img](https://i.imgur.com/olufDkY.png)
  - 状态机
    - ![img](https://i.imgur.com/WCfD5sp.png)
    - 一个消费者组最开始是 Empty 状态，当重平衡过程开启后，它会被置于 PreparingRebalance 状态等待成员加入，之后变更到 CompletingRebalance 状态等待分配方案，最后流转到 Stable 状态完成重平衡。
    - 当有新成员加入或已有成员退出时，消费者组的状态从 Stable 直接跳到 PreparingRebalance 状态，此时，所有现存成员就必须重新申请加入组。当所有成员都退出组后，消费者组状态变更为 Empty。Kafka 定期自动删除过期位移的条件就是，组要处于 Empty 状态。因此，如果你的消费者组停掉了很长时间（超过 7 天），那么 Kafka 很可能就把该组的位移数据删除了。
  - 消费者端重平衡流程
    - 在消费者端，重平衡分为两个步骤：分别是加入组和等待领导者消费者（Leader Consumer）分配方案。这两个步骤分别对应两类特定的请求：JoinGroup 请求和 SyncGroup 请求。
      - 当组内成员加入组时，它会向协调者发送 JoinGroup 请求。在该请求中，每个成员都要将自己订阅的主题上报，这样协调者就能收集到所有成员的订阅信息。一旦收集了全部成员的 JoinGroup 请求后，协调者会从这些成员中选择一个担任这个消费者组的领导者。通常情况下，第一个发送 JoinGroup 请求的成员自动成为领导者。你一定要注意区分这里的领导者和之前我们介绍的领导者副本，它们不是一个概念。这里的领导者是具体的消费者实例，它既不是副本，也不是协调者。领导者消费者的任务是收集所有成员的订阅信息，然后根据这些信息，制定具体的分区消费分配方案。
      - 选出领导者之后，协调者会把消费者组订阅信息封装进 JoinGroup 请求的响应体中，然后发给领导者，由领导者统一做出分配方案后，进入到下一步：发送 SyncGroup 请求。在这一步中，领导者向协调者发送 SyncGroup 请求，将刚刚做出的分配方案发给协调者。值得注意的是，其他成员也会向协调者发送 SyncGroup 请求，只不过请求体中并没有实际的内容。这一步的主要目的是让协调者接收分配方案，然后统一以 SyncGroup 响应的方式分发给所有成员，这样组内所有成员就都知道自己该消费哪些分区了。
      - JoinGroup 请求的主要作用是将组成员订阅信息发送给领导者消费者，待领导者制定好分配方案后，重平衡流程进入到 SyncGroup 请求阶段。
      - SyncGroup 请求的主要目的，就是让协调者把领导者制定的分配方案下发给各个组内成员。当所有成员都成功接收到分配方案后，消费者组进入到 Stable 状态，即开始正常的消费工作。
    - Broker 端重平衡场景
      - 新成员入组
        - 新成员入组是指组处于 Stable 状态后，有新成员加入。如果是全新启动一个消费者组，Kafka 是有一些自己的小优化的，流程上会有些许的不同。我们这里讨论的是，组稳定了之后有新成员加入的情形。当协调者收到新的 JoinGroup 请求后，它会通过心跳请求响应的方式通知组内现有的所有成员，强制它们开启新一轮的重平衡。具体的过程和之前的客户端重平衡流程是一样的。现在，我用一张时序图来说明协调者一端是如何处理新成员入组的。
      - 组成员主动离组。
        - 何谓主动离组？就是指消费者实例所在线程或进程调用 close() 方法主动通知协调者它要退出。这个场景就涉及到了第三类请求：LeaveGroup 请求。协调者收到 LeaveGroup 请求后，依然会以心跳响应的方式通知其他成员
      - 组成员崩溃离组。
        - 崩溃离组是指消费者实例出现严重故障，突然宕机导致的离组。它和主动离组是有区别的，因为后者是主动发起的离组，协调者能马上感知并处理。但崩溃离组是被动的，协调者通常需要等待一段时间才能感知到，这段时间一般是由消费者端参数 session.timeout.ms 控制的。也就是说，Kafka 一般不会超过 session.timeout.ms 就能感知到这个崩溃。
      - 场景四：重平衡时协调者对组内成员提交位移的处理。
        - 正常情况下，每个组内成员都会定期汇报位移给协调者。当重平衡开启时，协调者会给予成员一段缓冲时间，要求每个成员必须在这段时间内快速地上报自己的位移信息，然后再开启正常的 JoinGroup/SyncGroup 请求发送。
- Kafka 控制器
  - 控制器组件（Controller），是 Apache Kafka 的核心组件。它的主要作用是在 Apache ZooKeeper 的帮助下管理和协调整个 Kafka 集群。
  - 在运行过程中，只能有一个 Broker 成为控制器，行使其管理和协调的职责。换句话说，每个正常运转的 Kafka 集群，在任意时刻都有且只有一个控制器。
  - 控制器是重度依赖 ZooKeeper 的，ZooKeeper 常被用来实现集群成员管理、分布式锁、领导者选举等功能
  - Kafka 当前选举控制器的规则是：第一个成功创建 /controller 节点的 Broker 会被指定为控制器。
  - 控制器职责
    - 主题管理（创建、删除、增加分区）
      - 这里的主题管理，就是指控制器帮助我们完成对 Kafka 主题的创建、删除以及分区增加的操作。换句话说，当我们执行 kafka-topics 脚本时，大部分的后台工作都是控制器来完成的。关于 kafka-topics 脚本，我会在专栏后面的内容中，详细介绍它的使用方法。
    - 2.分区重分配
      - 分区重分配主要是指，kafka-reassign-partitions 脚本（关于这个脚本，后面我也会介绍）提供的对已有主题分区进行细粒度的分配功能。这部分功能也是控制器实现的。
    - 3.Preferred 领导者选举
      - Preferred 领导者选举主要是 Kafka 为了避免部分 Broker 负载过重而提供的一种换 Leader 的方案。在专栏后面说到工具的时候，我们再详谈 Preferred 领导者选举，这里你只需要了解这也是控制器的职责范围就可以了。
    - 4.集群成员管理（新增 Broker、Broker 主动关闭、Broker 宕机）
      - 这是控制器提供的第 4 类功能，包括自动检测新增 Broker、Broker 主动关闭及被动宕机。这种自动检测是依赖于前面提到的 Watch 功能和 ZooKeeper 临时节点组合实现的。 比如，控制器组件会利用 Watch 机制检查 ZooKeeper 的 /brokers/ids 节点下的子节点数量变更。目前，当有新 Broker 启动后，它会在 /brokers 下创建专属的 znode 节点。一旦创建完毕，ZooKeeper 会通过 Watch 机制将消息通知推送给控制器，这样，控制器就能自动地感知到这个变化，进而开启后续的新增 Broker 作业。 侦测 Broker 存活性则是依赖于刚刚提到的另一个机制：临时节点。每个 Broker 启动后，会在 /brokers/ids 下创建一个临时 znode。当 Broker 宕机或主动关闭后，该 Broker 与 ZooKeeper 的会话结束，这个 znode 会被自动删除。同理，ZooKeeper 的 Watch 机制将这一变更推送给控制器，这样控制器就能知道有 Broker 关闭或宕机了，从而进行“善后”。
    - 5.数据服务
      - 控制器的最后一大类工作，就是向其他 Broker 提供数据服务。控制器上保存了最全的集群元数据信息，其他所有 Broker 会定期接收控制器发来的元数据更新请求，从而更新其内存中的缓存数据。
  - 控制器维护数据
    - 所有主题信息。包括具体的分区信息，比如领导者副本是谁，ISR 集合中有哪些副本等。
    - 所有 Broker 信息。包括当前都有哪些运行中的 Broker，哪些正在关闭中的 Broker 等。
    - 所有涉及运维任务的分区。包括当前正在进行 Preferred 领导者选举以及分区重分配的分区列表。
  - 控制器故障转移
    - 故障转移指的是，当运行中的控制器突然宕机或意外终止时，Kafka 能够快速地感知到，并立即启用备用控制器来代替之前失败的控制器。
    - 最开始时，Broker 0 是控制器。当 Broker 0 宕机后，ZooKeeper 通过 Watch 机制感知到并删除了 /controller 临时节点。之后，所有存活的 Broker 开始竞选新的控制器身份。Broker 3 最终赢得了选举，成功地在 ZooKeeper 上重建了 /controller 节点。之后，Broker 3 会从 ZooKeeper 中读取集群元数据信息，并初始化到自己的缓存中。至此，控制器的 Failover 完成，可以行使正常的工作职责了。
  - 内部设计
    - 多线程的方案改成了单线程加事件队列的方案。控制器缓存中保存的状态只被一个线程处理，因此不再需要重量级的线程同步机制来维护线程安全，Kafka 不用再担心多线程并发访问的问题，非常利于社区定位和诊断控制器的各种问题。
    - 将之前同步操作 ZooKeeper 全部改为异步操作。ZooKeeper 本身的 API 提供了同步写和异步写两种方式。之前控制器操作 ZooKeeper 使用的是同步的 API，性能很差，集中表现为，当有大量主题分区发生变更时，ZooKeeper 容易成为系统的瓶颈。新版本 Kafka 修改了这部分设计，完全摒弃了之前的同步 API 调用，转而采用异步 API 写入 ZooKeeper，性能有了很大的提升。
