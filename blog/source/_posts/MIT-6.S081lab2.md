---
title: MIT-6.S081lab2
date: 2022-10-05 09:25:00
author: spikeHu
cover: true
coverImg: /images/cover1.jpg
summary: lab2的实现
categories: MIT-OS
tags:
  - MIT-OS
  - LAB-2
---

# lab2：system calls

## **Preparation**

[Read chapter 2](https://pdos.csail.mit.edu/6.S081/2021/xv6/book-riscv-rev2.pdf) and xv6 code: [kernel/proc.h](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/proc.h), [kernel/defs.h](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/defs.h), [kernel/entry.S](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/entry.S), [kernel/main.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/main.c), [user/initcode.S](https://github.com/mit-pdos/xv6-riscv/blob/riscv/user/initcode.S), [user/init.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/user/init.c), and skim [kernel/proc.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/proc.c) and [kernel/exec.c](https://github.com/mit-pdos/xv6-riscv/blob/riscv/kernel/exec.c)

In the last lab you used systems calls to write a few utilities. In this lab you will add some new system calls to xv6, which will help you understand how they work and will expose you to some of the internals of the xv6 kernel. You will add more system calls in later labs.
<!--more-->

Before you start coding, read Chapter 2 of the [xv6 book](https://pdos.csail.mit.edu/6.S081/2021/xv6/book-riscv-rev1.pdf), and Sections 4.3 and 4.4 of Chapter 4, and related source files:

- The user-space code for systems calls is in `user/user.h` and `user/usys.pl`.
- The kernel-space code is `kernel/syscall.h`, kernel/syscall.c.
- The process-related code is `kernel/proc.h` and `kernel/proc.c`.

这个实验前的准备比较复杂，参考B站Up进行配置

To start the lab, switch to the syscall branch:

```
  $ git fetch
  $ git checkout syscall
  $ make clean
```

https://www.bilibili.com/video/BV1VZ4y197uk/?spm_id_from=333.788&vd_source=3beb6fdc520324dc1427808056eb9e4e

得到的trace.c文件为：

~~~C
#include "kernel/param.h"
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(int argc, char *argv[])
{
  int i;
  char *nargv[MAXARG];

  if(argc < 3 || (argv[1][0] < '0' || argv[1][0] > '9')){
    fprintf(2, "Usage: %s mask command\n", argv[0]);
    exit(1);
  }

  if (trace(atoi(argv[1])) < 0) {
    fprintf(2, "%s: trace failed\n", argv[0]);
    exit(1);
  }
    
  for(i = 2; i < argc && i < MAXARG; i++){
    nargv[i-2] = argv[i];
  }
  exec(nargv[0], nargv);
  exit(0);
}

~~~

这段代码比较简单，就是在trace中调用后面输入的命令

## System call tracing(moderate)

~~~
In this assignment you will add a system call tracing feature that may help you when debugging later labs. You'll create a new trace system call that will control tracing. It should take one argument, an integer "mask", whose bits specify which system calls to trace. For example, to trace the fork system call, a program calls trace(1 << SYS_fork), where SYS_fork is a syscall number from kernel/syscall.h. You have to modify the xv6 kernel to print out a line when each system call is about to return, if the system call's number is set in the mask. The line should contain the process id, the name of the system call and the return value; you don't need to print the system call arguments. The trace system call should enable tracing for the process that calls it and any children that it subsequently forks, but should not affect other processes.
~~~

### 实验大意要求

让我们写一个trace的系统函数，这个函数有一个参数表示掩码mask，用来表示trace哪个系统函数。比如，trace fork这个系统函数，那么程序调用就是:trace(1<<SYS_fork),1<<fork就是表示将1左移多少位，SYS_fork是位于kernel/syscall.h文件的一个syscall number,表示一个系统函数的一个序号（比如fork就是1左移一位的数字就是1，大概是这个意思）.我们需要修改xv6的内核，打印出什么时候系统调用返回的，如果这个系统函数的number在mask中的话。我们需要打印出process id, the name of system call ,以及system call的返回值，不需要打印出system call 的参数。这个trace系统函数需要能够tracing调用它的进程以及它的子进程，但是不能影响其他进程。

We provide a `trace` user-level program that runs another program with tracing enabled (see `user/trace.c`). When you're done, you should see output like this:

```bash
$ trace 32 grep hello README
3: syscall read -> 1023
3: syscall read -> 966
3: syscall read -> 70
3: syscall read -> 0
$
$ trace 2147483647 grep hello README
4: syscall trace -> 0
4: syscall exec -> 3
4: syscall open -> 3
4: syscall read -> 1023
4: syscall read -> 966
4: syscall read -> 70
4: syscall read -> 0
4: syscall close -> 0
$
$ grep hello README
$
$ trace 2 usertests forkforkfork
usertests starting
test forkforkfork: 407: syscall fork -> 408
408: syscall fork -> 409
409: syscall fork -> 410
410: syscall fork -> 411
409: syscall fork -> 412
410: syscall fork -> 413
409: syscall fork -> 414
411: syscall fork -> 415
...
$   
```

In the first example above, trace invokes grep tracing just the read system call. The 32 is `1<<SYS_read`. In the second example, trace runs grep while tracing all system calls; the 2147483647 has all 31 low bits set. In the third example, the program isn't traced, so no trace output is printed. In the fourth example, the fork system calls of all the descendants of the `forkforkfork` test in `usertests` are being traced. Your solution is correct if your program behaves as shown above (though the process IDs may be different).

Some hints:

- Add `$U/_trace` to UPROGS in Makefile
- Run make qemu and you will see that the compiler cannot compile `user/trace.c`, because the user-space stubs for the system call don't exist yet: add a prototype for the system call to `user/user.h`, a stub to `user/usys.pl`, and a syscall number to `kernel/syscall.h`. The Makefile invokes the perl script `user/usys.pl`, which produces `user/usys.S`, the actual system call stubs, which use the RISC-V `ecall` instruction to transition to the kernel. Once you fix the compilation issues, run trace 32 grep hello README; it will fail because you haven't implemented the system call in the kernel yet.
- Add a `sys_trace()` function in `kernel/sysproc.c` that implements the new system call by remembering its argument in a new variable in the `proc` structure (see `kernel/proc.h`). The functions to retrieve system call arguments from user space are in `kernel/syscall.c`, and you can see examples of their use in `kernel/sysproc.c`.
- Modify `fork()` (see `kernel/proc.c`) to copy the trace mask from the parent to the child process.
- Modify the `syscall()` function in `kernel/syscall.c` to print the trace output. You will need to add an array of syscall names to index into.

跟着上面的步骤就可以将准备工作做好了。

### 前置知识

#### 函数指针：

函数指针是指向函数的指针，函数指针的声明方法为

~~~
返回值类型 (*指针变量名)（[形参列表]）
~~~

例子：

~~~C
int func(int x); /* 声明一个函数 */
int (*f) (int x); /* 声明一个函数指针  其中形参可有可无 视情况而定*/
f=func; /* 将func函数的首地址赋给指针f */
~~~

### 思路

分析命令 trace 32 grep hello README

首先在用户模式下调用trace.c ，由于在trace.c中调用了内核中得指令trace,所以sys_trace被调用。在trace.c中如下代码块中可以发现：

~~~C
  if (trace(atoi(argv[1])) < 0) {
    fprintf(2, "%s: trace failed\n", argv[0]);
    exit(1);
  }
~~~

其中atoi(argv[1])指传入的参数32。

### 代码

在sysproc.c中的sys_trace(void)加入如下代码：

~~~C
uint64 sys_trace(void)
{
  struct proc *p = myproc();
  //取得传入的参数
  int arg  ;
  if(argint(0,&arg)<0) return -1; 
  p->mask = arg;
  return 0;
}
~~~

在proc.h中 proc数据结构中加入mask，用来表示追踪的命令。

pro：存储进程相关信息的结构

~~~C
// Per-process state
struct proc {
  struct spinlock lock;

  // p->lock must be held when using these:
  enum procstate state;        // Process state
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  int xstate;                  // Exit status to be returned to parent's wait
  int pid;                     // Process ID
 
  // wait_lock must be held when using this:
  struct proc *parent;         // Parent process

  // these are private to the process, so p->lock need not be held.
  uint64 kstack;               // Virtual address of kernel stack
  uint64 sz;                   // Size of process memory (bytes)
  pagetable_t pagetable;       // User page table
  struct trapframe *trapframe; // data page for trampoline.S
  struct context context;      // swtch() here to run process
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
  
  //在该结构中存放trace中传入的掩码
  int mask;
};

~~~

在syscall.c中syscall(void)代码如下:

~~~C

void
syscall(void)
{
  int num;
  struct proc *p = myproc();
  num = p->trapframe->a7;


  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {

    p->trapframe->a0 = syscalls[num]();
  } else {
    printf("%d %s: unknown sys call %d\n",
            p->pid, p->name, num);
    p->trapframe->a0 = -1;
  }
    //如果该sysycall是从trace.c调用的 
  //且mask进行了标记 那么就打印出syscall的消息
  if((p->mask)>>num &1)
  {
      printf("syscall %s -> %d\n",syscall_names[num-1],p->trapframe->a0);
  }

}
~~~

以及

~~~C
char* syscall_names[]={"sys_fork","sys_exit",
                      "sys_wait","sys_pipe","sys_read",
                      "sys_kill","sys_exec","sys_fstat",
                      "sys_chdir","sys_dup","sys_getpid",
                      "sys_sbrk","sys_sleep","sys_uptime",
                      "sys_open","sys_write","sys_mknod",
                      "sys_unlink","sys_link","sys_mkdir",
                      "sys_close","sys_trace",
                      };

~~~

### 运行结果

~~~bash
$  trace 2 usertests forkforkfork
usertests starting
3: syscall sys_fork -> 4
test forkforkfork: 3: syscall sys_fork -> 5
5: syscall sys_fork -> 6
6: syscall sys_fork -> 7
7: syscall sys_fork -> 8
7: syscall sys_fork -> 9
6: syscall sys_fork -> 10
7: syscall sys_fork -> 11
8: syscall sys_fork -> 12
6: syscall sys_fork -> 13
6: syscall sys_fork -> 14
8: syscall sys_fork -> 15
7: syscall sys_fork -> 16
6: syscall sys_fork -> 17
8: syscall sys_fork -> 18
7: syscall sys_fork -> 19
6: syscall sys_fork -> 20
8: syscall sys_fork -> 21
6: syscall sys_fork -> 22
7: syscall sys_fork -> 23
7: syscall sys_fork -> 24
8: syscall sys_fork -> 25
6: syscall sys_fork -> 26
23: syscall sys_fork -> 27
23: syscall sys_fork -> 28
6: syscall sys_fork -> 29
7: syscall sys_fork -> 30
6: syscall sys_fork -> 31
7: syscall sys_fork -> 32
8: syscall sys_fork -> 33
6: syscall sys_fork -> 34
7: syscall sys_fork -> 35
8: syscall sys_fork -> 36
6: syscall sys_fork -> 37
9: syscall sys_fork -> 38
6: syscall sys_fork -> 39
22: syscall sys_fork -> 40
23: syscall sys_fork -> 41
9: syscall sys_fork -> 42
21: syscall sys_fork -> 43
6: syscall sys_fork -> 44
21: syscall sys_fork -> 45
6: syscall sys_fork -> 46
38: syscall sys_fork -> 47
38: syscall sys_fork -> 48
38: syscall sys_fork -> 49
38: syscall sys_fork -> 50
38: syscall sys_fork -> 51
7: syscall sys_fork -> 52
6: syscall sys_fork -> 53
9: syscall sys_fork -> 54
6: syscall sys_fork -> 55
7: syscall sys_fork -> 56
13: syscall sys_fork -> 57
24: syscall sys_fork -> 58
24: syscall sys_fork -> 59
OK
3: syscall sys_fork -> 60
ALL TESTS PASSED
~~~

## Sysinfo(Moderate)

~~~
In this assignment you will add a system call, sysinfo, that collects information about the running system. The system call takes one argument: a pointer to a struct sysinfo (see kernel/sysinfo.h). The kernel should fill out the fields of this struct: the freemem field should be set to the number of bytes of free memory, and the nproc field should be set to the number of processes whose state is not UNUSED. We provide a test program sysinfotest; you pass this assignment if it prints "sysinfotest: OK".
~~~

题目要求：

完成一个名为sysinfo的syste 

- freemem: 还可以使用的内存的字节数量
- nproc:进程状态**不为**UNUSED的状态的数量

完成后使用sysinfotest进行检测

Some hints:

- Add `$U/_sysinfotest` to UPROGS in Makefile

- Run make qemu; `user/sysinfotest.c` will fail to compile. Add the system call sysinfo, following the same steps as in the previous assignment. To declare the prototype for sysinfo() `in user/user.h` you need predeclare the existence of `struct sysinfo`:

  ​		

  ```C
      struct sysinfo;
      int sysinfo(struct sysinfo *);
  ```

​			Once you fix the compilation issues, run sysinfotest; it will fail because you haven't implemented the system call 			in the kernel yet.

- sysinfo needs to copy a `struct sysinfo` back to user space; see `sys_fstat()` (`kernel/sysfile.c`) and `filestat()` (`kernel/file.c`) for examples of how to do that using `copyout()`.

- To collect the amount of free memory, add a function to `kernel/kalloc.c`

- To collect the number of processes, add a function to `kernel/proc.c`

根据hint的提示，查看sys_fstat 和 filestat的代码

*sys_fstat：*

~~~C

uint64
sys_fstat(void)
{
  struct file *f;
  uint64 st; // user pointer to struct stat

  if(argfd(0, 0, &f) < 0 || argaddr(1, &st) < 0)
    return -1;
  return filestat(f, st);
}
~~~

*filestas:*

~~~C
// Get metadata about file f.
// addr is a user virtual address, pointing to a struct stat.
int
filestat(struct file *f, uint64 addr)
{
  struct proc *p = myproc();
  struct stat st;
  
  if(f->type == FD_INODE || f->type == FD_DEVICE){
    ilock(f->ip);
    stati(f->ip, &st);
    iunlock(f->ip);
    if(copyout(p->pagetable, addr, (char *)&st, sizeof(st)) < 0)
      return -1;
    return 0;
  }
  return -1;
}
~~~

*argfd:*

取得syscall传入的文件描述符

返回文件描述符以及文件的结构体

~~~C
// Fetch the nth word-sized system call argument as a file descriptor
// and return both the descriptor and the corresponding struct file.
static int
argfd(int n, int *pfd, struct file **pf)
{
  int fd;
  struct file *f;

  if(argint(n, &fd) < 0)
    return -1;
  if(fd < 0 || fd >= NOFILE || (f=myproc()->ofile[fd]) == 0)
    return -1;
  if(pfd)
    *pfd = fd;
  if(pf)
    *pf = f;
  return 0;
}
~~~

*argaddr*:

取得参数的地址

~~~C
// Retrieve an argument as a pointer.
// Doesn't check for legality, since
// copyin/copyout will do that.
int
argaddr(int n, uint64 *ip)
{
  *ip = argraw(n);
  return 0;
}
~~~



*copyout.c(vm.c):*

从内核复制到用户空间，复制n个字节从目的地址到虚拟地址（指定的page table）。

~~~C
// Copy from kernel to user.
// Copy len bytes from src to virtual address dstva in a given page table.
// Return 0 on success, -1 on error.
int
copyout(pagetable_t pagetable, uint64 dstva, char *src, uint64 len)
{
  uint64 n, va0, pa0;

  while(len > 0){
    va0 = PGROUNDDOWN(dstva);
    pa0 = walkaddr(pagetable, va0);
    if(pa0 == 0)
      return -1;
    n = PGSIZE - (dstva - va0);
    if(n > len)
      n = len;
    memmove((void *)(pa0 + (dstva - va0)), src, n);

    len -= n;
    src += n;
    dstva = va0 + PGSIZE;
  }
  return 0;
}

~~~

### 统计剩余内存的大小：

从kfree函数可以看出，freelist是一个链表，每一个节点表示可以使用的空的内存块，一块的大小是PAGESIZE个字节。

~~~C
// Physical memory allocator, for user processes,
// kernel stacks, page-table pages,
// and pipe buffers. Allocates whole 4096-byte pages.

#include "types.h"
#include "param.h"
#include "memlayout.h"
#include "spinlock.h"
#include "riscv.h"
#include "defs.h"

void freerange(void *pa_start, void *pa_end);

extern char end[]; // first address after kernel.
                   // defined by kernel.ld.

struct run {
  struct run *next;
};

struct {
  struct spinlock lock;
  struct run *freelist;
} kmem;

void
kinit()
{
  initlock(&kmem.lock, "kmem");
  freerange(end, (void*)PHYSTOP);
}

void
freerange(void *pa_start, void *pa_end)
{
  char *p;
  p = (char*)PGROUNDUP((uint64)pa_start);
  for(; p + PGSIZE <= (char*)pa_end; p += PGSIZE)
    kfree(p);
}

// Free the page of physical memory pointed at by v,
// which normally should have been returned by a
// call to kalloc().  (The exception is when
// initializing the allocator; see kinit above.)
void
kfree(void *pa)
{
  struct run *r;

  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    panic("kfree");

  // Fill with junk to catch dangling refs.
  memset(pa, 1, PGSIZE);

  r = (struct run*)pa;

  acquire(&kmem.lock);
  r->next = kmem.freelist;
  kmem.freelist = r;
  release(&kmem.lock);
}

// Allocate one 4096-byte page of physical memory.
// Returns a pointer that the kernel can use.
// Returns 0 if the memory cannot be allocated.
void *
kalloc(void)
{
  struct run *r;

  acquire(&kmem.lock);
  r = kmem.freelist;
  if(r)
    kmem.freelist = r->next;
  release(&kmem.lock);

    
  if(r)
    memset((char*)r, 5, PGSIZE); // fill with junk
  return (void*)r;
}
~~~

于是统计剩余内存大小的函数为:

~~~C
//统计还有多少可以使用的字节大小
uint64 acquire_free_mem()
{
   uint64  freeMemNum = 0;
   struct run* r;
   r = kmem.freelist;
   while(r)
   {
      freeMemNum++;
      r =r->next;
   }
   return freeMemNum*PGSIZE;
}
~~~



### 统计状态不为UNUSED的进程的数量

首先阅读proc.c文件

前面一行有如下代码：

~~~c
struct proc proc[NPROC];
~~~

这应该是将所有线程的一个数组（线程池？），后面这个创建线程的方法可以佐证这一点（在寻找state为UNUSED这个方法中可以看到）

~~~C
// Look in the process table for an UNUSED proc.
// If found, initialize state required to run in the kernel,
// and return with p->lock held.
// If there are no free procs, or a memory allocation fails, return 0.
static struct proc*
allocproc(void)
{
  struct proc *p;

  for(p = proc; p < &proc[NPROC]; p++) {
    acquire(&p->lock);
    if(p->state == UNUSED) {
      goto found;
    } else {
      release(&p->lock);
    }
  }
  return 0;

found:
  p->pid = allocpid();
  p->state = USED;

  // Allocate a trapframe page.
  if((p->trapframe = (struct trapframe *)kalloc()) == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }

  // An empty user page table.
  p->pagetable = proc_pagetable(p);
  if(p->pagetable == 0){
    freeproc(p);
    release(&p->lock);
    return 0;
  }

  // Set up new context to start executing at forkret,
  // which returns to user space.
  memset(&p->context, 0, sizeof(p->context));
  p->context.ra = (uint64)forkret;
  p->context.sp = p->kstack + PGSIZE;

  return p;
}
~~~

再看struct proc的结构

~~~C
// Per-process state
struct proc {
  struct spinlock lock;

  // p->lock must be held when using these:
  enum procstate state;        // Process state
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  int xstate;                  // Exit status to be returned to parent's wait
  int pid;                     // Process ID
 
  // wait_lock must be held when using this:
  struct proc *parent;         // Parent process

  // these are private to the process, so p->lock need not be held.
  uint64 kstack;               // Virtual address of kernel stack
  uint64 sz;                   // Size of process memory (bytes)
  pagetable_t pagetable;       // User page table
  struct trapframe *trapframe; // data page for trampoline.S
  struct context context;      // swtch() here to run process
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)
  
  //在该结构中存放trace中传入的掩码
  int mask;
};
~~~

~~~C
  // p->lock must be held when using these:
  enum procstate state;        // Process state
~~~

由上代码及注释可以知道，如果要访问一个线程的state，必须hold   p->lock，调用acquire(&p->lock)去实现，释放的话就是release(&p->lock);

代码如下：

~~~C

//统计state 不是UNUSED的进程的数量
//需要找到可以遍历所以线程的一个变量
uint64 acquire_unused_pro_num()
{
   struct proc *p;
   int unused_proc_num =0;
   for(p = proc ; p <  &proc[NPROC];p++)
   {
      acquire(&p->lock);
      if(p->state != UNUSED)
      {
        unused_proc_num++;
      }
      release(&p->lock);
   }
  
   return unused_proc_num;
}
~~~

### sys_sysinfo的代码部分

~~~C
uint64 sys_sysinfo(void)
{
  struct proc *p = myproc();
  struct sysinfo sf;
  uint64 infoAddr;//指向struct sysinfo的指针
  if(argaddr(0,&infoAddr) <0)return -1;
  // copyout(p->pagetable,infoAddr,);
  sf.freemem = acquire_free_mem();
  sf.nproc =acquire_unused_pro_num();

  if(copyout(p->pagetable,infoAddr,(char*)&sf,sizeof(sf)))
  return -1;
  return 0;
}
~~~

运行结果：

~~~bash
$ sysinfotest
sysinfotest: start
sysinfotest: OK
~~~







