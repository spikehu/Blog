---
title: C++基础知识
date: 2022-10-25 15:19:28
categories: C++
author: spikeHu
tags: 
    -C++primer
    -C++
---

# C++基础知识

## 静态成员变量

<!--more-->

- 类的静态成员包括静态成员变量和静态成员函数。
- 用static关键字把类的成员变量声明为静态，表示在程序中是共享的。
- 静态成员必须在好程序的全局区用代码清晰的初始化（用范围解析运算符::）
- 静态成员使用类名加范围解析运算符::就可以访问，不需要创建对象。
- 如果把类的成员声明为静态的，就可以把它与类的对象独立开来（静态成员不属于对象），存放在静态存储区。
- 静态函数只能访问静态成员。
- 静态函数没有this
- 私有静态成员在类外无法访问。
- const静态成员变量可以在定义类的时候初始化

### 对象和类-简单对象模型

C++有两种数据成员：nonstatic 、static，三种函数成员：nonstatic、static、virtual。



- 对象内存的大小包括：1）所有静态数据成员的大小；2）由内存对齐而填补的内存的内存大小；3）为了支持virtural成员而产生的额外负担。
- 静态成员变量属于类，不计算在对象大小之内
- 成员函数是分开存储的，不论对象是否存在都占用存储空间，在内存中只有一个副本，也不计算在对象大小之内
- 用空指针可以访问到没有用到this指针的非静态成员函数（就是这个成员函数没有用到成员变量就可以用空指针去访问）
- 对象的地址的第一个非静态成员变量的地址，如果类中没有非静态成员变量，编译器会隐含的增加一个1字节的占位成员

## 关系运算符重载

有6种关系运算符进行重载，最后以成员函数进行重载

- ==  、 != 、   > 、 <、 >= 、<= 

## 左移运算符重载

用于输出自定义对象的成员变量，只能使用非成员函数版本，如果要输出对象的私有变量，可以配合友元一起使用。

~~~
Ostream& operator <<(Ostream& cout , 传入的第二个参数)
~~~

## 重载下标运算符

必须以成员函数的形式进行重载

~~~c++
返回值类型 &operator[](参数)
或
const 返回值类型 &operator[](参数)const 
~~~

使用第二种是为了应对常对象的情况。

## 重载赋值运算符

 ~~~C++
 类名& operator=(const 类名& 对象 )
 ~~~

拷贝构造和赋值构造的不同：拷贝构造指原来的对象不存在，用已存在的对象进行构造；赋值构造是已经存在了两个对象，把其中一个对象的成员变量赋值给另一个对象的成员变量

## 使用类-重载new & delete运算符

重载的new和delete函数就算不加static关键字也是静态成员函数，不能访问非静态成员函数。

C++中，使用new时，编译器做了两件事情：

1. 调用标准坤函数operator new()分配内存
2. 调用构造函数初始化内存

~~~c++
void* operator new(size_t size)
~~~



使用delete时候做了两件事情

1. 先调用析构函数
2. 调用标准库函数operator delete()释放内存

~~~c++
void operator delete(void* ptr)
~~~

## 内存池

- 预先分配一大块的内存空间
- 提升分配和归还的速度
- 减少内存碎片
- initpool
- freepool



~~~C++
#include <iostream>

#include <cstring>

using namespace  std;
class Dog
{
    friend ostream & operator <<(const ostream &cout,const Dog& dog);
public:
    string m_name;
    int m_age;
    static char* m_pool;//内存池开始的位置

public:
    static bool initPool()
    {
        m_pool = (char*)malloc(58);
        if(m_pool == 0)return false;
        memset(m_pool,0,10);
        cout<<"the start of pool:"<<(void*)m_pool<<endl;
        return true;
    }
    static void freePool()
    {
        if(m_pool==0)return ;
        free(m_pool);
    }

    //重载new 和 delete
    void* operator new(size_t size)
    {
//        void* ptr = malloc(size);
//        return ptr;
        if(m_pool[0] == 0)
        {
            m_pool[0]=1;
            return m_pool+1;
        }
        if(m_pool[29] == 0)
        {
            m_pool[29] = 1;
            return m_pool+30;
        }
        void* ptr = (void*) malloc(28);
        return ptr;
    }
    void operator delete (void* ptr)
    {
        if(ptr== nullptr)return ;
        if(ptr==m_pool+1)
        {
            m_pool[0] = 0;
            return ;
        }
        if(ptr == m_pool+30)
        {
            m_pool[29] =0;
            return ;
        }
        free(ptr);
    }

    Dog(string dogName,int age)
    {
        m_age= age;
        m_name  = dogName;
        cout<<"constructor called"<<endl;
    }
    ~Dog()
    {
        cout<<"destructor called"<<endl;
    }


};
char * Dog::m_pool=0;
//重载<<
ostream & operator <<( ostream &cout,const Dog& dog)
{
    cout<<"dog`s name:"<<dog.m_name<<","<<"dog`s age:"<<dog.m_age;
}
int main() {

    if(Dog::initPool()== false)return -1;
    Dog* dog1 = new Dog("nicci",2);
    Dog* dog2 = new Dog("popi",3);
    cout<<*dog1<<"addr:"<<dog1<<endl;
    cout<<sizeof(*dog1)<<endl;
    cout<<*dog2<<"addr:"<<dog2<<endl;
    delete dog2;
    delete dog1;
    Dog* dog3 = new Dog("ppp",4);
    cout<<*dog3<<"addr:"<<dog3<<endl;
    Dog::freePool();
    return 0;
}

~~~

## 重载括号运算符

~~~
返回值类型 operator()(参数列表);
~~~

只能用类的成员函数重载。

函数对象的用途：

- 表面像函数，部分场景中可以代替函数，在STL中得到广泛应用
- 函数对象本质是类，可以用成员存放更过的信息
- 函数对象有自己的数据模型
- 可以提供继承体系

## 重载一元运算符

可重载的一元运算符

- ++自增，分为前置和后置，增加一个int形参就成了++后置的函数，后置返回临时对象

- --自减
- !逻辑非
- &取地址
- ~二进制反码
- *解引用
- +一元加
- -一元求反 

## 自动类型转换

构造函数，可以使用explicit强调不适用隐式的类型转换。

## 转换函数

构造函数只用于从某种类型到类类型的转换，如果进行相反的转换，使用特殊的运算符函数--转换函数

~~~c++
operator 数据类型();
~~~

转换函数必须是类的成员函数；不能指定返回值类型；不能有参数。

可以让编译器决定选择转换函数（隐式转换），可以像使用强制类型转换那样使用它们。（显示转换）

**也可以使用成员函数代替。（更好）**

## 类的继承

基类中的Private对于派生类是不可见的。

在派生类中，可以通过基类的共有成员函数间接访问基类的私有成员。

使用using关键字可以改变基类成员在派生类中的访问权限。

注意：using只能改变基类中public和protected成员的访问权限，不能改变private成员的访问权限，因为基类中private成员在派生类中是不可见的，无法使用 。

~~~~
public:
using 基类::基类成员变量
private:
using 基类：：基类成员变量
~~~~



## 类继承-继承的对象模型

1. 创建派生类对象时，先调用基类的构造函数，在调用派生类的构造函数。
2. 销毁派生类对象时，先调用派生类的析构函数，在调用基类的析构函数
3. 创建派生类对象时只会申请一次内存，派生类对象包含了基类对象的内存空间，this指针相同的
4. 创建派生类对象时，先初始化基类对象，再初始化派生类对象
5. 再VS中，用cl.exe可以查看类的内存模型
6. 对派生类对象用sizeof得到的是基类所有成隐患（包括私有成员）+派生类对象所有成员的大小
7. 在C++中，不同继承方式的访问权限知识语法上的处理
8. 对派生类对象用memset()会清空基类私有成员
9. 用指针可以访问到基类的私有成员（没有内存对齐，没有占位符）

## 如何构造基类

- 基类的成员变量由基类的构造函数初始化 ，派生类新增的成员变量由派生类的构造函数初始化

## 名字遮蔽

- 如果派生类中的成员和基类中的成员重名，通过派生类对象或者在派生类的成员函数中使用该成员时，将使用派生类新增的成员，而不是基类的。**注意：基类的成员函数和派生类的成员函数不会构成重载，如果派生类有同名函数，那么就会遮蔽基类中的所有同名函数**



## 继承的特殊关系

注意：

- 基类指针或引用只能调用基类的方法，不能调用派生类的方法
- 可以用派生类构造基类
- 如果函数的形参是基类，实参可以用派生类
- C++要求指针引用类型与赋给的类型匹配，这一规则对继承来说是例外。

## 多继承与虚继承

使用virtual关键字避免二义性和数据冗余

## 类多态-多态的基本概念

如果基类指针只能调用基类的成员函数，不能调用派生类的。

使用virtual关键字可以使得调用的是派生类的成员函数，通过派生类的成员函数还可以访问派生类的成员变量。

## 多态的内存模型

 

- 静态多态：编译时的多态；在编译时期确定要执行的函数地址；主要有函数重载和函数模板。
- 动态多态：即动态绑定，在运行时采取确定对象类型和正确选择需要调用的函数，一般用于解决基类指针或引用派生类对象调用类中重写的方法（函数）时出现的问题。

## 如何析构派生类

- 在生成过程中，一般是先调用基类的构造函数，在调用派生类的构造函数，销毁时候就是先调用派生类的析构函数，在调用基类的析构函数。
- 如果时多态的话，基类就要提供一个虚析构函数
- 在析构函数中销毁内存时，记得将指针指向空

## 纯虚函数和抽象类

- virtual 返回值类型 函数名（参数列表）=0
- 纯虚析构函数必须实现
