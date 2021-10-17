---
title: "深入理解Java虚拟机::虚拟机性能监控、故障处理工具"
date: 2021-04-03T22:10:56+08:00
draft: false
---

## 虚拟机性能监控、故障处理工具

- jps
  - 虚拟机进程状况工具
  - 可以列出正在运行的虚拟机进程，并显示虚拟机执行主类(Main Class，main()函数所在的类)名称以及这些进程的本地虚拟机唯一 ID(LVMID，Local Virtual Machine Identifier)
- jstat
  - 虚拟机统计信息监视工具
  - 显示本地或者远程虚拟机进程中的类加载、内存、垃圾收集、即时编译等运行时数据
- jinfo
  - 是实时查看和调整虚拟机各项参数
  - System.getProperties()
- jmap
  - 生成堆转储快照
  - 查询 finalize 执行队列
  - Java 堆和方法区的详细信息
  - 如空间使用率
  - 当前用的是哪种收集器
- jhat
  - 与 jmap 搭配使用，来分析 jmap 生成的堆转储快照
- jstack
  - 用于生成虚拟机当前时刻的线程快照
