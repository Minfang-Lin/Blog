---
title: Geekband C++ STL 第三周作业
date: 2016-04-21 20:15
tags: Cpp
---

## 题目说明：

假设有个计算类Calculator，它要处理int, long, float, double等数值类型。用模板实现`GetLimit()`方法，获得每种类型的数值的上限LIMIT，比如int的上限是100，long的上限是1000，float的上限是999.99，double的上限是888.8888888等等。


<!--more-->

## 解答

其实我不是特别理解这道题目的意思，大概分析了一下可能是这样的，首先有一个叫做`Calculator`的类，顾名思义它是用来做计算用的（这题没有要求我们实现计算的功能），`Calculator`类里面有一个成员函数叫做`GetLimit`，返回的是对应数值类型的值。由于`LIMIT`的数据类型以及值都是需要根据实际数据类型确定的，所以`LIMIT`不是`Calculator`的一个成员常量。为了处理不同的数值类型，我另外写了一个`CalculatorTraits`的类，里面定义了不同数值类型对应的返回值及`LIMIT`的值。代码如下：
``` cpp
#include <iostream>
using namespace std;
 
template <typename T>
struct CalculatorTraits { };
 
template<>
struct CalculatorTraits<int> {
    typedef int ReturnType;
    static constexpr ReturnType LIMIT = 100;
};
 
template<>
struct CalculatorTraits<long> {
    typedef long ReturnType;
    static constexpr ReturnType LIMIT = 1000;
};
 
template<>
struct CalculatorTraits<float> {
    typedef float ReturnType;
    static constexpr ReturnType LIMIT = 999.99;
};
 
template<>
struct CalculatorTraits<double> {
    typedef double ReturnType;
    static constexpr ReturnType LIMIT = 888.888888;
};
 
template <typename T>
struct Calculator {
    typename CalculatorTraits<T>::ReturnType GetLimit() {
        return CalculatorTraits<T>::LIMIT;
    }   
};
 
int main() {
    Calculator<int> c1;
    Calculator<long> c2;
    Calculator<float> c3;
    Calculator<double> c4;
    cout << c1.GetLimit() << endl;
    cout << c2.GetLimit() << endl;
    cout << c3.GetLimit() << endl;
    cout << c4.GetLimit() << endl;
    return 0;
}
```
`CalculatorTraits`里面的`LIMIT`如果不写`static`是局部变量，不能被外部调用。静态成员在类内初始化时必须加上`const`（只适用于`int`类型，包括`enum`），`double`类型在使用TDM-GCC 4.9.2时会报错，可以通过把`const`改成`constexpr`解决。或者在类外写，如以`int`为例：
``` cpp
template<>
struct CalculatorTraits<int> {
    typedef int ReturnType;
    static ReturnType LIMIT;
};
CalculatorTraits<int>::ReturnType CalculatorTraits<int>::LIMIT = 100;
```
这样也是可以的。而且一般都是在类外做初始化。

在群里谈到了`limits`，可以使用`numeric_limits<int>::max()`返回int类型的最大值，对于之前的程序，只需要加上头文件`#include <limits>`，然后改成`static constexpr ReturnType LIMIT = numeric_limits<int>::max();`之类的就可以了。


在讲解的时候，文杰老师提了出题的用意，有两点：第一是希望学员会使用`traits`表达类型信息，因为C++没有C#/Java中类似类型的判断，比如C#里有`if (type is SomeType) { … }`，但C++没有。这个时候考虑用`traits`表达；第二，希望学员注意类似`typename CalculatorTraits<T>::ReturnType`的返回类型，不要觉得奇怪，是模板中常用的。

以下是GeekBandSuyan20160307的实现方法：
``` cpp
#include <iostream>
using namespace std;

struct Calculator {
    template<typename T> T GetLimit() const { return T(65); }
};

template<> int Calculator::GetLimit() const { return int(100); }
template<> long Calculator::GetLimit() const { return long(1000); }
template<> float Calculator::GetLimit() const { return float(999.99); }
template<> double Calculator::GetLimit() const { return double(888.8888888); }

int main() {
    Calculator ca;
    cout << "Limit of int : " << ca.GetLimit<int>() << endl;
    cout << "Limit of long : " << ca.GetLimit<long>() << endl;
    cout << "Limit of float : " << ca.GetLimit<float>() << endl;
    cout << "Limit of double : " << ca.GetLimit<double>() << endl;
    cout << "Limit of char : " << ca.GetLimit<char>() << endl;
    return 0;
}
```