---
title: Geekband C++面向对象高级编程 第五周作业
date: 2016-03-05 22:00
tags: Cpp
---

## 题目说明：

为上周题目中的 Fruit和Apple 添加 构造函数与 析构函数，并在构造函数与析构函数中打印控制台信息，观察构造和析构调用过程。然后为Apple类重载::operator new和::operator delete，在控制台打印信息，并观察调用结果。


<!--more-->


## 解答

原始提交代码：[Geekband-cpp-w5.rar](\Geekband-cpp-w5.rar)

附件2个文件夹，1是添加完构造函数和析构函数后，调用测试；2是添加了operator new、delete后的调用测试。由于题目里面::有一种全局的感觉，所以Apple类和全局的重载都写了，写完想想既然都写了，那么就把数组的也写一下（正好可以观察下第三周做法的内部实现）。

第一部分：测试了4种创建对象的方式：
1. 在栈区创建Fruit对象，创建时调用了Fruit类的构造函数，销毁时调用Fruit类的析构函数。
2. 在堆区创建Fruit对象，创建时会先分配空间，然后调用Fruit类的构造函数，并把指针传回，控制台只能显示调用构造函数这一步。销毁时会先调用析构函数，然后再释放内存，控制台只能显示调用析构函数这一步。
3. 在堆区创建Apple对象，并用父类指针指向它（upcast），这样构造函数的调用次序依次是父类构造函数、子类构造函数。析构时先调用子类析构函数，再调用父类析构函数。
4. 在栈区创建Apple对象，调用构造函数时，先父类再子类，调用析构函数时，先子类再父类。

第二部分：重载`operator new`和`operator delete`后观察调用结果：
1. 同前。 
2. 同前。
3. 在堆区创建Fruit对象，会调用全局new、delete运算符。
4. 在堆区创建Apple对象，并用父类指针指向它（upcast），会调用Apple类的new、delete运算符。
5. 在堆区创建Apple类数组，大小为5个sizeof(Apple)加上一个指针的大小，且后被创建的对象先被销毁。
6. 在创建时强制调用全局的new运算符，在销毁时由于没有要求强制调用全局delete运算符，Apple类的delete运算符被调用了。为了保持一致性，应该在delete前面也加上::
7. 为了呼应第三周的作业而写的测试代码，首先在堆区创建一个数组，每个元素是一个Fruit类型的指针，大小是5×指针大小，20（32位）或40（64位）。再创建5个Apple类型对象，以父类指针指向它们（upcast），会调用Apple类的new分配空间，父类构造函数，子类构造函数，把指针传回。销毁时先调用子类析构函数，父类析构函数，再销毁内存，最后调用全局delete运算符释放分配给指针数组的空间。

关于本次作业的思考：
1. 结合第4周的作业，我们计算出了sizeof(Fruit)和sizeof(Apple)，我觉得此题的第一个目的是让我们在调用的时候知道每次new和delete型别X的对象实际上是去呼叫operator new(sizeof(X))，delete时会据此释放对应的空间大小(sizeof(X))。同样的申请一个含N个X型别的数组，就会呼叫opreator new(N*sizeof(X))。
2. 这道题目的另一个出题目的是让我们了解为什么基类需要有虚析构函数。假如我将提交的代码做小小改动，基类和子类的析构函数去掉virtual，为Fruit类写一个operator delete（为了打印delete内存大小），就会得到这样的结果（运行环境Win10 TDM-GCC 4.9.2 32-bit）：
Apple::new(), size = 32, return: 0xc621a0
default Fruit ctor.this = 0xc621a0
default Apple ctor.this = 0xc621a0
Fruit dtor.this = 0xc621a0
Fruit::delete(), ptr = 0xc621a0, size = 24
很明显的，内存泄漏了。因为当使用upcast时，传给delete的大小是sizeof(Fruit)（基类的），并不是sizeof(Apple)，为了得到正确的大小，必须将基类的析构函数设置为虚析构函数。
3. 我在看《C++ Primer》和《The C++ Programming Language》发现了一点不一样。前者在做delete的时候是直接free(p)的（就像我提交的代码一样），但是后者对p做了判断，截图如下：
![TCPL P547.png](\images\TCPL P547.png)
![TCPL P557.png](\images\TCPL P557.png)
我用黄色高亮标注的if(p)，如果单纯从free(p)这个操作来讲，是不需要再判断p是否非空的，不知道此处的用意如何。比如说p指向了别的程序内存，或者系统内存，那么delete就会出现严重后果，但这个时候p也不是空，照样会执行。系统默认的delete是否有机制去避免这样的情况。

点评中提到这道题目的本意是重载Apple类里的new和delete，一般做工程时也不会去“重载”全局new和delete，在Apple类里面可以调用全局的new和delete。.h文件可以改成如下：

``` cpp
#ifndef __FRUIT_H__
#define __FRUIT_H__ 

#include <iostream>
using namespace std;
//----------------------------------------------------------------  
class Fruit {
	int no;
	double weight;
	char key;
public:
	Fruit() {
		cout << "default Fruit ctor.this = " << this << endl;
	}
	virtual ~Fruit() {
		cout << "Fruit dtor.this = " << this << endl;
	}
	virtual void process() {
	}
	void print() {
	}
};
//----------------------------------------------------------------  
class Apple: public Fruit {
	int size;
	char type;
public:
	Apple() {
		cout << "default Apple ctor.this = " << this << endl;
	}
	virtual ~Apple() {
		cout << "Apple dtor.this = " << this << endl;
	}
	virtual void process() {
	}
	void save() {
	}
	static void* operator new(size_t size);
	static void operator delete(void* ptr, size_t size) noexcept;
	static void* operator new[](size_t size);
	static void operator delete[](void* ptr, size_t size) noexcept;
};

inline
void* Apple::operator new(size_t size) {
	if (void* ptr = ::operator new(size)) {
		cout << "Apple::new(), size = " << size << ", return: " << ptr << endl;
		return ptr;
	} else {
		throw bad_alloc();
	}
}

inline
void Apple::operator delete(void* ptr, size_t size) noexcept {
	cout << "Apple::delete(), ptr = " << ptr << ", size = " << size << endl;
	::operator delete(ptr);
}

inline
void* Apple::operator new[](size_t size) {
	if (void* ptr = ::operator new(size)) {
		cout << "Apple::new[](), size = " << size << ", return: " << ptr
				<< endl;
		return ptr;
	} else {
		throw bad_alloc();
	}
}

inline
void Apple::operator delete[](void* ptr, size_t size) noexcept {
	cout << "Apple::delete[](), ptr = " << ptr << ", size = " << size << endl;
	::operator delete(ptr);
}
//----------------------------------------------------------------  

#endif  //fruit.h
```

另前面提到的第三点，在TCPL里面也提到了，这里的delete是针对正常new出来内存做处理，如果自己随便给指针一个地址（悬浮指针），可能就会造成无法估计的错误。李老师提出可能p非nullptr的时候会做其他处理，而nullptr不需要，这也是一种可能。