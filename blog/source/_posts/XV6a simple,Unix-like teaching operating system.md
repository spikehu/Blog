---
title: XV6a simple,Unix-like teaching operating system
date: 2022-10-05 09:25:00
author: spikeHu
cover: true
coverImg: /images/cover1.jpg
summary: XV6说明
categories: MIT-OS
tags:
  - MIT-OS
  - XV6
---

# XV6:a simple,Unix-like teaching operating system

## chapter 1 Operating System interfaces

![image-20220930100403335](typora-user-images\image-20220930100403335.png)

内核：一个为程序提供服务的特殊程序。（**kernel**: a special program that provides services to running programs）
<!--more-->

当一个进程需要唤起一个内核的服务的时候，it invoke a **system call**, 该方法属于操作系统的接口。

内核使用CPU提供的硬件保护机制确保每个运行在用户空间的进程只能访问自己的内存。

下图是XV6可以调用的所有系统方法。

![image-20220930102440246](typora-user-images\image-20220930102440246.png)

### 1.1 Processes and memory

An xv6 process consists of user-space memory(instruction ,data , and stack),and per-process state private to the kernel.

XV6 分时进程（time-shares processes）:显示的切换CPU资源给等待运行的进程队列。当一个进程没有运行的时候，xv6将保存它的CPU寄存器，当它再次运行的时候就恢复它们。内核会把进程与标识符或者PID相关联。



