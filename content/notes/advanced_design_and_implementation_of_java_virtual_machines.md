---
title: "Java虚拟机设计与实现"
date: 2021-03-21T01:26:08+08:00
draft: true
tags: ["JVM"]
---

# Advanced Design and Implementation of Virtual Machines

## Basics of Virtual Machines

### Introduction of the Virtual Machine

- TYPES OF VIRTUAL MACHINES
  - four types of virtual machines according to the level of abstraction and scope of emulation
    - Full instruction set architecture (ISA) virtual machine
      - VirtualBox, QEMU, and XEN
    - Application Binary Interface (ABI) virtual machine
      - Intel’s IA-32 Execution Layer on Itanium, Transmeta’s Code Morphing for X86 emulation, and Apple’s Rosetta trans-lation layer for PowerPC emulation
    - Virtual ISA virtual machine
      - Microsystem’s JVM, Microsoft’s Common Language Runtime, and Parrot Foundation’s Parrot virtual machine
    - Language virtual machine
      - the runtime engines for Basic, Lisp, Tcl, and Ruby
- WHY VIRTUAL MACHINE
  - Virtual machines are indispensable to modern programming. They help (computer) security, (programming) productivity, and (application) portability.
    - security
      - Memory safety ensures that a certain type of data in the memory always follow the restrictions of that type
      - Operation safety ensures that the operations on a certain type of data always follow the restrictions of that type
      - Control safety ensures that the flow of code execution never reach any point that either gets stuck or goes wild
    - productivity
      - Since a safe language can catch program bugs or execution errors early and safely at the compile-time or runtime, it largely improves developer’s productivity.
    - portability
      - Virtual machine helps portability in the sense that the virtual ISA or guest language is not tied to any specific native ISA or ABI definition
- VIRTUAL MACHINE EXAMPLES
  - execution engines
    - A virtual machine, as the runtime engine of the guest language, can be categorized accord- ing to the implementation of its execution engine. The two basic execution engines are interpretation and compilation.
  - examples
    - JavaScript Engine
    - Perl Engine
    - Android Java VM
    - Apache Harmony

### Inside of a Virtual Machine

- A full language implementation usually includes no less than three major parts:
  - the virtual machine ,the language libraries ,the tool set
- CORE COMPONENTS OF VIRTUAL MACHINE
  - Loader
    - parse the package into data structures
      - The data structures in memory have semantic meanings such as code and data
    - load additional resources needed by the application
  - Dynamic linker
    - a runtime component used when the application is going to be executed
    - resolve all the referenced symbols into accessible memory addresses
  - Execution Engine
    - performs the operations specified by the program code
    - the core component
  - Memory Manager
    - manage its data and the memory containing the data
    - Virtual machine data: Virtual machine needs memory to load the application code and hold supporting data. The data in this category are invisible to the application while necessary for the application’s execution.
    - Application data: An application needs storage for its static data and dynamic data. The data in this category are visible to the application. Application dynamic data are stored in the application’s heap.
    - manage mainly the application dynamic data, the memory of application heap
    - Necessary
      - A memory manager is necessary as a middle layer between what the application can see and what the underlying system can provide.
    - Desirable
      - Although the underlying system may provide certain level of memory reclamation support, it is desirable for the virtual machine to directly manage the application data (and the associated memory), because only virtual machine accurately knows the application’s data type and life cycles. If memory manager does not help recycle the no-longer useful data, the virtual machine may still run correctly, but the footprint and performance may suffer.
  - Thread Scheduler
    - threadingis a straightforward way to provide multitasking, parallelization, and event coordination
  - Language Extension
    - Safe language or high-level language has to depend on the virtual machine to access low- level resources due to the safety requirements. There are two complementary ways to pro- vide this kind of capabilities:
      - Runtime services
        - Runtime services can be provided to the application in various forms, such as APIs, runtime objects, and environment variables.
      - Language extension
        - foreign function interface
        - JNI
- VIRTUAL ISA
  - Virtual language
    - here means that it is not directly used by anyone in programming; instead, it is only automatically generated through tools. In other words, virtual language is usually used as the compilation target of other languages.
    - virtual languages are born to be compilation target languages.
    - Virtual instruction set architecture (ISA) is a kind of virtual language that defines the instruction set and execution model of a virtual machine.
  - Java Virtual Machine
    - JVM specification is not only a set of virtual instructions, but also all the architectural models of an abstract computing machine, including the execution model, memory model, threading model, and security mode.
    - The JVM instruction’s opcodes are encoded into one byte, thus called bytecode.
    - A byte can encode 256 numbers, of which 198 are currently used, 51 are unused, and 3 are reserved for JVM implementation’s runtime services
    - Java application is distributed in the form of Java class files. A Java class file contains the definition of a single class or interface. Like other binary file format such as executable and linkable format, Java class file includes mainly bytecode sequence and symbol table that contains the symbols referenced by the bytecode sequence.
