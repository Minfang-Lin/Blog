---
title: C++ 新标准 11/14 - 01
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

> 摘自Cplusplus.com [[链接]](http://www.cplusplus.com/reference/cstdio/printf/)
 ... (additional arguments)
Depending on the format string, the function may expect a sequence of additional arguments, each containing a value to be used to replace a format specifier in the format string (or a pointer to a storage location, for n).
There should be at least as many of these arguments as the number of values specified in the format specifiers. Additional arguments are ignored by the function.

在C++2.0里`...`也是用于表示一包（pack）东西。

* 用于template parameters，就是template parameters pack（模板参数包）
* 用于function parameter types，就是function parameter types pack（函数参数类型包）
* 用于function parameters，就是function parameters pack（函数参数包）

<div class="tip">需要注意3处`...`的位置（语法规定就那样写）。</div>

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

<div class="tip">第一个函数（不带参数的）是必须的，因为最后会分解为一个参数和一个数量为零的包，后者调用不带参数的函数。</div>

如果想知道这包参数（args）一共有多少个参数，则可以使用`sizeof...(args)`来获取。

``` cpp
// function 3
template <typename...Types>
void printX (const Types&... args)
{ /*...*/ }
```

> 问题：函数2和函数3可以共存吗？若可，谁比较泛化？谁比较特化？

函数2和函数3都是数量不定的模板（variadic template），不同的是函数2接受一个和一包参数，函数3接受一包参数。两者并不会引起ambiguous，经过测试会调用函数2。