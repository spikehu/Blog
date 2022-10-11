---
title: XV6的运行以及退出
date: 2022-10-05 09:25:00
author: spikeHu
cover: true
coverImg: /images/cover1.jpg
summary: XV6的一些操作讲解
categories: MIT-OS
tags:
  - MIT-OS
  - XV6
---

# XV6的运行以及退出

## xv6的运行

进入到xv6项目文件夹下，输入make qemu命令运行，运行后出现一下信息
<!--more-->

~~~
xv6 kernel is booting
hart 1 starting
hart 2 starting sh
~~~

## xv6的推出

ctrl + a ，按c回到monitor界面，如果按x推出QEMU.

 WARNING: The scripts cmake, cpack and ctest are installed in '/home/ubuntu/.local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.