---
title: C++ 新标准 11/14 标准库新部件 - 01
date: 2017-01-29 21:00
author: Minfang Lin
tags: [Cpp,11/14,侯捷]
---

## Rvalue references

右值引用是C++03出现的一种新的引用类型，用于解决不必要的copy。当赋值的右边（拷贝来源端）是一个右值（rvalue），那么左边（拷贝接受端）的对象可以不重新分配内存，而直接“偷”右边对象的内容（这种情形只有在指针的情况下才会发生）。

* Lvalue：可以出现于operator=左侧者，如变量
* Rvalue：只能出现于operator=右侧者，如临时对象

``` cpp
// 以int试验
int a = 9;
int b = 4;

a = b;   // ok
b = a;   // ok
a = a+b; // ok

a + b = 42; // Error: lvalue required as left operand of assignment
```

``` cpp
// 以string试验
string s1("Hello");
string s2("World");
s1 + s2 = s2;                  // ok, s1+s2可以当做lvalue
cout << "s1: " << s1 << endl;  // s1: Hello
cout << "s2: " << s2 << endl;  // s2: World
string() = "World";            // ok, 居然可以对临时对象赋值
```

``` cpp
// 以complex试验
complex<int> c1(3,8), c2(1,0);
c1 + c2 = complex<int>(4,9);         // ok, c1+c2可以当做lvalue
cout << "c1: " << c1 << endl;        // c1:(3,8)
cout << "c2: " << c2 << endl;        // c2:(1,0)
complex<int>() = complex<int>(4,9);  // ok, 居然可以对临时对象赋值
```

C++本身的一些class/type会引来一些对于赋值动作不可预见的情况，如前面的string和complex，而造成定义的不正确（临时对象是rvalue）。

``` cpp
int foo() { return 5; }
int x = foo();    // ok
int* p = &foo();  // Error: 不能对rvalue取其地址
foo() = 7;        // Error
```

函数的返回值是rvalue，对rvalue取地址是不允许的。

对于临时对象（rvalue），可以通过右值引用（&&）避免重新分配内存创建对象；而对于lvalue，可以通过调用move()函数实现，但必须确保这个对象之后不会再被使用。

## Perfect Forwarding

所谓完美的传递，指的是在多级函数调用的过程中，参数的属性（可变的还是const的，lvalue还是rvalue）也能够被传递。

``` cpp
void process(int& i) {
	cout << "process(int&):" << i << endl;
}

void process(int&& i) {
	cout << "process(int&&):" << i << endl;
}

void forward(int&& i) {
	cout << "forward(int&&):" << i << ", ";
	process(i);
}

     int a = 0;
     process(a);        // process(int&):0
                        // 视为lvalue处理
                        
     process(1);        // process（int&&）：1 
                        // 临时对象视为rvalue处理
                        
     process(move(a));  // process(int&&):0 
                        // 强制将lvalue改为rvalue
                        
     forward(2);        // forward(int&&):2, process(int&):2
                        // rvalue经由forward()传给另一个函数后却变为lvalue
                        // （原因是在传递过程中把它变为了非临时对象）
                        
     forward(move(a));  // forward(int&&):0, process(int&):0
     
//!  forward(a);        // Error: cannot bind 'int' lvalue to 'int&&'

     const int &b = 1;

//!  process(b);        // Error: no matching function for call 'process(const int&)

//!  process(move(b));  // Error: no matching function for call to
                        // 'process(std::remove_rederence<const int&>::type)'

//!  int& x(5);         // Error: invaild initialization of non-const reference of
                        // tyoe 'int&' from an rvalue of type 'int'
```

显然目前的写法并不能实现属性传递。所以标准库里面做move的操作之前会调用forward()，借用下面这段代码的设计完成完美的传递。

``` cpp
// ...\4.9.2\include\c++\bits\move.h

//  @brief  Forward an lvalue.
//  @return The parameter cast to the specified type.
//  This function is used to implement "perfect forwarding".
template<typename _Tp>
	constexpr _Tp&&
	forward(typename std::remove_reference<_Tp>::type& __t) noexcept
	{ return static_cast<_Tp&&>(__t); }

//  @brief  Forward an rvalue.
//  @return The parameter cast to the specified type.
//  This function is used to implement "perfect forwarding".
template<typename _Tp>
	constexpr _Tp&&
	forward(typename std::remove_reference<_Tp>::type&& __t) noexcept
	{
		static_assert(!std::is_lvalue_reference<_Tp>::value, "template argument"
		" substituting _Tp is an lvalue reference type");
		return static_cast<_Tp&&>(__t);
	}

//  @brief  Convert a value to an rvalue.
//  @param  __t  A thing of arbitrary type.
//  @return The parameter cast to an rvalue-reference to allow moving it.
template<typename _Tp>
	constexpr typename std::remove_reference<_Tp>::type&&
	move(_Tp&& __t) noexcept
	{ return static_cast<typename std::remove_reference<_Tp>::type&&>(__t); }
```

## 设计一个带有move的class

``` cpp
class MyString { 
private: 
	char* _data; 
	size_t _len; 
	void _init_data(const char *s) { 
			_data = new char[_len+1]; 
			memcpy(_data, s, _len); 
			_data[_len] = '\0'; 
	} 
public: 
	// default ctor
	MyString() : _data(NULL), _len(0) {  }

	// ctor
	MyString(const char* p) : _len(strlen(p)) { 
		_init_data(p); 
	} 

	// copy ctor
	MyString(const MyString& str) : _len(str._len) { 
		cout << "Copy Constructor is called! source: " << str._data 
		     << " [" << (void*)(str._data) << ']' << endl;   	
		_init_data(str._data); 	// COPY
	} 

	// move ctor, with "noexcept"
	MyString(MyString&& str) noexcept : _data(str._data), _len(str._len) {  
		cout << "Move Constructor is called! source: " << str._data 
		     << " [" << (void*)(str._data) << ']' << endl; 
		str._len = 0; 		
		str._data = NULL;   // 这句很重要!!!
		                    // 另外需要避免 delete (in dtor)，不然会把临时对象杀掉  
	}
 
	// copy assignment
	MyString& operator=(const MyString& str) { 
		cout << "Copy Assignment is called! source: " << str._data 
		     << " [" << (void*)(str._data) << ']' << endl; 
		if (this != &str) { 
			if (_data) delete _data;  
			_len = str._len; 
			_init_data(str._data); 	// COPY! 
		} 
		else {
			cout << "Self Assignment, Nothing to do." << endl;   
		}
		return *this; 
	} 

	// move assignment
	MyString& operator=(MyString&& str) noexcept { 
		// 注意 noexcept 
		cout << "Move Assignment is called! source: " << str._data 
		     << " [" << (void*)(str._data) << ']' << endl; 
		if (this != &str) { 
			if (_data) delete _data; 
			_len = str._len; 
			_data = str._data;	// MOVE!
			str._len = 0; 
			str._data = NULL; 	// 这句很重要!!!
			                    // 另外需要避免 delete (in dtor)，不然会把临时对象杀掉 
		} 
		return *this; 
	}
 
	// dtor
	virtual ~MyString() { 	
	// 文档说需 noexcept 但本处无. destructor is noexcept by default.
	// Johan Lundberg Mar 18 '13 at 12:12    
		cout << "Destructor is called! " << "source: "; 
		if (_data)
			cout << _data; 
		cout << " [" << (void*)(_data) << ']' << endl;
		
		if (_data) {
			delete _data; 	
		}
	}   	
}; 
```