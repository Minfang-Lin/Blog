---
title: Geekband C++面向对象高级编程 第二周作业
date: 2016-01-30 22:20
tags: Cpp
---

## 题目说明：

为 Rectangle 类实现构造函数,拷贝构造函数,赋值操作符,析构函数。

``` cpp
class Shape {
    int no; 
}; 
class Point {
    int x;
    int y; 
}; 
class Rectangle: public Shape {
    int width;
    int height;
    Point * leftUp;
public:
    Rectangle(int width, int height, int x, int y);
    Rectangle(const Rectangle& other);
    Rectangle& operator=(const Rectangle& other);
    ~Rectangle();
};
```

<!--more-->

## 解答

原始提交代码：

``` cpp
// rectangle.h
#ifndef __RECTANGLE_H__
#define __RECTANGLE_H__

#include <iostream>
using namespace std;

class Shape {                   
	int no_;
	static int order_;
	static int total_; 
public:
	Shape() { ++total_; no_ = ++order_; } 
	~Shape() { --total_; }
	int getNo() const { return no_; }
	static int ShapeTotal() { return total_; }
//	Shape& operator=(const Shape& other) { no_ = other.no_; return *this; }
//	Shape(const Shape& other) {	no_ = other.no_; ++total_; ++order_; }
};  
int Shape::order_ = 0;
int Shape::total_ = 0;
            
class Point {
	int x_;
	int y_;
public:
	Point(int x = 0, int y = 0): x_(x), y_(y) { }
	int getX() const { return x_; }
	int getY() const { return y_; }
};        
      
class Rectangle: public Shape {
private:
	int width_;
	int height_;
	Point* leftUp_;
public:
	Rectangle(int width, int height, int x, int y);
	Rectangle(const Rectangle& other);
	Rectangle& operator=(const Rectangle& other);
	~Rectangle(); 
	int getWidth() const { return width_; }    
	int getHeight() const { return height_; }    
	Point* getLeftUP() const { return leftUp_; }
};

inline
Rectangle::Rectangle(int width = 0, int height = 0, int x = 0, int y = 0): width_(width), height_(height) {
	leftUp_ = new Point(x,y);
}

inline
Rectangle& Rectangle::operator=(const Rectangle& other) {
//	Shape::operator=(other);
	if (this == &other)
		return *this;
	width_ = other.width_;
	height_ = other.height_;
	delete leftUp_;
	leftUp_ = new Point(*(other.leftUp_));
	return *this;
}

inline 
//Rectangle::Rectangle(const Rectangle& other): Shape(other), width_(other.width_), height_(other.height_)  {
Rectangle::Rectangle(const Rectangle& other): width_(other.width_), height_(other.height_) {
	leftUp_ = new Point(*(other.leftUp_));
}

inline
Rectangle::~Rectangle() {
	delete leftUp_;
	leftUp_ = nullptr;
}

ostream& operator<<(ostream& os, const Rectangle& rect) {
	os << rect.getNo()             << "\t"
	   << rect.getLeftUP()->getX() << "\t"
	   << rect.getLeftUP()->getY() << "\t"
	   << rect.getWidth()          << "\t"
	   << rect.getHeight()         << endl;
}

#endif
```

``` cpp
// rectangle_test.cpp
#include "rectangle.h" 
#include <iostream>
using namespace std;

int main(int argc, char** argv) {
	Rectangle r1(1,1,2,3);
	Rectangle r2; 
	Rectangle r3(r1);
	Rectangle r4;
	r4 = r2;
	cout << "Current total of shapes: " << Shape::ShapeTotal() << endl;
	cout << "No\tX\tY\tWidth\tHight" << endl;
	cout << r1 << r2;
	cout << r3;
	cout << r4;
	cout << "--------------------------------------------------" << endl;
	{
		Rectangle r5 = Rectangle(2,2,3,4);
		Rectangle r6 = r3;
		Rectangle r7(r2);
		Rectangle r8;
		r8 = r1;
		cout << "Current total of shapes: " << Shape::ShapeTotal() << endl;
		cout << "No\tX\tY\tWidth\tHight" << endl;
		cout << r1 << r2 << r3 << r4 << r5 << r6 << r7 << r8;
	}
	cout << "--------------------------------------------------" << endl;
	Rectangle r9(3,3,4,5);
	cout << "Current total of shapes: " << Shape::ShapeTotal() << endl;
	cout << "No\tX\tY\tWidth\tHight" << endl;
	cout << r1 << r2 << r3 << r4 << r9;
	return 0;
}
```

我使用了打印x和y地址的方法测试了拷贝构造及拷贝赋值是否是深拷贝，确定确实是深拷贝，在提交代码里面已经删去了相关代码。

关于本次作业的一些想法：

* 题目要求里面并没有说明要怎么处理父类`Shape`里面的`no`，我的理解是`no`是`Shape`的序号，也就是当第一个`Shape`对象被构造的时候`no`为1，第二个`Shape`对象被构造的时候`no`为2，依此类推。加上本周已经讲过了`static`，我想用`static`处理这个问题再好不过了。于是我给`Shape`类加上了两个私有成员`order_`和`total_`，`order_`用于记录当前编号到几号，`total_`用于记录当前有多少个`Shape`对象。与之相应的逻辑便产生了，在构造函数里面，`total_`和`order_`都需要增加1，在析构函数里`total_`需要减去1。随之而来的是另外一个问题，在子类`Rectangle`拷贝构造和拷贝赋值的时候，`no_`（序号）的值是否需要复制，我觉得按照逻辑上来说因为是一个新的`Shape`，是不需要复制的，这样就变得比较简单，我并不需要再去做`Shape`类的拷贝构造和拷贝赋值函数。但我还是试着做了一下（在提交代码时候已经把相应代码注释掉了，分别是`rectangle.h`里16、17、53、64这4行）。而确实这样做还是能够发现一些问题的，如果是拷贝赋值的话，因为本身已经生成了对象（调用了无参构造函数），所以对应的`order_`和`total_`都会处理好，但是拷贝构造的时候就需要处理`total_`和`order_`不然数据就不对了。
此外在`Shape`重载赋值运算符的时候我并没有检查`other`和`this`是否是同一个，因为一方面不涉及到指针，另一方面因为只复制一个值，只需要做一步，如果写了if假如不是同一个，必然多做一步，这步是没有必要的。不知道这样的考虑是否妥当。
* 在写测试代码的时候也发现了一个问题，假如代码是这样子的：
``` cpp
Rectangle r1 = Rectangle(2,2,3,4);
Rectangle r2;
```
  那么r2的序号并不会变成3。
  如果是这样写：
``` cpp
Rectangle r1;
r1 = Rectangle(2,2,3,4);
Rectangle r2;
```
   r2的序号就会变成3。
   说明直接初始化和赋值还是不一样的，不知道其中的机理如何（<-编译器优化）。
* 为什么用`static`修饰函数后不能再用`const`修饰？
``` cpp
static int ShapeTotal() { return total_; }
```
* 像Rectangle类里面的leftUp_的get函数，一般会是返回指针（`Point*`）还是返回所指向的内容（`Point`）？因为像下面这样也是可以的。请问一般使用哪一种是更为合理的做法。
``` cpp
Point getLeftUP() const { return *leftUp_; }
//输出时改成
rect.getLeftUP().getX() rect.getLeftUP().getY();
```
* 在Rectangle的析构函数里面，我还加了一句`leftUp_ = nullptr`，因为C里面`free(p)`后面经常加的`p = NULL`，我试了下载`delete leftUp_`后面加上了`cout << leftUp_.getX();`确实是可以打印出来的，说明此时`leftUp_`所指的空间只是释放并没有解除这种指向关系，但析构完了的话，这个`leftUp_`也就不存在了，在这里看起来，确实有点多此一举。
* 在拷贝赋值、拷贝构造的时候是否需要判断`leftUp_`是否是`nullptr`？我觉得按照我目前的做法没有必要，因为在创建`Rectangle`的时候不论是否有给参数，都会`new`一个`Point`对象并且把地址赋值给`leftUp_`，所以无论是在做拷贝赋值或是拷贝构造的时候都`other`的`leftUp_`都没有机会是`nullptr`。我忘记考虑new失败的情况了，但是`new`失败说明内存耗尽了，程序也运行不下去，再检查没有意义。而且`leftUp_`从逻辑上讲，只要这个`Rectangle`对象没有销毁，就应该存在，有一个`leftUP_`是`nullptr`的`Rectangle`对象存在是不合理的（如果我没理解错的话，我的代码本身并不会出现这样的情况）。所以我觉得不需要判断。
* 因为题目给的变量名和类成员的名字是一样的，我按照编码规范给所有的私有成员变量加了下划线以示区别。
* 由于代码很短，`Shape`、`Point`、`Rectangle`类的声明都放在了同一个`.h`文件里。


受到第二周字符串示例代码的影响，大家在拷贝赋值的时候，都上来把leftUp释放了，假如`this`和`other`的都不是空，到确实可以直接拷贝过去更合理些。
这次直播中主要讲了如果`leftUp`是`nullptr`的情况。如前面第6点说的，从目前代码来看，似乎是不需要做判断的，但后来想到翁恺老师的C/C++课讲因为有一个东西叫做指针，利用指针可以干一些很邪恶的事情。所以假如代码是这样子的：

``` cpp
#include <iostream>
using namespace std; 

class A {
public:
	int i;
	int *p;
	A(): i(10), p(&i) { }
};

int main(){
	A a;
	int* pt = (int*)&a;
	cout << a.p << endl;
	cout << *(a.p) << endl;
	pt++;
	*pt = 0;
	if(a.p == nullptr) cout << "true" << endl;
	return 0;
}
```

不过我也不知道是不是这样的问题（用32位编译）。
然后有同学因为学了第三周后认为父类（`Shape`）的析构函数应该设置为虚函数，从我目前代码的角度来讲因为没有抽象类是不需要的，看cpp的测试程序就可以，而且侯捷老师也讲过，以继承的方式析构时就是先调用子类，再调用父类的，编译器也会自动加。
下面是我把`leftUp`判断是否是`nullptr`修改后的代码：

``` cpp
#ifndef __RECTANGLE_H__
#define __RECTANGLE_H__

#include <iostream>
using namespace std;

class Shape {                   
	int no_;
	static int order_;
	static int total_; 
public:
	Shape() { ++total_; no_ = ++order_; } 
	~Shape() { --total_; }
	int getNo() const { return no_; }
	static int ShapeTotal() { return total_; }
//	Shape& operator=(const Shape& other) { no_ = other.no_; return *this; }
//	Shape(const Shape& other) {	no_ = other.no_; ++total_; ++order_; }
};  
int Shape::order_ = 0;
int Shape::total_ = 0;
            
class Point {
	int x_;
	int y_;
public:
	Point(int x = 0, int y = 0): x_(x), y_(y) { }
	int getX() const { return x_; }
	int getY() const { return y_; }
};        
      
class Rectangle: public Shape {
private:
	int width_;
	int height_;
	Point* leftUp_;
public:
	Rectangle(int width, int height, int x, int y);
	Rectangle(const Rectangle& other);
	Rectangle& operator=(const Rectangle& other);
	~Rectangle(); 
	int getWidth() const { return width_; }    
	int getHeight() const { return height_; }    
	Point* getLeftUP() const { return leftUp_; }
};

inline
Rectangle::Rectangle(int width = 0, int height = 0, int x = 0, int y = 0): width_(width), height_(height), leftUp_(new Point(x,y)) { }

inline
Rectangle& Rectangle::operator=(const Rectangle& other) {
//	Shape::operator=(other);
	if (this == &other)
		return *this;
	width_ = other.width_;
	height_ = other.height_;
	
	if (leftUp_ != nullptr) {
		if (other.leftUp_ != nullptr) {  
			*leftUp_ = *(other.leftUp_);
		} else {
			delete leftUp_;
			leftUp_ = nullptr;
		}
	} else {
		if (other.leftUp_ != nullptr){
			leftUp_ = new Point(*(other.leftUp_));
		}
	}
	
	return *this;
}

inline 
//Rectangle::Rectangle(const Rectangle& other): Shape(other), width_(other.width_), height_(other.height_)  {
Rectangle::Rectangle(const Rectangle& other): width_(other.width_), height_(other.height_) {
	if(other.leftUp_ != nullptr)
		leftUp_ = new Point(*(other.leftUp_));
	else
		leftUp_ = nullptr;	
}

inline
Rectangle::~Rectangle() {
	delete leftUp_;
	leftUp_ = nullptr;
}

ostream& operator<<(ostream& os, const Rectangle& rect) {
	os << rect.getNo()             << "\t"
	   << rect.getLeftUP()->getX() << "\t"
	   << rect.getLeftUP()->getY() << "\t"
	   << rect.getWidth()          << "\t"
	   << rect.getHeight()         << endl;
}

#endif
```

``` cpp
#include "rectangle.h" 
#include <iostream>
using namespace std;

int main(int argc, char** argv) {
	Rectangle r1(1,1,2,3);
	Rectangle r2; 
	Rectangle r3(r1);
	Rectangle r4;
	r4 = r2;
	cout << "Current total of shapes: " << Shape::ShapeTotal() << endl;
	cout << "No\tX\tY\tWidth\tHight" << endl;
	cout << r1 << r2;
	cout << r3;
	cout << r4;
	cout << "--------------------------------------------------" << endl;
	{
		Rectangle r5 = Rectangle(2,2,3,4);
		Rectangle r6 = r3;
		Rectangle r7(r2);
		Rectangle r8;
		r8 = r1;
		cout << "Current total of shapes: " << Shape::ShapeTotal() << endl;
		cout << "No\tX\tY\tWidth\tHight" << endl;
		cout << r1 << r2 << r3 << r4 << r5 << r6 << r7 << r8;
	}
	cout << "--------------------------------------------------" << endl;
	Rectangle r9(3,3,4,5);
	cout << "Current total of shapes: " << Shape::ShapeTotal() << endl;
	cout << "No\tX\tY\tWidth\tHight" << endl;
	cout << r1 << r2 << r3 << r4 << r9;
	return 0;
}
```

<div class = "tip">注意：cpp里面仍然是两种实现方式，我保留的是序号不能相同的，当然你认为序号可以是相同的话，只需要把16、17、51、74行的代码取消注释，75行代码注释掉就可以了。51行代码也放到if条件后更合理。</div>