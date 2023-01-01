---
title: "Operating Systems Three Easy Pieces"
date: 2021-04-07T22:52:08+08:00
draft: true
tags: ["Operating Systems"]
---

# Operating Systems Three Easy Pieces

## 1. A Dialogue on the Book

- They are the three key ideas we’re going to learn about: virtualiza- tion, concurrency, and persistence.
- how an operating system works
  - including how it decides what program to run next on a CPU
  - how it handles memory overload in a virtual memory system
  - how virtual machine monitors work
  - how to manage information on disks
  - even a little about how to build a distributed system that works when parts have failed.

## 2. Introduction to Operating Systems

- what happens when a program runs?
  - the processor fetches an instruction from memory
  - decodes it
  - executes it
  - moves on to the next instruction
- The primary way the OS does this is through a general technique that we call virtualization.
  - That is, the OS takes a physical resource (such as the processor, or memory, or a disk) and transforms it into a more general, powerful, and easy-to-use virtual form of itself.
  - Thus, we sometimes refer to the operating system as a virtual machine.
- System call
  - some interfaces (APIs) that you can call provide by OS
- Resource Manager
  - Each of the CPU, memory, and disk is a resource of the system
  - it is thus the operating system’s role to manage those resources
- Virtualizing The CPU
  - Turning a single CPU (or a small set of them) into a seemingly infinite number of CPUs and thus allowing many programs to seemingly run at once is what we call virtualizing the CPU.
- Virtualizing Memory
  - Each process accesses its own private virtual address space
  - the OS somehow maps onto the physical memory of the machine
- Concurrency
  - Pthread_create
  - Atomically
- Persistence
  - In system memory, data can be easily lost, as devices such as DRAM store values in a volatile manner
  - when power goes away or the system crashes, any data in memory is lost
  - Thus, we need hardware and software to be able to store data persistently
  - such storage is thus critical to any system as users care a great deal about their data.
- Design Goals
  - One goal in designing and implementing an operating system is to provide high performance; another way to say this is our goal is to mini- mize the overheads of the OS.
  - Another goal will be to provide protection between applications, as well as between the OS and applications.
- Some History
  - Early Operating Systems: Just Libraries
    - batch processing
  - Beyond Libraries: Protection
    - system call
    - procedure call
    - hardware privilege level
    - user mode
    - trap handler
    - kernel mode
    - retur-from-trap
  - The Era of Multiprogramming
    - minicomputer
    - multiprogramming
    - memory protection
    - concurrency
  - The Modern Era
    - personal computer
    - Disk Operating System
    - Mac OS
    - Linux
    - Window NT

## 3. A Dialogue on Virtualization

- What virtualization does is take that single CPU and make it look like many virtual CPUs to the applications running on the system. Thus, while each application thinks it has its own CPU to use, there is really only one. And thus the OS has created a beautiful illusion: it has virtualized the CPU.

## 4. The Abstraction: The Process

- The definition of a process, informally, is quite simple: it is a running program
- This basic technique, known as time sharing of the CPU, allows users to run as many concurrent processes as they would like
- the potential cost is performance, as each will run more slowly if the CPU(s) must be shared
- Process API
  - Create: An operating system must include some method to create new processes.
    - load its code and any static data (e.g. initialized variables) into memory, into the ad dress space of the process.
      - modern OSes perform the process lazily, by loading pieces of code or data only as they are needed during program execution
        - paging and swapping
    - Some memory must be allocated for the program’s run-time stack
    - The OS will also likely initialize the stack with arguments; specifically, it will fill in the parameters to the `main()` function, i.e., argc and the argv array.
    - The OS may also allocate some memory for the program’s heap.
      - `malloc()` and `free()`
    - The OS will also do some other initialization tasks, particularly as related to input/output (I/O).
      - file descriptors
        - standard input, output, and error
      - persistence
  - Destroy: As there is an interface for process creation, systems also provide an interface to destroy processes forcefully.
  - Wait: Sometimes it is useful to wait for a process to stop running
  - Miscellaneous Control: suspend a process and then resume it .
  - Status: There are usually interfaces to get some status information about a process.
    - Running: In the running state, a process is running on a processor. This means it is executing instructions.
    - Ready: In the ready state, a process is ready to run but for some reason the OS has chosen not to run it at this given moment.
    - Blocked: In the blocked state, a process has performed some kind of operation that makes it not ready to run until some other event takes place.
    - OS scheduler
- Data Structures
  - process list
    - contains information about all processes in the system
  - process control block (PCB)
    - a structure that contains information about a specific process.
