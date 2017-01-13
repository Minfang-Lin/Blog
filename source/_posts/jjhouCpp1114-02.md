---
title: C++ 新标准 11/14 - 02
date: 2017-01-11 20:00
author: Minfang Lin
tags: [Cpp,11/14,侯捷]
---

## explicit关键字

C++2.0之前explicit用于一个实参的构造函数（none explicit one argument constructor)。多余一个参数的不会发生隐式转换。

``` cpp
struct Complex {
	int real, imag;
	Complex(int re, int im=0): real(re), imag(im) { }
	Complex operator+(const Complex& x) {
		return Complex((real+x.real), (imag+x.imag));
	}
}

Complex c1(12,5);
Complex c2 = c1 + 5;
```

这里的im有默认值，所以只有一个需要的实参，还是属于none explicit one argument constructor。

编译器会去找是否可以将5转换为复数（5+0i），所以利用构造函数将5变为复数。

``` cpp
struct Complex {
	int real, imag;
	explicit Complex(int re, int im=0): real(re), imag(im) { }
	Complex operator+(const Complex& x) {
		return Complex((real+x.real), (imag+x.imag));
	}
}

Complex c1(12,5);
Complex c2 = c1 + 5;  // error
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
	
//explicit ctor with multi-arguments
P p1 (77, 5);	        //P(int a, int b)
P p2 {77, 5};           //P(initializer_list<int>)
P p3 {77, 5, 42};       //P(initializer_list<int>)
P p4 = {77, 5};	 		//P(initializer_list<int>)
P p5 = {77, 5, 42};     //[Error] converting to 'P' from initializer list would use explicit constructor 'P::P(int, int, int)'
P p6 (77,5,42);         //explicit P(int a, int b, int c)
	
fp( {47,11} );			//P(initializer_list<int>)
fp( {47,11,3} );	    //[Error] converting to 'const P' from initializer list would use explicit constructor 'P::P(int, int, int)'
fp( P{47,11} );			//P(initializer_list<int>)
fp( P{47,11,3} );		//P(initializer_list<int>)
	
P p11 {77, 5, 42, 500};	 	 //P(initializer_list<int>)
P p12 = {77, 5, 42, 500};    //P(initializer_list<int>)
P p13 {10};					 //P(initializer_list<int>)
	
```