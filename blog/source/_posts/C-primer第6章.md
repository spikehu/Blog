---
title: C++primer第6章
date: 2022-10-19 15:36:16
categories: C++
author: spikeHu
tags: 
    -C++primer
    -C++
---

# 第6章 函数

## 局部对象

### 自动对象

形参是一种自动对象，函数一旦终止，形参就被销毁。我们用传递给函数的实参对象初始化形参对应的自动对象。如果变量定义本身含有初始值，就用这个初始值进行初始化；否则执行默认初始化。 

### 局部静态对象

局部静态对象的生命周期贯穿函数调用及之后的时间。**局部静态对象**在程序的执行路径第一次经过对象定义语句时初始（后面就会跳过这个初始化语句），知道程序终止才被销毁。

<!--more-->

## 分离式编译

分离式编译允许我们将程序分割道几个文件中去，每个文件单独编译。

### 编译和链接多个源文件

在名为factMain.cc文件中创建main函数，main函数将用到fact函数，在fact.cc中。如果只修改了其中一个源文件，只需要重新编译那个改动了的文件。

![image-20221019160717804](../typora-user-images/image-20221019160717804.png)

## 参数传递

这个比较简单，引用传递，传递指针，传递值。

### 一些思考

- **在一个函数返回值的时候（比如一个类或者其他的），在调用它的程序中得到的还是那个返回的值吗？**
- **在向一个函数进行值传递的时候，那么在函数中进行修改，原来本身传递进去的值是不会被修改的，所以如果传递的是一个类，那么是会生成一个类吗**？

~~~C++
using namespace  std;
class  TT
{
public:
    TT(int a)
    {
        cout<<this<<" constructor"<<endl;
    }
    TT& operator=(const TT t)
    {
        cout<<"operator ="<<endl;
    }
    ~TT()
    {
        cout<<this<<" destructor"<<endl;
    }

};
TT getClass(TT T2)
{
    cout<<"getClass T2 "<<&T2<<endl;
    return T2;
}
int main(int argc , char* argv[]) {
    TT T2(2);

    TT T3 = getClass(T2);
    cout<<"main T2 "<<&T2<<endl;
    cout<<"main T3 "<<&T3<<endl;

    exit(1);
}

~~~

得到结果

~~~
0x61ff0e constructor
getClass T2 0x61ff0f
0x61ff0f destructor
main T2 0x61ff0e
main T3 0x61ff0d
~~~

可以知道值传递的话，传入的类和接收的类不是一个类。且函数局部变量会销毁。这里调用的就是拷贝构造。

### 6.2.3 const和实参

虽然C++有函数重载但是会忽略顶层const，如

~~~
void fcn(const int i ){}
void fcn(int i){}
~~~

这两个就是错误的，相当于重复定了

#### 尽量使用常量引用

防止不必要的修改，以及不能将const对象、字面值或者需要类型转换的对象传递给普通的参数引用。

### 6.2.4 数组形参

![image-20221020153658802](../typora-user-images/image-20221020153658802.png)

![image-20221020153709006](../typora-user-images/image-20221020153709006.png)

### 6.2.5处理命令行选项

main函数中argc表示传入命令参数的个数，argv就是指向C风格字符串的指针数组。

### 6.2.6含有可变形参的函数

有时无法提前预知应该向函数传递几个实参。C++提供了2中方法：

- 名为initializer_list的标准库模板；
- 如果实参类型不同，可以写一种特殊的函数，16.4节将展开介绍
- 还有一种特殊的形参类型即省略符，用其传递可变数量的实参。一般只用于C函数交互的接口程序。

#### initializer_list形参

函数实参数量未知但是类型相同，可以使用initializer_list类型的实参。initializer_list定义在同名的头文件中。

![image-20221020155039333](../typora-user-images/image-20221020155039333.png)

initializer_list对象中的元素永远是常量值，我们无法进行修改。如果想向一个initializer_list形参传递一个值得序列，则必须把序列放在一对花括号内：

~~~c
//expected 和actual是string对象
if(expected != actual)
	error_msg({"functionX",expected,actual});
else
error_msg({"functionX","okay"});
~~~

含有initializer_list形参的函数也可以含有其他形参，如：

![image-20221020155815848](../typora-user-images/image-20221020155815848.png)

#### 省略符形参

为了便于C++程序访问某些特殊的C代码设置，这些代码 使用了名为varargs的C标准库功能。

![image-20221020160045115](../typora-user-images/image-20221020160045115.png)

### 6.3.2返回类型和return语句

返回值类型必须与函数的返回类型相同，或者能够隐式的转换成函数的返回类型。

#### 值是如何被返回的

返回一个值的方式和初始化一个变量或形参的方式完全一样：返回的值用户初始化调用点的一个临时量，该临时量就是函数调用的结果。

![image-20221020215003472](../typora-user-images/image-20221020215003472.png)

该函数的返回类型是string,意味着返回值将被**拷贝**到调用点。因此该函数将返回word的副本和一个未命名的临时string对象，该对象的内容是word和ending的和。

![image-20221020215224480](../typora-user-images/image-20221020215224480.png)

*如果返回的是引用*：形参和返回值都是const string的引用，所以不会真正拷贝string对象。

#### 不要返回局部对象的引用或指针

函数完成，占用的存储空间也会被释放，指向就不再有效。

#### 返回类类型的函数和调用运算符

#### 引用返回左值

函数返回类型决定函数调用是否是左值，调用一个返回引用的函数得到左值，其他的得到右值。

- 什么是左值：可以取地址的，有名称的，非临时的是左值
- 不可以取地址，匿名的，临时的是右值

也就是说如果返回的不是引用，那么返回值会被销毁。

#### 列表初始化返回值

C++11新标准规定，函数可以返回花括号包围的值列表。此处列表表示用来表示对返回的临时变量进行初始化。如果为空执行值初始化；否则有返回函数的返回类型决定。
