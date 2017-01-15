---
title: C++ 新标准 11/14 - 03
date: 2017-01-15 20:00
author: Minfang Lin
tags: [Cpp,11/14,侯捷]
---

## decltype

使用decltype可以让编译器找出一个表达式结果的类型，这个很像是对于**typeof**的需求（GUN C当中有typeof，但是不是C++标准里的东西）。

``` cpp
map<string,float> coll;
decltype(coll)::value_type elem;

// 在C++11之前，无法通过对象取得type，而是必须确切知道它的类型，所以只能写成这样
map<string,float>::value_type elem;
```

{% blockquote Nicolai M.Josuttis,the C++ Standard Library 2/e %}
By using the new decltype keyword, you can <u>let the compiler find out the type of an expression.</u> This is the realization of often requested **typeof** feature.
One application of decltype is ①<u>to declare return types</u>. Another is to use it ②<u>in metaprogramming</u> or to ③<u>pass the type of a lambda</u>.
{% endblockquote %}

### ① decltype, used to declare return types

标准库里面的add()函数，加的两样东西都是一样的，比如int+int，但下面这种情况就不一样了。

``` cpp
template<typename T1, typename T2>
decltype(x+y) add(T1 x, T2 y); // 由于x和y未知，所以编译通不过
```

x和y可以是两个不同的类型，比如青菜加萝卜。能不能相加在于程序员对T1和T2的设计。在C++11之前因为不知道T1和T2是什么，所以不知道怎么设置返回类型，有了decltype就可以了（当然这只是让编译可以通过，如果x和y真的不能相加，调用它的代码编译时还是会报错）。而上面的代码，由于编译器是顺序编译的，看到x和y的时候不知道它们是什么，所以编译还是通不过。需要用下面的写法：

``` cpp
template<typename T1, typename T2>
auto add(T1 x, T2 y)->decltype(x+y);
```

### ② decltype, used in metaprogramming

``` cpp
template<typename T>
void test(T obj) {
	// 当我们手上有type，可取其inner typedef，没问题
	map<string,float>::value_type elem1;
	
	// 面对obj取其class type的inner typedef，因为如今我们有了工具decltype
	map<string,float> coll;
	decltype(coll)::value_type elem2;
	
	// 有了decltype也可以这样写
	typedef typename decltype(obj)::iterator iType;
	// typedef typename T::iterator iType;
	decltype(obj) anotherOby(obj);
}
```

如果使用者使用的时候，指定进入的是复数`test(complex<int>())`，由于复数没有迭代器，上面的代码还是会报错。所以，模板只是半成品，还需要看调用它的代码。

### ③ decltype, used to pass the type of a lambda

``` cpp
auto cmp = [](const Person& p1, const Person& p2) {
				return p1.lastname()<p2.lastname() ||
						(p1.lastname()==p2.lastname() &&
						p1.firstname()<p2.firstname());
			}; 

std::set<Person,delctype(cmp)> coll<cmp>;
```

面对lambda，我们手上往往只有object，没有type，要获得其type就得借助于decltype。

## lambdas

C++11引入了lambdas，它允许程序员定义出inline function，可以作为参数或者本地变量使用。lambdas改变了程序员对C++标准库的使用方式。我们在使用C++标准库的时候，往往会写一些小东西来声明我们的意图，比如sort时怎么比大小就可以由程序员来定义，通过函数对象（仿函数）来实现，也可以用过lambda来写。

``` cpp
[] {
	std::cout << "hello lambda" << std::endl;
}
```

这看起来是一个函数，其实是一个对象，所以不会有输出。可以写成下面这样：

``` cpp
[] {
	std::cout << "hello lambda" << std::endl;
} ();  // prints "hello lambda" 
```

这里的小括号不是产生临时对象的意思，而是当做函数执行的意思。当然上面的代码并没有什么意义，要输出直接cout就可以了，所以一般的用法是下面这样的：

``` cpp
auto l = [] {
	std::cout << "hello lambda" << std::endl;
};

l(); // prints "hello lambda" 
```

