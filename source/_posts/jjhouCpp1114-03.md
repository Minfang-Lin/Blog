---
title: C++ 新标准 11/14 语言新特性 - 03
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

### lambdas表达式的完整形式

**\[...](...)mutable*<sub>opt</sub> throwSpec<sub>opt</sub>* -> *retType<sub>opt</sub>* {...}**

* []为lambda introducer，即lambda导入器（导入符号），说明这是一个lambda表达式。里面可以放lambda表达式需要取用（捕获）的外部变量。可以有值捕获和引用捕获两种方式。
* opt表示mutable、throwSpec、retType是可选的（可写可不写），但如果出现其中任一的话，必须写上前面的小括号指定参数。如果三个都没有，小括号可写可不写（视是否有参数而定）。
* retType用于描述返回值类型，由于lambda表达式写法特殊，由中括号开始，所以没有办法在前面指定返回类型。所以换作这种方式。
* 对于值捕获的变量，lambda不会改变其值，如果希望改变其值，需要加上mutable。

``` cpp
int id = 0;
auto f = [id]() mutable {
	std::cout << "id:" << id << std::endl;
	++id;
};

id = 42;
f();
f();
f();
std::cout << id << std::endl;
// 运行结果
// id:0
// id:1
// id:2
// 42
// 如果不加mutable，会报错Error: increment of read-only variabel 'id'
```

lambda表达式的type是一种匿名函数对象（仿函数），对于每个lambda表达式都有唯一的type与之对应。如果想定义这个type的一个对象，则需要用到模板（templates）或者auto关键词。如果确实需要知道具体是什么type，可以使用decltype()。

``` cpp
auto cmp = [](const Person& p1, const Person& p2) {
				return p1.lastname()<p2.lastname()||
						(p1.lastname()==p2.lastname()&&
			 			 p1.firstname()<p2.firstname());
			};

std::set<Person,decltype(cmp)>coll(cmp);
```

## Variadic Templates

### 例1：使用variadic templates处理不定类型不定数量的参数

从C++11开始，模板可以接受数量不定的参数，这就是所谓的variadic templates。例如：

``` cpp
// function 1
void printX() { }

// function 2
template <typename T, typename... Types>
void printX(const T& firstArg, const Types&... args) {
	cout << firstArg << endl;  // print first argument
	printX(args...);           // call printX() for remaining arguments
}

// function 3
template <typename... Types>
void printX(const Types&... args) { }
```

函数2比较特化，所以函数2和函数3可以共存，且函数3永远不会被调用。

### 例2：使用variadic templates重写printf()

``` cpp
// http://stackoverflow.com/questions/3634379/variadic-templates
void printf(const char* s) {
	while (*s) {
		if (*s=='%' && *(++s)!='%')
			throw std::runtime_error("invaild format string: missing arguments");
		std::cout << *s++;
	}
}

template<typename T, typename... Args>
void printf(const char* s, T value, Args... args) {
	while (*s) {
		if (*s=='%' && *(++s)!='%') {
			std::cout << value;
			printf(++s, args...); // call even when *s==0 to detect extra arguments
			return;
		}
		std::cout << *s++;
	}
}

int* pi = new int;
printf("%d %s %p %f\n", 15, "This is Ace.", pi, 3.14159); 
```

`char* s`接收到的是控制字符串（其实根本没用到它来控制输出的格式，全部都是通过cout处理的输出，但为了模拟C语言的printf，所以还是需要的），后面的参数被分解为一个（value）和一包（args）。

### 例3：设计max函数，用于检测一堆数据中最大的那个（一）

``` cpp
cout << max({57, 48, 60, 100, 20, 18}) << endl;
// 这里为了配合后面max函数接收的是initializer_list，所以需要给initializer_list的参数
```

如果参数types皆同，则无数动用variadic templates，只需要使用initializer_list<T>就可以了，以下是标准库中的做法：

``` cpp
// ...\4.9.2\include\c++\bits\predefined_oops.h
struct _Iter_less_iter
{
	template<typename _Iterator1, typename _Iterator2>
  	bool
  	operator()(_Iterator1 __it1, _Iterator2 __it2) const
  	{ return *__it1 < *__it2; }
};
  
inline _Iter_less_iter
__iter_less_iter()
{ return _Iter_less_iter(); }  
  
// ...\4.9.2\include\c++\bits\stl_algo.h	
template<typename _ForwardIterator, typename _Compare>
_ForwardIterator
__max_element(_ForwardIterator __first, _ForwardIterator __last,
	          _Compare __comp)
{
	if (__first == __last) return __first;
	_ForwardIterator __result = __first;
	while (++__first != __last)
		if (__comp(__result, __first))
  		__result = __first;
	return __result;
}

template<typename _ForwardIterator>
inline _ForwardIterator
max_element(_ForwardIterator __first, _ForwardIterator __last)
{
	return __max_element(__first, __last, 
					   __iter_less_iter());
}

template<typename _Tp>
inline _Tp
max(initializer_list<_Tp> __l)
{ return *max_element(__l.begin(), __l.end()); }	
```

### 例4：设计max函数，用于检测一堆数据中最大的那个（二）

``` cpp
cout << maximum(57, 48, 60, 100, 20, 18) << endl;
```

直接使用标准库里面的max：

``` cpp
// http://stackoverflow.com/questions/3634379/variadic-templates	
int maximum(int n) {
    return n;
}

template<typename... Args>
int maximum(int n, Args... args) {
    return std::max(n, maximum(args...));
}	
```

利用不断调用std::max()而完成最大值的获取。

同样地，如果参数type相同，则不需动用variadic templates。

### 例5：类模板，使用不同方式处理first和last元素

``` cpp
cout << make_tuple(7.5, string("hello"), bitset<16>(377), 42);
// 需要实现的打印效果 [7.5,hello,0000000101111001,42]
```

要进行这样的处理，必须知道处理的元素有几个，所以需要使用sizeof...()获取元素的个数。

``` cpp
// output operator for tuples
template <Typename... Args>
ostream& operator<<(ostream& os, const tuple<Args...>& t) {
	os << "[";
	PRINT_TUPLE<0, sizeof...(Args), Args...>::print(os, t);
	return os << "]";
}

// boost: util/printtuple.hpp
// helper: print element with index IDX of tuple with MAX elements
template<int IDX, int MAX, typename... Args>
struct PRINT_TUPLE {
	static void print(ostream& os, const tuple<Args...>& t) {
		os << get<IDX>(t) << (IDX+1==MAX ? "" : ",");
		PRINT_TUPLE<IDX+1, MAX, Args...>::print(os, t);
	}
};

// partial specialization to end the recursion
template<int MAX, typename... Args>
struct PRINT_TUPLE<MAX, MAX, Args...> {
	static void print (std::ostream& os, const tuple<Args...>& t) {
	
	}
};
```

### 例6：用于递归继承

递归调用处理的都是参数，所以使用函数模板；递归继承处理的都是类型，所以使用类模板。

``` cpp
template<typename... Values> class tuple;
template<> class tuple<> { };

// 将数据分为一个和一包，一个用于声明变量，一包再做成一个tuple，用于继承
template<typename Head, typename... Tail>
class tuple<Head, Tail...> 
	:private tuple<Tail...> // 使用私有继承，因为这样做只是为了满足递归继承，并不是is-a的关系
{
    typedef tuple<Tail...> inherited;
    
 protected:	
	Head m_head;

 public:
    tuple() { }
    tuple(Head v, Tail... vtail) 
      : m_head(v), inherited(vtail...) { }
         
    // typename Head::type head() { return m_head; }
    // [Error] no type named 'type' in 'class std::basic_string<char>'
    // 上面这样写会报错，因为像int、float并不知道Head::type是什么，所以可以用decltype
    // auto head()->decltype(m_head) { return m_head; }
    // 但其实可以直接用Head，因为返回类型就是Head
    Head head() { return m_head; } 
            
    inherited& tail() { return *this; }  // 强转后得到的就是tail的部分
    
};

tuple<int, float, string>t(41, 6.3, "nico");
```

对于t，继承关系如下：

```
        tuple<>
           ↑
     tuple<string>
  string m_head("nico");
           ↑
  tuple<float, string>
   float m_head(6.3);
           ↑ 
tuple<int, float, string>
     int m_head(41);      
```

### 例7：用于递归复合

``` cpp
template<typename... Values> class tup;
template<> class tup<> { };

template<typename Head, typename... Tail>
class tup<Head, Tail...> 
{
    typedef tup<Tail...> composited;
 protected:
	composited m_tail;	
	Head m_head;

 public:
    tup() { }
    tup(Head v, Tail... vtail) 
      : m_tail(vtail...), m_head(v) { }
         
    Head head() { return m_head; }  	          
    composited& tail() { return m_tail; } 
    // 这里需要用引用，不然修改值时因为改的是拷贝版本，原始版本不会被改变
	  
};
```

## C++ keywords

> http://en.cppreference.com/w/cpp/keyword

The end!