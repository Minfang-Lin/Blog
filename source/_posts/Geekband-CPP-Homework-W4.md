---
title: Geekband C++面向对象高级编程 第四周作业
date: 2016-02-08 11:19
tags: Cpp
---

## 题目说明：

分别给出下面的类型Fruit和Apple的类型大小（即对象size），并通过画出二者对象模型的方式来解释该size的构成原因。

``` cpp
class Fruit {
	int no;
	double weight;
	char key;
public:
	void print() { }
	virtual void process() { }
};
  
class Apple: public Fruit {
	int size;
	char type;
public:
	void save() { }
	virtual void process() { }
};
```

<!--more-->

## 解答
注：析构函数设置为虚函数并不影响sizeof结果，只是在vtbl里面多增加一项。
观察这道题目主要有两点需要考虑。第一，当类里面存在虚函数时，这个类所占的内存就会比没有虚函数时候大一点，大的位置是在类的成员变量前面会多出一个指针（vptr），它指向虚指针表（vtbl），虚指针表里面的每一个指针再指向对应的虚函数。从而实现动态绑定；第二，在C语言介绍struct时，就说过一点，大多数计算机，数据项要求从某个数量字节的倍数开始存放，如short从偶数地址开始，int则被对齐在4字节边界。为了满足内存对齐，在比较小的成员后面会加入补位。在用不同的操作系统和编译器时也发现了，sizeof的结果有所不同，所以这道题目并没有正确的答案，说明所使用的操作系统和某种编译器下的情形就可以了。首先画出<s>对象模型图</s>示意图：
![Class Diagram.png](\images\Class Diagram.png)
<s>因为我不会画UML类图，</s>大概就是这个意思，<s>虚函数表的指针（vptr）在内存中会出现在其他所有成员之前</s>，C++语言规范明确定义了内存上的成员变量的顺序和代码定义时的顺序是一致的（为了保证与C语言兼容）。正因为存在这样的顺序，所以在初始化子类的时候，会先初始化父类的成员变量，再初始化子类的。对象切割（Object Slicing）也可以顺利进行。

vptr的位置在规范中没有确定。当然我们可以去测试一下看看vptr到底在什么位置。测试代码如下：

``` cpp
#include <iostream>
using namespace std;

class Fruit {
	int no = 10;
	double weight;
	char key;
public:
	void print() { }
	virtual void process() { }
};

class Apple: public Fruit {
	int size;
	char type;
public:
	void save() { }
	virtual void process() { }
};

int main(int argc, char const *argv[]) {
	Fruit f1, f2;
	Apple a1;

	int* pf1 = (int*) &f1;
	int* pf2 = (int*) &f2;
	int* hpf1 = (int*) *pf1;
	int* hpf2 = (int*) *pf2;
	cout << *pf1 << endl << *(pf1 + 1) << endl;
	cout << *pf2 << endl << *(pf2 + 1) << endl;
	cout << hpf1 << endl;

	int* pa1 = (int*) &a1;
	int* hpa1 = (int*) *pa1;
	cout << hpa1 << endl;
	return 0;
}
```

**在32位下编译（因为32位指针和int都占4个字节）。**

``` bash
# 某次运行结果
4780872
10
4780872
10
0x48f348
0x48f338
```

这里我将no写了个默认值是10，通过打印f1、f2对象第一个位置的值，发现第一项是vptr，第二项才是no。而且Fruit和Apple类具有各自不同的vptr。（TDM-GCC以及LLVM(Clang)都是vptr在最前面）。

通过把所有成员设置为公有成员输出地址，或者调试的方法，可以测出在内存分布情况。测试代码如下：

``` cpp
#include <iostream>
using namespace std;

class Fruit {
public:
	int no;
	double weight;
	char key;
	void print() { }
	virtual void process() { }
};
 
class Apple: public Fruit {
public:
	int size;
	char type;
	void save() { }
	virtual void process() { }
};

int main(int argc, char const *argv[]) {
	cout << "sizeof(Fruit) = " << sizeof(Fruit) << endl;
	cout << "sizeof(Apple) = " << sizeof(Apple) << endl;
	Fruit fruit;
	Apple apple;
	printf("Fruit        = %x\n", &fruit);
	printf("Fruit.no     = %x\n", &fruit.no);
	printf("Fruit.weight = %x\n", &fruit.weight);
	printf("Fruit.key    = %x\n", &fruit.key);
	printf("Apple        = %x\n", &apple);
	printf("Apple.no     = %x\n", &apple.no);
	printf("Apple.weight = %x\n", &apple.weight);
	printf("Apple.key    = %x\n", &apple.key);
	printf("Apple.size   = %x\n", &apple.size);
	printf("Apple.type   = %x\n", &apple.type);
	return 0;
}
```

Windows 10 TDM-GCC 4.9.2 64-bit 某次运行结果：

``` bash
sizeof(Fruit) = 32
sizeof(Apple) = 40
Fruit        = 9ffe00
Fruit.no     = 9ffe08
Fruit.weight = 9ffe10
Fruit.key    = 9ffe18
Apple        = 9ffe20
Apple.no     = 9ffe28
Apple.weight = 9ffe30
Apple.key    = 9ffe38
Apple.size   = 9ffe3c
Apple.type   = 9ffe40
```

Mac OS X 10.11.3 LLVM version 7.0.2 (clang-700.1.81) 64-bit 某次运行结果

``` bash
sizeof(Fruit) = 32
sizeof(Apple) = 40
Fruit        = 50960c50
Fruit.no     = 50960c58
Fruit.weight = 50960c60
Fruit.key    = 50960c68
Apple        = 50960c28
Apple.no     = 50960c30
Apple.weight = 50960c38
Apple.key    = 50960c40
Apple.size   = 50960c44
Apple.type   = 50960c48
```

对应内存图：
![gcc64bit.png](\images\gcc64bit.png)

64位和32位的区别在于指针的大小，64位是8个字节，32位是4个字节。而当我使用不同的编译器时，32位呈现出的排布也有所不同。
Windows 10 TDM-GCC 4.9.2 32-bit 某次运行结果

``` bash
sizeof(Fruit) = 24
sizeof(Apple) = 32
Fruit        = 6dfe88
Fruit.no     = 6dfe8c
Fruit.weight = 6dfe90
Fruit.key    = 6dfe98
Apple        = 6dfe68
Apple.no     = 6dfe6c
Apple.weight = 6dfe70
Apple.key    = 6dfe78
Apple.size   = 6dfe7c
Apple.type   = 6dfe80
```

对应内存图
![gcc32bit.png](\images\gcc32bit.png)

Mac OS X 10.11.3 LLVM version 7.0.2 (clang-700.1.81) 32-bit 某次运行结果

``` bash
sizeof(Fruit) = 20
sizeof(Apple) = 28
Fruit        = bff79c30
Fruit.no     = bff79c34
Fruit.weight = bff79c38
Fruit.key    = bff79c40
Apple        = bff79c10
Apple.no     = bff79c14
Apple.weight = bff79c18
Apple.key    = bff79c20
Apple.size   = bff79c24
Apple.type   = bff79c28
```

对应内存图
![llvm32bit.png](\images\llvm32bit.png)
不管是stack还是heap建立的对象，以sizeof所求得的值都是相同的。

猜想：以结果来看，似乎GCC是以最宽的数据作为基始值倍数增加的，而LLVM在32位下是4，所以最后并没有将4字节补位，造成两个编译器的结果不同。
对于以下代码：

``` cpp
class A {
	char c1;
	double d1;
	char c2;
};

int main(int argc, char const *argv[]) {
	cout << "sizeof(A) = " << sizeof(A) << endl;
	return 0;
}
```

32位下GCC结果为24，LLVM结果为16。
