---
title: MIT6.S081-lec3
date: 2022-10-05 09:25:00
author: spikeHu
cover: true
coverImg: /images/cover1.jpg
summary: MIT-OS lec3课程笔记

categories: MIT-OS
tags:
  - MIT-OS
  - LEC-3
---
# 6.S08-lecture 3 -OS organization and System Call

## 操作系统隔离性（isolation）

操作系统需要隔离性，在应用程序和操作系统之间需要强隔离性。

 实现隔离性需要硬件支持：

- user/kernel mode
- page table 或者说 virtual memory
<!--more-->
### User /Kernel Mode

当处于kernel mode 的时候，可以使用一些特别权限的指令（privileged instruction），当处于user mode的时候，使用普通的权限。

## Virtual Memory



page table :将 virtaul addr 映射到 physic addr

每一个进程都有一个自己独立的页表，所以每个进程只能访问自己页表中映射的物理地址,即每个用户有自己的内存视图。从而实现 memory isolatioon.

一般的用户程序就运行在user mode 下，而操作系统就运行在内核模式下（kernel mode），当用户程序调用read  write等system call的时候，需要得到内核的控制权，所以需要一种方法将控制权转如内核（transfro control into kernel）。在RISC-V中，有**ecall**实现该功能。每次调用ecal,程序就会通过一个接入点进入到内核中。比如用户程序调用write函数，并不是直接调用内核的write，而是通过ecall指令进行调用，之后控制权就到了sysycall函数（then transfer control to syscall）,syscall 会实际调用write函数。

## 内核的集中设计方法

### 宏内核

将整个内核代码运行在kernel mode，集成性好，有更好的性能，但这样会出现更多的bug，有安全隐患。

### 微内核

只在内核空间运行一小部分操作系统的模块。比如IPC, page table,一部分运行在user mode，比如文件系统FS。由于运行在kernel mode的代码更少，所以更加的安全。

如果用户空间的程序需要调用文件系统，该程序就需要通过IPC和FS进行通信，FS将运行结果通过IPC告知该程序结果。所以需要<u>**在kernel mode 和 user mode 进行2次跳转，性能上就比宏内核差一些**。同时微内核的模块之间分隔的比较好，那么共享就比较差，二宏内核中，共享性就比较好，比如文件系统和虚拟内存系统，可以很容易的共享page cache。</u>

## XV6

kernel:包含所有的内核文件，所有的文件会被编译成一个叫做kernel的二进制文件。这个二进制文件运行在kernel mode。

user:基本上是运行在user mode的程序。

mkfs:会创建一个空的镜像文件，这样就可以使用一个空的文件系统。 

