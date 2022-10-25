---
title: const修饰成员函数
date: 2022-10-25 11:11:32
categories: C++
author: spikeHu
tags: 
    -C++primer
    -C++

---

# const修饰成员函数，以及Mutable

在类成员函数后面加上const关键字，表示成员函数不会修改该调用对象的成员变量。

<!--more-->

1. mutable可以突破const的限制，被mutable修饰的成员变量，永远处于可变的状态
2. 非const成员函数可以调用const成员函数和非const成员函数
3. const成员函数不能调用非const成员函数
4. 常对象只能访问加了const的成员函数
