---
title: "深入理解Java虚拟机::类文件结构"
date: 2021-04-05T22:10:56+08:00
draft: false
keywords: ["Class文件结构", "JVM字节码", "常量池", "属性表", "字节码指令"]
---

## 类文件结构

- 实现语言无关性的基础是虚拟机和字节码存储格式
- Class 类文件的结构
  - Class 文件格式采用一种类似于 C 语言结构体的伪结构来存储数据，这种伪结构中只有两种数据类型无符号数和表
    - 无符号数属于基本的数据类型，以 u1、u2、u4、u8 来分别代表 1 个字节、2 个字节、4 个字节和 8 个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或者按照 UTF-8 编码构成字符串值
    - 表是由多个无符号数或者其他表作为数据项构成的复合数据类型，为了便于区分，所有表的命名都习惯性地以“\_info”结尾。表用于描述有层次关系的复合结构的数据，整个 Class 文件本质上也可以视作是一张表，
  ```c
  ClassFile {
      u4 magic; //0xCAFEBABE
      u2 minor_version; //class file minor version
      u2 major_version; //class file major version
      u2 constant_pool_count; //count of entries in next item
      cp_info constant_pool[constant_pool_count-1]; //constants
      u2 access_flags; //class assess flags
      u2 this_class; //index of this class to const pool
      u2 super_class; //index of super class to const pool
      u2 interfaces_count; //number of interfaces implemented
      u2 interfaces[interfaces_count];//indices of interfaces
      u2 fields_count; //number of fields in the class
      field_info fields[fields_count];//fields descriptions
      u2 methods_count; //number of methods in the class
      method_info methods[methods_count]; //methods descriptions
      u2 attributes_count; //number of attributes of the class attribute_info
      attributes[attributes_count]; //attributes
  }
  ```
  - 魔数与 Class 文件的版本
    - 魔数
      - 每个 Class 文件的头 4 个字节被称为魔数(Magic Number)，它的唯一作用是确定这个文件是否为一个能被虚拟机接受的 Class 文件。
    - Class 文件的版本号
      - 第 5 和第 6 个字节是次版本号(Minor Version)，第 7 和第 8 个字节是主版本号(Major Version)。
      - 高版本需要向下兼容以前版本的 Class 文件，但不能运行以后版本的 Class 文件。
  - 常量池
    - 常量池的入口放置一项 u2 类型的数据，代表常量池容量计数值(constant_pool_count)
    - Class 文件结构中只有常量池的容量计数是从 1 开始, 0 用于表示无常量池引用
    - 常量池中主要存放两大类常量: 字面量(Literal)和符号引用(Symbolic References)
      - 字面量
        - 文本字符串
        - 声明为 final 的常量值
      - 符号引用
        - 被模块导出或者开放的包
        - 类和接口的全限定名
        - 字段的名称和描述符
        - 方法的名称和描述符
        - 方法句柄和方法类型
        - 动态调用点和动态常量
    - 常量池中的每一项常量都是一个表，互相之间的结构都不相同
      - [JVM 规范中关于 Class 文件格式的章节](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html)
  - 访问标志
    - 识别一些类或者接口层次的访问信息
    - 有 16 个标志位可以使用，当前只定义了其中 9 个
  - 类索引、父类索引与接口索引集合
    - 类索引用于确定这个类的全限定名
    - 父类索引用于确定这个类的父类的全限定名
    - 接口索引集合就用来描述这个类实现了哪些接口，这些被实现的接口将按 implements 关键字后的接口顺序从左到右排列在接口索引集合中。
  - 字段表集合
    - 字段表用于描述接口或者类中声明的变量
      - 类级变量以及实例级变量，但不包括在方法内部声明的局部变量
    - 字段表集合中不会列出从父类或者父接口中继承而来的字段
  - 方法表集合
    - volatile 关键字和 transient 关键字不能修饰方法
    - Java 语言中，重载方法需要签名不同
    - Class 文件格式中，允许方法签名相同，返回值不同的方法存在
  - 属性表集合
    - Code 属性
      - Java 程序方法体里面的代码经过 Javac 编译器处理之后，最终变为字节码指令存储在 Code 属性内。
      - 接口和抽象方法没有 Code 属性
      - 《Java 虚拟机规范》中明确限制了一个方法不允许超过 65535 条字节码指令
    - Exceptions 属性
      - Exceptions 属性的作用是列举出方法中可能抛出的受查异常
    - LineNumberTable 属性
      - LineNumberTable 属性用于描述 Java 源码行号与字节码行号(字节码的偏移量)之间的对应关系
      - 非必需，对程序运行产生的最主要影响就是当抛出异常时，堆栈中将不会显示出错的行号
    - LocalVariableTable 及 LocalVariableTypeTable 属性
      - LocalVariableTable 属性用于描述栈帧中局部变量表的变量与 Java 源码中定义的变量之间的关系
      - 非必需，影响是当其他人引用这个方法时，所有的参数名称都将会丢失
    - SourceFile 及 SourceDebugExtension 属性
      - SourceFile 属性用于记录生成这个 Class 文件的源码文件名称
      - 非必需，当抛出异常时，堆栈中将不会显示出错代码所属的文件名
    - ConstantValue 属性
      - ConstantValue 属性的作用是通知虚拟机自动为静态变量赋值。只有被 static 关键字修饰的变量(类变量)才可以使用这项属性。
    - InnerClasses 属性
      - InnerClasses 属性用于记录内部类与宿主类之间的关联。
      - 如果一个类中定义了内部类，那编译器将会为它以及它所包含的内部类生成 InnerClasses 属性。
    - Deprecated 及 Synthetic 属性
      - Deprecated 和 Synthetic 两个属性都属于标志类型的布尔属性，只存在有和没有的区别，没有属性值的概念。
      - Deprecated 属性用于表示某个类、字段或者方法，已经被程序作者定为不再推荐使用，它可以通过代码中使用`@deprecated`注解进行设置。
    - StackMapTable 属性
      - 目的在于代替以前比较消耗性能的基于数据流分析的类型推导验证器。
    - Signature 属性
      - 它是一个可选的定长属性，可以出现于类、字段表和方法表结构的属性表中
      - Signature 属性会为记录泛型签名信息。之所以要专门使用这样一个属性去记录泛型类型，是因为 Java 语言的泛型采用的是擦除法实现的伪泛型，字节码(Code 属性)中所有的泛型信息编译(类型变量、参数化类型)在编译之后都通通被擦除掉
    - BootstrapMethods 属性
      - 用于保存 invokedynamic 指令引用的引导方法限定符。
    - MethodParameters 属性
      - MethodParameters 的作用是记录方法的各个形参名称和信息。
    - 模块化相关属性
      - Module 属性是一个非常复杂的变长属性，除了表示该模块的名称、版本、标志信息以外，还存储了这个模块 requires、exports、opens、uses 和 provides 定义的全部内容
    - 运行时注解相关属性
      - RuntimeVisibleAnnotations 是一个变长属性，它记录了类、字段或方法的声明上记录运行时可见注解，当我们使用反射 API 来获取类、字段或方法上的注解时，返回值就是通过这个属性来取到的
- 字节码指令简介
  - Java 虚拟机的指令由一个字节长度的、代表着某种特定操作含义的数字以及跟随其后的零至多个代表此操作所需的参数构成
  - 加载和存储指令
    - 加载和存储指令用于将数据在栈帧中的局部变量表和操作数栈之间来回传输
  - 运算指令
    - 算术指令用于对两个操作数栈上的值进行某种特定运算，并把结果重新存入到操作栈顶
    - 大体上运算指令可以分为两种: 对整型数据进行运算的指令与对浮点型数据进行运算的指令。
  - 类型转换指令
    - 类型转换指令可以将两种不同的数值类型相互转换，这些转换操作一般用于实现用户代码中的显式类型转换操作
  - 对象创建与访问指令
    - 虽然类实例和数组都是对象，但 Java 虚拟机对类实例和数组的创建与操作使用了不同的字节码指令(在下一章会讲到数组和普通类的类型创建过程是不同的)。对象创建后，就可以通过对象访问指令获取对象实例或者数组实例中的字段或者数组元素
  - 操作数栈管理指令
    - 用于直接操作操作数栈的指令
  - 控制转移指令
    - 控制转移指令可以让 Java 虚拟机有条件或无条件地从指定位置指令的下 一条指令继续执行程序，从概念模型上理解，可以认为控制指令就是在有条件或无条件地修改 PC 寄存器的值
  - 方法调用和返回指令
    - 用于方法调用
  - 异常处理指令
    - 显式抛出异常的操作都由 athrow 指令来实现
  - 同步指令
    - Java 虚拟机可以支持方法级的同步和方法内部一段指令序列的同步，这两种同步结构都是使用管程来实现
- 公有设计，私有实现
  - 《Java 虚拟机规范》描绘了 Java 虚拟机应有的共同程序存储格式:Class 文件格式以及字节码指令集。这些内容与硬件、操作系统和具体的 Java 虚拟机实现之间是完全独立的，虚拟机实现者可能更愿意把它们看作程序在各种 Java 平台实现之间互相安全地交互的手段。
  - 一个优秀的虚拟机实现，在满足《Java 虚拟机规范》的约束下对具体实现做出修改和优化也是完全可行的，并且《Java 虚拟机规范》中明确鼓励实现者这样去做。只要优化以后 Class 文件依然可以被正确读取，并且包含在其中的语义能得到完整保持，那实现者就可以选择以任何方式去实现这些语义，虚拟机在后台如何处理 Class 文件完全是实现者自己的事情，只要它在外部接口上看起来与规范描述的一致即可
- Class 文件结构的发展
  - Class 文件结构一直处于一个相对比较稳定的状态，Class 文件的主体结构、字节码指令的语义和数量几乎没有出现过变动，所有对 Class 文件格式的改进，都集中在访问标志、属性表这些设计上原本就是可扩展的数据结构中添加新内容。
  - 二十余年间，字节码的数量和语义只发生过屈指可数的几次变动，例如 JDK 1.0.2 时改动过 invokespecial 指令的语义，JDK 7 增加了 invokedynamic 指令，禁止了 ret 和 jsr 指令。
