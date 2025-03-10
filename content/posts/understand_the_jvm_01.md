---
title: "深入理解Java虚拟机::走进 Java"
date: 2021-03-31T22:10:56+08:00
draft: false
keywords: ["Java虚拟机", "JVM", "JDK", "Java发展史", "HotSpot"]
---

## 走进 Java

- 发展史
  - 1996.01.23 JDK 1.0 发布
    - JVM
    - Applet
    - AWT
  - 1996.02.19 JDK 1.1 发布
    - JAR 文件格式
    - JDBC
    - JavaBeans
    - RMI
    - 内部类
    - 反射
  - 1998.12.04 JDK 1.2 发布
    - J2SE
    - J2EE
    - J2ME
    - EJB
    - Java Plug-in
    - Java IDL
    - Swing
    - JIT
    - VM
      - Classic VM
      - HotSpot VM
      - Exact VM
      - strictfp 关键字
  - 2000.05.08 JDK 1.3 发布
    - JNDI
    - Java 2D API
    - JavaSound
  - 2002.02.13 JDK 1.4 发布
    - 正则表达式
    - 异常链
    - NIO
    - 日志类
    - XML 解析器和 XSLT 转换器
  - 2004.09.30 JDK 5 发布
    - 自动装箱
    - 范型
    - 动态注解
    - 枚举
    - 可变长参数
    - 遍历循环
    - JMM
    - JUC
  - 2006.12.11 JDK 6 发布
    - 动态语言支持
    - 编译期注解处理器
    - 微型 HTTP 服务器 API
    - 锁和同步，垃圾回收，类加载改进
  - 2009.02.19 JDK 7 发布
    - 支持 Mac OS
    - G1 收集器
    - 加强对非 Java 语言的调用支持
    - 可并行的类加载架构
  - 2014.03.18 JDK 8 发布
    - Lambda 支持
    - Nashorn JavaScript 引擎
    - 新的时间日期 API
    - 彻底移除 HotSpot 的永久代
  - 2017.09.21 JDK 9 发布
    - Jigsaw 模块化
    - JShell
    - JLink
    - HTTP 2
  - 2018.03.20 JDK 10 发布
    - 本地类型推断增强
    - 整合 JDK 代码仓库
    - 统一的垃圾回收接口
    - 应用程序类数据共享
  - 2018.09.25 JDK 11 发布
    - ZGC
    - 授权许可调整
    - Epsilon：低开销垃圾回收器
    - 标准 HTTP Client 升级
    - 基于嵌套的访问控制
    - 简化启动单个源代码文件的方法
      - 用于 Lambda 参数的局部变量语法
      - 低开销的 Heap Profiling
      - 支持 TLS 1.3 协议
      - 飞行记录器
  - 2019.03.20 JDK 12 发布
    - Switch 表达式
    - Shenandoah 垃圾回收器
  - 2019.08.17 JDK 13 发布
    - ZGC 增强
    - 更新 Socket 实现
    - Switch 表达式更新
    - 文本块
  - 2020.03.17 JDK 14 发布
    - instanceof 模式匹配
    - G1 的 NUMA 可识别内存分配
    - 改进 NullPointerExceptions 提示信息
    - Record 类型
    - Switch 表达式
    - 删除 CMS 垃圾回收器
  - 2020.09.15 JDK 15 发布
    - Edwards-Curve 数字签名算法
    - 封闭类
    - 禁用、弃用偏向锁
- Java 虚拟机家族
  - Sun Classic/Exact VM
  - HotSpot VM
  - Mobile/Embedded VM
  - BEA JRockit/IBM J9 VM
  - BEA Liquid VM/Azul VM
  - Apache Harmony/Google Android Dalvik VM
  - Microsoft JVM 及其他
