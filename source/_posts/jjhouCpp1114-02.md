---
title: C++ 新标准 11/14 - 02
date: 2017-01-11 20:00
author: Minfang Lin
tags: [Cpp,11/14,侯捷]
---

## explicit关键字

C++2.0之前explicit用于一个实参的构造函数（none explicit one argument constructor)。多余一个参数的不会发生隐式转换。

``` cpp
struct Complex1 
{
    int real, imag;
		
    Complex1(int re, int im=0) : real(re), imag(im) {  }

    Complex1 operator+(const Complex1& x) 
    { return Complex1((real + x.real), (imag + x.imag)); }     	
};	

Complex1 c1(12,5);
Complex1 c2 = c1 + 5;
```

这里的im有默认值，所以只有一个需要的实参，还是属于none explicit one argument constructor。

编译器会去找是否可以将5转换为复数（5+0i），所以利用构造函数将5变为复数。

<!-- more -->

``` cpp
struct Complex2 
{
	int real, imag;
	
	explicit Complex2(int re, int im=0) : real(re), imag(im) {  }

	Complex2 operator+(const Complex2& x) 
	{ return Complex2((real + x.real), (imag + x.imag)); }     	
};	

Complex2 c1(12,5);
Complex2 c2 = c1 + 5;  // error
```

这里的explicit的意思是当**明确**调用构造函数的时候才会调用它，故5不会调用构造函数转换成复数。

在C++2.0之后，explicit用于有一个以上实参的构造函数，禁止隐式转换。

``` cpp
class P
{
public:
	P(int a, int b) {
		cout << "P(int a, int b) \n"; 
	}
	  
	P(initializer_list<int>) { 
		cout << "P(initializer_list<int>) \n"; 
	}
		
	explicit P(int a, int b, int c) {
	   	cout << "explicit P(int a, int b, int c) \n"; 
	}
};

void fp(const P&) {  };
	
// explicit ctor with multi-arguments
P p1 (77, 5);           // P(int a, int b)
P p2 {77, 5};           // P(initializer_list<int>)
P p3 {77, 5, 42};       // P(initializer_list<int>)
P p4 = {77, 5};         // P(initializer_list<int>)
P p5 = {77, 5, 42};     // [Error] converting to 'P' from initializer list would use explicit constructor 'P::P(int, int, int)'
P p6 (77,5,42);         // explicit P(int a, int b, int c)
	
fp( {47,11} );          // P(initializer_list<int>)
fp( {47,11,3} );        // [Error] converting to 'const P' from initializer list would use explicit constructor 'P::P(int, int, int)'
fp( P{47,11} );	        // P(initializer_list<int>)
fp( P{47,11,3} );       // P(initializer_list<int>)
	
P p11 {77, 5, 42, 500};	     // P(initializer_list<int>)
P p12 = {77, 5, 42, 500};    // P(initializer_list<int>)
P p13 {10};                  // P(initializer_list<int>)
```

## range-based for statement

``` cpp
for ( decl : coll ) {
	statement
}

// 相当于（并非源代码说是这样做的）
for (auto _pos=coll.begin(), _end=coll.end(); _pos!=end; ++_pos) {
	decl = *_pos;
	statement
}

// 或者相当于，其中begin()和end()为全局函数，可以接受容器
for (auto _pos=begin(coll), _end=end(coll); _pos!=end; ++_pos) {
	decl = *_pos;
	statement
}
```

编译器在操作的时候，就是将coll容器中的数据一个个取出来，放到左边的decl（一个声明的变量）中，每拿出一个元素，执行statement的操作。

``` cpp
for (int i : {2, 3, 5, 7, 9, 13, 17, 19} ) {
	cout << i << endl;
}
```

``` cpp
vector<double> vec;

for (auto elem : vec) {
	cout << elem << endl;
}

for (auto& elem : vec) {
	elem *= 3;
}
```

如果要改变元素，需要声明为引用，不然只是拷贝而已。关联式容器（如set、map）都不允许直接使用迭代器修改元素的值。

``` cpp
template <typename T>
void printElements (const T& coll) {
	for (const auto& elem : coll) {
		cout << elem << endl;
	}
}

// 相当于
{
	for ( auto _pos=coll.begin(); _pos!=coll.end(); ++_pos) {
		const auto& elem = *_pos;
		cout << elem << endl;
	}
}
```

当容器里的数据类型和decl的数据类型不匹配的时候，编译器会尝试去做类型转换，但如果像下面这样不允许转换，则会出现编译错误。

``` cpp
class C {
public:
	explicit C (const string& s); // explicit(!) type conversion from strings
};

vector<string> vs;
for (const C& elem : vs) { // ERROR: no conversion from string to C
	cout << elem << endl;
}
```

## =default, =delete

如果自行定义了一个ctor，则编译器不会再给一个default ctor。如果强制加上=default，就可以重新获得并使用default ctor。

``` cpp
class Zoo {
public:
	Zoo(int i1, int i2): d1(i1), d2(i2) { }
	Zoo(const Zoo&)=delete;              // 拷贝构造
	Zoo(Zoo&&)=default;                  // Move Constructor，C++11新增的右值引用
	Zoo& operator=(const Zoo&)=default;  // 拷贝赋值 
	Zoo& operator=(const Zoo&&)=delete;  // Move assignment，搬移赋值
	virtual ~Zoo() { }
private:
	int d1, d2;
};
```

default ctor不需要任何参数，且是一个空函数。看起来什么也没做，但比如是一个继承自父类的子类，子类的ctor必须调用父类的ctor，这时候默认的ctor就有了价值，虽然看起来是空的，但编译器在别的地方有一些隐藏的动作，调用父类的ctor。

构造函数、拷贝构造函数、拷贝赋值函数、析构函数，编译器都会生成默认的版本（假如程序员没有自行定义的话）。且这些函数都是public、inline的。

<i class="fa fa-question-circle" aria-hidden="true"></i> 这些函数做了些什么呢？

default ctor和dtor主要给编译器一个地方用来放置隐藏在幕后的code，像是唤起base classed以及non-static members的ctors和dtors。编译器产出的dtor是non-virtual的，除非这个class的base class本身宣告有virtual dtor。至于copy ctor和copy assignment operator，编译器合成版只是单纯将source object的每一个non-static data members拷贝到destination object。

``` cpp
class Foo {
public:
	Foo(int i): _i(i) { }
	Foo()=default;
	
	Foo(const Foo& x):_i(x._i) { }
	//! Foo(const Foo&)=default; // Error: 'Foo::Foo(const Foo&)'cannot be overloaded
	//! Foo(const Foo&)=delete;  // Error: 'Foo::Foo(const Foo&)'cannot be overloaded
};

	Foo& operator=(const Foo& x) { _i=x._i; return *this;}
	//! Foo& opreator=(const Foo&)=default; // Error: 'Foo& opreator=(const Foo&)' cannot be overloaded
	//! Foo& opreator=(const Foo&)=delete;  // Error: 'Foo& opreator=(const Foo&)' cannot be overloaded
	
	//! void func1()=default; // Error: 'void Foo::func1()' cannot be defaulted
	void func2()=delete;      // ok
	
	//! ~Foo()=delete; // 这会造成使用Foo obj时出错->Error：use of deleted function 'Foo::~Foo()'
	~Foo()=default;
private:
	int _i;
};

Foo f1(5);
Foo f2;     // 如果没有写出=default版本->Error: no matching function for call to 'Foo::Foo()'
Foo f3(f1); // 如果没有copy ctor=delete;->Error: use of deleted function 'Foo::Foo(const Foo&)'
f3 = f2;    // 如果没有copy assign=delete;->Error: use of deleted function 'Foo& Foo::operator=(const Foo&)' 
```

`=default`用于Big-Five之外无意义，编译报错。
`=delete`可用于任何函数身上（=0只能用于virtual函数）。

## Alias Template（template typedef）

``` cpp
template <typename T>
using Vec = std::vector<T, MyAlloc<T>>;  // standard vector using own allocator

// the term
Vec<int> coll;
// is equivalent to
std::vector<int, MyAlloc<int>> coll;
```

这是不是跟typedef很像呢？但是如果用typedef，是无法达到相同效果的。

``` cpp
#define Vec<T> template<typename T> std::vector<T, MyAlloc<T>>;

Vec<int> coll;
// 将变成下面这样
template<typename int> std::vector<int, MyAlloc<int>>; // 这不是我们想要的

// 如果写成下面这样，只能固定接受int类型的参数，也不是我们想要的
typedef std::vector<int, MyAlloc<int>> Vec;
```

不能对Alias Template进行偏特化或特化，只能对于原本的类进行。化名并非本尊！

## Type Alias（similar to typedef）

``` cpp
// type alias, identical to
// typedef void (*func)(int, int);
using func = void(*)(int, int);

// the name 'func' now denotes a pointer to function:
void example(int, int) { }
func fn = example;
```

``` cpp
// type alias can introduce a member typedef name
template<typename T>
struct Container {
	using value_type = T; // 等价于typedef T value_type
};

// which can be used in generic programming
template<typename Cntr>
void fn2(const Cntr& c)
{
	typename Cntr::value_type n;
}
```

``` cpp
// type alias used to hide a template parameter
template <class CharT>
using mystring = std::basic_string<CharT, std::char_traits<CharT>>;

mystring<char> str;

// 标准库的<string>和<string_fwd.h>都有以下typedef，作用和上面的写法是一样的
typedef basic_string<char> string;
```

typedef和type alias这两种声明方式并没有不同。这种声明可以写在块作用域（block scope）、类作用域（class scope）或是命名空间作用域（namespace scope)。

### using的用法总结

* using-directives(`using namespace std`) for namespaces and using-declarations(`using std::cout`) for namespace members.
* using-declarations for class members.
* type alias and alias template declaration. (since C++11)

## noexcept

在函数名后面跟上noexcept表示程序员肯定这个函数不会抛出异常。如果一个函数有异常没有被处理的话，就会向调用它的函数抛异常，如果都没有处理异常的话，程序就会终止（通过调用std::terminate()，这个函数默认调用std::abort()把程序结束掉）。

``` cpp
void foo() noexcept;
```

noexcept后面还可以跟上一个()，里面写上条件（结果是bool的表达式），意思为当满足这个条件时，不会抛出异常。如果不写的话，相当于(true)，所以上面的代码相当于下面这样写：

``` cpp
void foo() noexcept(true);
```

()内也可以下面这样的表达式：

``` cpp
void swap(Type& x, Type& y) noexcept(noexcept(x.swap(y))) {
	x.swap(y);
}
```

意思是当x.swap(y)不抛出异常的时候swap函数也不会抛出异常。

### <i class="fa fa-exclamation-triangle" aria-hidden="true"></i> 必须使用noexcept的情况

当类同时有拷贝构造和搬移构造的时候，会优先使用搬移构造（move constructor）。如果是一些特殊的类，比如vector，必须在搬移构造和搬移析构上加上noexcept，因为在vector二倍增长的时候，会调用搬移构造函数，如果不加noexcept，则vector无法使用它。

{% blockquote StackOverflow http://stackoverflow.com/questions/8001823/how-to-enforce-move-semantics-when-a-vector-grows how to enforce move semantics when a vector grows %}
You need to inform C++ (specifically std::vector) that your move constructor and destructor does not throw, using noexcept. Then the move constructor will be called when the vector **grows**.
If the constructor is not noexcept, std::vector can't use it, since then it can't ensure the exception guarantees demanded by the standard.
{% endblockquote %}

growable conatiners（会发生memory reallocation）的只有两种：vector和deque。

## override

C++11新增一个关键字override，所谓override，函数签名必须完全相同。若本意是要override，但写错了，导致违背原本意图，比如下面这样：

``` cpp
struct Base {
	virtual void vfunc(float) { }
};

struct Derived1 : Base {
	virtual void vfunc(int) { }
	// accidentally create a new virtual function, when one intended to override a base class function
	// This is a common problem, particularly when a user goes to modify the base class.
};
```

``` cpp
struct Derived2 : Base {
	virtual void vfunc(int) override { }
	// Error 'virtual void Derived2::vfunc(int)' marked override, but does not override
	
	virtual void vfunc(float) override { }
};
```

override的意思是告诉编译器去检查这个类的父类（或父类的父类等）是否有一个具有相同函数签名的virtual函数，如果没有，则编译器会报错。

## final

用于修饰类或者虚函数。

``` cpp
struct Base1 final { };

struct Derived1 : Base { };
// Error: cannot derive from 'final' base 'Base1' in derived type 'Derived1'
```

``` cpp
struct Base2 {
	virtual void f() final;
};

struct Derived2 : Base2 {
	void f();
	// Error: overriding final function 'virtual void Base2::f()'
};
```
