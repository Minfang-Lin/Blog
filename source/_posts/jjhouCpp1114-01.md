---
title: C++ 新标准 11/14 语言新特性 - 01
date: 2017-01-06 19:40
author: Minfang Lin
tags: [Cpp,11/14,侯捷]
---

## C++ Standard之演化

* C++ 98（1.0）
* C++ 03（TR1，technical report1）
* C++ 11（2.0）
* C++ 14

## Header files

C++ 2.0新特性包含语言和标准库两个层面，后者以header files的形式呈现。

* C++标准库的header files不带副档名（.h），例如`#include <vector>`
* 新式C header files不带副档名（.h），例如`#include <cstdio>`
* 旧式C header files（带有副档名.h）仍可使用，例如`#include <stdio.h>`

<!--more-->

下面是一些在C++ 2.0新增加的头文件

``` cpp
#include <type_traits>
#include <unordered_set>
#include <forward_list>
#include <array>
#include <tuple>
#include <regex>
#include <thread>
```

TR1版本中的很多特性是放在`namespace std:tr1`下的（如shared _ptr、regex），现在也被放到了`namespace std`里.

## 确认编译器是否支持C++11

编译器会定义一个`__cplusplus`的常量，对于C++11是这样定义的`#define __cplusplus 201103L`，对于C++98和C++03，则是`#define __cplusplus 199711L`

``` cpp
// VS2012
#include "stdafx.h"
#include <iostream>
using namespace std;

int main() {
	cout << __cplusplus << endl;
	return 0;
}

// Dev C++ 5+
#include <iostream>
using namespace std;

int main() {
	cout << __cplusplus << endl;
	return 0;
}
```

## C++11里一些重要的内容

* 语言层面
  * Variadic Templates
  * move Semantics
  * auto
  * Rabge-base for loop
  * Initializer list
  * Lambdas

* 标准库层面
  * type_traits
  * Unordered容器
  * forward_list
  * array
  * tuple
  * Con-currency
  * RegEx

## Variadic Templates

在C语言的printf函数里面就有用到过`...`，意为可以接受任意变化个数的参数。

``` c
int printf ( const char * format, ... );
```

{% blockquote Cplusplus.com http://www.cplusplus.com/reference/cstdio/printf/ printf %}
... (additional arguments)
Depending on the format string, the function may expect a sequence of additional arguments, each containing a value to be used to replace a format specifier in the format string (or a pointer to a storage location, for n).
There should be at least as many of these arguments as the number of values specified in the format specifiers. Additional arguments are ignored by the function.
{% endblockquote %}

在C++2.0里`...`也是用于表示一包（pack）东西。

* 用于template parameters，就是template parameters pack（模板参数包）
* 用于function parameter types，就是function parameter types pack（函数参数类型包）
* 用于function parameters，就是function parameters pack（函数参数包）

<i class="fa fa-exclamation-triangle" aria-hidden="true"></i> 需要注意3处`...`的位置（语法规定就那样写）。

``` cpp
// function 1
void printX()
{
}

// function 2
template <typename T, typename... Types>
void printX(const T& firstArg, const Types&... args)
{
    cout << firstArg << endl; 	// print first argument
    printX(args...); 			// call printX() for remaining arguments
}

printX(7.5, "hello", bitset<16>(377), 42);			
	//7.5 
	//hello 
	//0000000101111001 
	//42	
```

第二个函数里面的参数表示可以接受**一个参数**和**一包参数**，并且每个参数可以是不同数据类型的，可以用于做递归（将不定参数的数据一一分解）。

<i class="fa fa-exclamation-triangle" aria-hidden="true"></i> 第一个函数（不带参数的）是必须的，因为最后会分解为一个参数和一个数量为零的包，后者调用不带参数的函数。

如果想知道这包参数（args）一共有多少个参数，则可以使用`sizeof...(args)`来获取。

``` cpp
// function 3
template <typename...Types>
void printX (const Types&... args)
{ /*...*/ }
```

<i class="fa fa-question-circle" aria-hidden="true"></i> 问题：函数2和函数3可以共存吗？若可，谁比较泛化？谁比较特化？

函数2和函数3都是数量不定的模板（variadic template），不同的是函数2接受一个和一包参数，函数3接受一包参数。两者并不会引起ambiguous，经过测试会调用函数2。

## Space in Template Expressions

``` cpp
vector<list<int> > // ok in each C++ version
vector<list<int>>  // ok since C++11
```

## nullptr and std::nullptr_t

C++11使用nullptr代替NULL（或0）表示空指针。

``` cpp
void f(int);
void f(void*);
f(0);       // calls f(int)
f(NULL);    // calls f(int) if NULL is 0, ambiguous otherwise
f(nullptr); // calls f(void*)
```

对于重载的f函数，按照之前NULL就是0的话，f(NULL)就会存在二义性。

## Automatic Type Deduction with auto

以前auto用来描述本地变量，因为运行结束之后本地变量就会销毁，所以local变量也叫作auto变量。

C++11里面用auto来定义变量或类，编译器会自动判断数据类型。

``` cpp
auto i = 42;  // i has type int
double f();
auto d = f(); // d has type double
```

编译器在编译的过程中知道什么东西是什么类型，也因此可以做模板的实参推导，所以在使用标准库中的算法时不需要指明每个参数的类型。auto主要用于过长或者过于复杂的数据类型，比如：

``` cpp
vector<string> v;
// ...
auto pos = v:begin();       // pos has type vector<string>::iterator
auto l = [](int x)->bool {  // l has the type of a lambda 
	// ...                  // taking on int and returning a bool
} 
```

## Uniform Initialization

在C++11之前，程序员（特别是新手），很容易对初始化对象或变量时怎么写产生疑惑。初始化可能发生在小括号，大括号或者赋值运算符上。因此C++11引入**一致性初始化**（Uniform Initialization）的概念，任何初始化都可以使用共通的语法，即大括号。例如：

``` cpp
int values[] {1, 2, 3};
vector<int> v {2, 3, 5, 7, 11, 13, 17};
vector<string> cities {
	"Berlin", "New York", "London", "Braunschweig", "Cairo", "Cologne"
};
complex<double> c {4.0, 3.0} // equivalent to c(4.0,3.0)
```

所有初始化都可以通过大括号完成，是如何实现的呢？

实际上，编译器看到`{t1,t2... tn}`便做出一个`initializer_list<T>`，它关联到一个`array<T,n>`。调用函数（例如ctor）时，该array内的元素可被编译器逐一传给函数。
但若调用的函数参数是一个`initializer_list<T>`，则不会分解为数个类型为T的参数，而是直接将做出的`initializer_list<T>`传入。

以上面的代码为例，`{"Berlin", "New York", "London", "Braunschweig", "Cairo", "Cologne"}`形成一个`initializer_list<string>`，背后有一个`array<string,6>`。调用`vector<string>`的ctors时编译器找到一个`vector<string>`接受一个`initializer_list<string>`，所有容器皆有如此的ctor。而{3.0,4.0}也形成一个`initializer_list<double>`，背后有一个`array<double,2>`。调用`complex<double>`的ctor时该array内的2个元素被分解传给ctor。`complex<double>`并无任何ctor接受`initializer_list<T>`。

## Initializer Lists

Initializer Lists也跟Variadic Templates一样可以用于不定个数的元素。

当使用大括号时，若未定义具体的值，则会初始化为0值，若数据类型是指针，则初始化为nullptr。

``` cpp
int i;      // i has undefined value
int j{};    // j is initialized by 0
int* p;     // p has undefined value
int* q{};   // q is initialized by nullptr
```

若声明的数据类型和大括号内的数据类型不匹配时，会自动转换类型，但缩窄范围（如把double数据赋值给int）的转换是不允许的。

``` cpp
int x1(5.3);     // ok, but x1 becomes 5
int x2 = 5.3     // ok, but x2 becomes 5
int x3{5.3};     // error, narrowing
int x4 = {5.3}   // error, narrowing
char c1{7};      // ok, even though 7 is an int, this is not narrowing
char c2{9999};   // error, narrowing (if 9999 doesn't fit into a char)
std::vector<int> v1 {1, 2, 4, 5}      // ok
std::vector<int> v2 {1, 2.3, 4, 5.6}  // error, narrowing
```

Initializer Lists的背后通过`initializer_list<>`实现。

``` cpp
void print (std::initializer_list<int> vals) {
	for (auto p = vals.begin(); p != vals.end(); ++p) {
		std::cout << *p << std::endl;
	}
}
print({2,3,5,7,11,13,17});  // pass a list of values to print()
```

传给`initializer_list`者，一定必须也是个`initializer_list`(or `{...}`形式)。

当同时有特定个数的参数和initializer list作为参数时，会优先选用后者，测试代码如下：

``` cpp
class P
{
public:
	// ctor version 1
	P(int a, int b) {
		cout << "P(int, int), a=" << a << ", b=" << b << endl;
	}

	// ctor version 2
	P(initializer_list<int> initlist) {
		cout << "P(initializer_list<int>), values= ";
		for (auto i : initlist)
		cout << i << ' ';
		cout << endl;
	}
};

P p(77,5);          // P(int, int), a=77, b=5
P q {77,5};         // P(initializer_list<int>), values= 77 5
P r {77,5,42};      // P(initializer_list<int>), values= 77 5 42
P s = { 77, 5 };    // P(initializer_list<int>), values= 77 5
```

<i class="fa fa-exclamation-triangle" aria-hidden="true"></i> `P s={77,5};`调用的是构造函数，而不是赋值。

如果没有initializer list的版本（版本2），q和s将会调用版本1的构造函数，而r会报错。