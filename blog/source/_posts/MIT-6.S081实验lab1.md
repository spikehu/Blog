---
title: MIT-6.S081实验lab-1
date: 2022-10-03 09:25:00
author: spikeHu
cover: true
coverImg: /images/cover1.jpg
summary: lab1的实现以及如何进行测试
categories: MIT-OS
tags:
  - MIT-OS
  - LAB1
---


# MIT 6.S081实验 lab1

### 前言：测试时候注意事项

### 1.完成程序，添加配置

在完成程序之后，需要在Makefile中添加配置，这下面添加的是copy.c
<!--more-->

![image-20220930222707955](typora-user-images\image-20220930222707955.png)

## 2.如何进行测试

进行程序的测试的时候需要将qemu退出，然后测试，在linux终端中进行如下测试：

~~~
./grade-lab-util pingpong
~~~

## Lab-1

### lab-1.1 sleep函数

~~~
Implement the UNIX program sleep for xv6; your sleep should pause for a user-specified number of ticks. (A tick is a notion of time defined by the xv6 kernel, namely the time between two interrupts from the timer chip.) Your solution should be in the file user/sleep.c.
~~~

**Some hints:**

- Look at some of the other programs in `user/` to see how you can obtain the command-line arguments passed to a program. If the user forgets to pass an argument, sleep should print an error message.
- The command-line argument is passed as a string; you can convert it to an integer using `atoi` (see user/ulib.c).
- Use the system call `sleep` (see user/usys.S and kernel/sysproc.c).
- Make sure `main` calls `exit()` in order to exit your program.
- Add the program to `UPROGS` in Makefile and compile user programs by typing `make fs.img`.
- Look at Kernighan and Ritchie's book *The C programming language (second edition)* (K&R) to learn about C.

编写如下sleep.c 文件

~~~c

#include "kernel/types.h"
#include"user/user.h"

int main(int argc , char* argv[])
{
    if(argc==0||argc>2)
    {
         printf("argument number error(Usage: sleep number(integer)) \n");
         exit(1);
    }
    int sleepTime = atoi(argv[1]);
    sleep(sleepTime);
    exit(1);
}
~~~

进行测试

![image-20220930223202780](typora-user-images\image-20220930223202780.png)

### lab-1.2 pingpong

~~~
Write a program that uses UNIX system calls to ``ping-pong'' a byte between two processes over a pair of pipes, one for each direction. The parent sends by writing a byte to parent_fd[1] and the child receives it by reading from parent_fd[0]. After receiving a byte from parent, the child responds with its own byte by writing to child_fd[1], which the parent then reads. Your solution should be in the file user/pingpong.c.
~~~

- Use `pipe` to create a pipe.
- Use `fork` to create a child.
- Use `read` to read from the pipe, and `write` to write to the pipe.

Run the program from the xv6 shell and it should produce the following output:

```
   $ make qemu
    ...
    init: starting sh
    $ pingpong
    4: received ping
    3: received pong
    $
  
```

Your solution is correct, if your program behaves as shown above. The number before ":" is the process id of the process printing the output. You can get the process id by calling the system call `getpid`.

**dup：重新生成一个文件描述符指向oldfd指向的文件，且取当前可以取的最小值**

~~~
The dup() system call creates a copy of the file descriptor oldfd, using the lowest-numbered unused file descriptor for the new descriptor.

       After a successful return, the old and new file descriptors may be used interchangeably.  They refer to the same open file description (see open(2)) and thus share file offset and file status
       flags; for example, if the file offset is modified by using lseek(2) on one of the file descriptors, the offset is also changed for the other.

       The two file descriptors do not share file descriptor flags (the close-on-exec flag).  The close-on-exec flag (FD_CLOEXEC; see fcntl(2)) for the duplicate descriptor off.
~~~

代码实现：

~~~C
#include "kernel/types.h"
#include"user/user.h"
//The parent sends by writing a byte to parent_fd[1] 
//the child receives it by reading from parent_fd[0]. 
// After receiving a byte from parent, the child responds with its own byte by writing to child_fd[1], which the parent then reads. 

#define BUF_SIZE 1024

int main(int argc ,char* argv[])
{
  
    if(argc>1)
    {
        printf("too much argument(Usage: pingpong)");
        exit(1);
    }
    int parent_fd[2];//a pair if pipes that parent send the message
    pipe(parent_fd);

    int child_fd[2];//child process send message to 
    pipe(child_fd);

    char buf[BUF_SIZE];
    //create a process 
    if( fork() ==0)//child process
    {

        close(parent_fd[1]);
        close(child_fd[0]);
        //read from parent process
        if(read(parent_fd[0],buf,BUF_SIZE)==-1)exit(1);
        if(sizeof(buf) == 0)
        {
            printf("receive no byete");   
            exit(1);
        }
        printf("%d: received %s",getpid(),buf);
        if(write(child_fd[1],"pong\n",5)==-1)exit(1);
    }else
    {
        //parent process
        //write byte to the pipe
        close(parent_fd[0]);
        close(child_fd[1]);
        if(write(parent_fd[1],"ping\n",5)==-1)exit(1);
        if(read(child_fd[0],buf,BUF_SIZE)==-1)exit(1);
        printf("%d: received %s",getpid(),buf);
    }
    
    exit(1);
}
~~~

### lab-1.3 primes

~~~
Write a concurrent version of prime sieve using pipes. This idea is due to Doug McIlroy, inventor of Unix pipes. The picture halfway down [this page](http://swtch.com/~rsc/thread/) and the surrounding text explain how to do it. Your solution should be in the file `user/primes.c`.
~~~

#### 1.3.1前置知识

阅读[Bell Labs and CSP Threads](https://swtch.com/~rsc/thread/)。

需要掌握的模型如下：

![image-20221004142714644](typora-user-images\image-20221004142714644.png)



题目要求：

Your goal is to use `pipe` and `fork` to set up the pipeline. The first process feeds the numbers 2 through 35 into the pipeline. For each prime number, you will arrange to create one process that reads from its left neighbor over a pipe and writes to its right neighbor over another pipe. Since xv6 has limited number of file descriptors and processes, the first process can stop at 35.

Some hints:

- Be careful to close file descriptors that a process doesn't need, because otherwise your program will run xv6 out of resources before the first process reaches 35.
- Once the first process reaches 35, you should arrange that the pipeline terminates cleanly, including all children (Hint: read will return an end-of-file when the write-side of the pipe is closed).
- It's simplest to directly write 32-bit `int`s to the pipes, rather than using formatted ASCII I/O.
- You should create the processes in the pipeline as they are needed.

​	

Your solution is correct if it produces the following output:

```shell
  $ make qemu
    ...
    init: starting sh
    $ primes
    prime 2
    prime 3
    prime 5
    prime 7
    prime 11
    prime 13
    prime 17
    prime 19
    prime 23
    prime 29
    prime 31
    $
```

题目大意就是通过一个线程将处理一组数据，将第一个数字num输出，后面可以被num整除的数字舍弃，否则传入下一个子进程，两个进程间使用管道进行通信。

代码如下：

~~~C
#include"kernel/types.h"
#include"user/user.h"
#define WR 1
#define RD 0

const int INT_SIZE = sizeof(int);
void primes(int* p)
{
    close(p[WR]);
    //从左管道读取数据
    int firstNum = 0;
    if(read(p[RD],&firstNum,INT_SIZE)!=INT_SIZE)exit(1);

    printf("prime %d\n",firstNum);
    //读取管道中剩余的数字 将符合要求的数字写入下一个管道
    int data = 0; 
    int p2[2];
    pipe(p2);
    while(read(p[RD],&data,INT_SIZE)==INT_SIZE)
    {
        if(data%firstNum)write(p2[WR],&data,INT_SIZE);
    }
    close(p[RD]);
    if(fork()==0)
    {
        primes(p2);
    }else
    {
        close(p2[WR]);
        wait(0);
    }
     exit(1);
}
int main(int argc , char* argv[]) {
    int fd[2];
    pipe(fd);
    for(int i = 2 ; i<=35;i++)
    {
        write(fd[WR],&i,INT_SIZE);
    }
    if(fork()==0)//子进程 对数据进行处理
    {
        primes(fd);
    }else
    {
        close(fd[1]);
        close(fd[0]);
        wait(0);
    }
    exit(1);    
}

~~~

测试结果如下：

![image-20221005095836273](typora-user-images\image-20221005095836273.png)

### lab-1.4 find

~~~
Write a simple version of the UNIX find program: find all the files in a directory tree whose name matches a string. Your solution should be in the file user/find.c.
~~~

Some hints:

- Look at user/ls.c to see how to read directories.
- Use recursion to allow find to descend into sub-directories.
- Don't recurse into "." and "..".
- Changes to the file system persist across runs of qemu; to get a clean file system run make clean and then make qemu.
- You'll need to use C strings. Have a look at K&R (the C book), for example Section 5.5.

Optional: support regular expressions in name matching. `grep.c` has some primitive support for regular expressions.

Your solution is correct if produces the following output (when the file system contains a file `a/b`):

```shell
  $ make qemu
    ...
    init: starting sh
    $ mkdir a
    $ echo > a/b
    $ find . b
    ./a/b
    $ 
  
```

#### 1.4.1前置知识

##### *strut stat*

~~~
用来描述linux文件系统中的文件属性的结构
~~~



在vx6中 stat.h：

~~~C
#define T_DIR     1   // Directory
#define T_FILE    2   // File
#define T_DEVICE  3   // Device

struct stat {
  int dev;     // File system's disk device
  uint ino;    // Inode number
  short type;  // Type of file
  short nlink; // Number of links to file
  uint64 size; // Size of file in bytes
};

~~~

##### fstat函数

~~~C
int fstat(int fd, struct stat *statbuf);
~~~

**description:**

~~~
fstat() is identical to stat(), except that the file about which information is to be retrieved is specified by the file descriptor fd.即通过指针获取文件信息。
~~~

##### struct dirent

~~~
LINUX系统下的一个头文件,在这个目录下/usr/include
为了获取某文件夹目录内容，所使用的结构体。
~~~

在vx6中的结构如下：

~~~c
struct dirent {
  ushort inum;
  char name[DIRSIZ];
};
~~~

##### VX6中ls.c源码

~~~C
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

char*
fmtname(char *path)
{
  static char buf[DIRSIZ+1];
  char *p;

  // Find first character after last slash.
  for(p=path+strlen(path); p >= path && *p != '/'; p--)
    ;
  p++;

  // Return blank-padded name.
  if(strlen(p) >= DIRSIZ)
    return p;
  memmove(buf, p, strlen(p));
  memset(buf+strlen(p), ' ', DIRSIZ-strlen(p));
  return buf;
}

void
ls(char *path)
{
  char buf[512], *p;//文件名
  int fd;//文件描述符
  struct dirent de;//linux 目录信息数据结构
  struct stat st;//文件信息数据结构

  if((fd = open(path, 0)) < 0){
    fprintf(2, "ls: cannot open %s\n", path);
    return;
  }

  if(fstat(fd, &st) < 0){
    fprintf(2, "ls: cannot stat %s\n", path);
    close(fd);
    return;
  }

  switch(st.type){
  case T_FILE:
    printf("%s %d %d %l\n", fmtname(path), st.type, st.ino, st.size);
    break;

  case T_DIR:
    if(strlen(path) + 1 + DIRSIZ + 1 > sizeof buf){
      printf("ls: path too long\n");
      break;
    }
    strcpy(buf, path);
    p = buf+strlen(buf);
    *p++ = '/';
    while(read(fd, &de, sizeof(de)) == sizeof(de)){
      if(de.inum == 0)//inum 都是从1开始
        continue;
      memmove(p, de.name, DIRSIZ);
      p[DIRSIZ] = 0;
      if(stat(buf, &st) < 0){
        printf("ls: cannot stat %s\n", buf);
        continue;
      }
      printf("%s %d %d %d\n", fmtname(buf), st.type, st.ino, st.size);
    }
    break;
  }
  close(fd);
}

int
main(int argc, char *argv[])
{
  int i;

  if(argc < 2){
    ls(".");
    exit(0);
  }
  for(i=1; i<argc; i++)
    ls(argv[i]);
  exit(0);
}

~~~

要求就是打印出指定目录下文件的所有路径。

##### 1.4.2代码

~~~c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"
//p:文件名字的末尾
//fil:文件名
int myStrcmp(char* p , char* fil)
{
    while(*p!='/')p--;
    p++;
    return strcmp(p,fil);
}

//find all the files in a directory tree whose name matches a string. Your solution should be in the file user/find.c.
void  find(char* path ,char* filename)
{
    //p指针指向文件名字(buf)的最后一个字符
    char buf[512] , *p;
    int fd ;
    struct stat st;//文件信息结构
    struct dirent de; //目录的信息结构


    memset(buf,0,sizeof(buf));
    strcpy(buf,path);
    p = buf+strlen(buf);
    

   
    //先打开文件目录
    if((fd = open(path,0)) < 0)
    {
        printf("can not open %s\n",path);
        return ;
    }
    
    //在获取文件信息
    if(fstat(fd,&st)<0)
    {
        printf("can not stat %s\n",path);
        close(fd);
        return ;
    }
   

    //根据path 是文件还是目录进行不同的处理
    switch(st.type)
    {
        case T_FILE:
        //如果是文件 就看文件的名字和查找的文件名字是否一致 一致的话打印出该文件的路径 
        //由于path中包含目录 需要将其去掉只比较文件名
        if(myStrcmp(p,filename)==0)printf("%s\n",path);
        break;
        case T_DIR:
        //进行目录相关的处理
        //读取目录中的所有文件
        *p++= '/';
        while(read(fd,&de,sizeof(de))==sizeof(de))
        {
            
            
            if(de.inum == 0 ||de.name[0]=='.')continue;
            
            //在buf后面加上遍历到的文件的名字
           
           
            memmove(p,de.name,DIRSIZ);
            p[DIRSIZ] = 0;
            //最后buf 就是该目录下的一个子文件
            //使用递归进行再次比较
            // printf("directory:%s\n",buf);
            find(buf,filename);
        }
        break;
    }
    close(fd);
}
int main(int argc , char* argv[])
{
    //需要从main中获取3个参数 一个命令 一个文件目录 一个需要查找的文件名
    if(argc !=3)
    {
        printf("wrong argument,Usage: find [directory] [filename]");
        exit(1);
    }
    //输入文件名 进行查找 并将相关信息进行打印
    find(argv[1],argv[2]);
    exit(1);
}

~~~



![image-20221005193325550](typora-user-images\image-20221005193325550.png)

### lab1.5 Xargs

~~~
Write a simple version of the UNIX xargs program: read lines from standard input and run a command for each line, supplying the line as arguments to the command. Your solution should be in the file user/xargs.c.
~~~

The following example illustrates xarg's behavior:

```shell
 $ xargs echo bye
    hello too
    bye hello too
    ctrl-d
    $
```

Note that the command here is "echo bye" and the additional arguments are "hello too", making the command "echo bye hello too", which outputs "bye hello too".

Some hints:

- Use `fork` and `exec` system call to invoke the command on each line of input. Use `wait` in the parent to wait for the child to complete running the command.
- Read from stdin a character at the time until the newline character ('\n').
- kernel/param.h declares MAXARG, which may be useful if you need to declare an argv.
- Changes to the file system persist across runs of qemu; to get a clean file system run make clean and then make qemu.

xargs, find, and grep combine well:

```shell
  $ find . b | xargs grep hello
```

will run "grep hello" on each file named b in the directories below ".".

To test your solution for xargs, run the shell script xargstest.sh. Your solution is correct if it produces the following output:

```bash
 $ make qemu
  ...
  init: starting sh
  $ sh < xargstest.sh
  $ $ $ $ $ $ hello
  hello
  hello
  $ $   
```

You may have to fix bugs in your find program. The output has many `$` because the xv6 shell is primitive and doesn't realize it is processing commands from a file instead of from the console, and prints a `$` for each command in the file.

#### 1.5.1前置知识

##### 管道符

~~~bash
cmdA | cmdB
~~~

命令行cmdA的输出会作为cmdB的输入。

##### xargs

~~~bash
cmdA| xargs cmdB
~~~

将xargs搭配管道符使用，cmdA的输出会作为cmdB的一个参数进行执行。

~~~C
#include "kernel/types.h"
#include "user/user.h"
#include"kernel/param.h"
//Write a simple version of the UNIX xargs program: read lines from standard input 
//and run a command for each line, supplying the line as arguments to the command. 
//配合管道符使用：cmdA|cmdB 将cmdA的输出作为cmdB命令的参数 然后执行 cmdB 
int main(int argc ,char* argv[])
{
    if(argc==1)
    {
        printf("too few argument,usage:xargs [command]...");
    } 
    char buf[MAXARG];
    char *newArgv[MAXARG+1];
    while(1)
    {
        //将cmdA执行的输出进行读取
    //需要舍弃换行符
    int index = 0; 
    while(read(0,&buf[index],sizeof(char))>0 && buf[index] !='\n')
    {
        index++;
    }
    if(index == 0)break;
    //将换行符 舍弃
    buf[index]  = 0;
    //argv[0]是 xargs
    //argv[1] :是指令
    for(int i = 1; i < argc;i++)
    {
         newArgv[i-1] = argv[i];
    }
    newArgv[argc]= 0;
    newArgv[argc-1] =buf;

    //执行
    if(fork()==0)
    {
        exec(argv[1],newArgv);
        printf("exec error.\n");
        exit(0);
    }else
    {
        wait(0);
    }
    }
    exit(1);

}   
~~~

运行结果：

~~~bash
make: 'kernel/kernel' is up to date.
== Test xargs == xargs: OK (6.1s) 
~~~







