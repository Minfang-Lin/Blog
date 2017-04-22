---
title: Geekband C++面向对象高级编程 第一周作业
date: 2016-01-30 20:35
tags: Cpp
---

## 题目说明：

为`Date`类实现如下成员：

* 构造器，可以初始化年、月、日。
* 大于、小于、等于（> 、< 、==）操作符重载，进行日期比较。
* `print()`打印出类似2015-10-1这样的格式。

然后创建两个全局函数：

* 第1个函数 `CreatePoints`生成10个随机的`Date`，并以数组形式返回；
* 第2个函数 `Sort` 对第1个函数`CreatePoints`生成的结果，将其按照从小到大进行排序。

最后在`main`函数中调用`CreatePoints`，并调用`print`将结果打印出来。然后调用`Sort`函数对前面结果处理后，并再次调用`print`将结果打印出来。

``` cpp
class Date{
	int year;
	int month;
	int day;
}
```

<!--more-->

## 解答

原始提交的代码：

``` cpp
// data.h
#ifndef __DATE_H__
#define __DATE_H__

#include <iostream>
using namespace std;

class Date {
private:
	int year;
	int month;
	int day;
public:
	Date (int yy = 1970, int mm = 1, int dd = 1): year(yy), month(mm), day(dd) { }
	bool operator == (const Date&);
	bool operator > (const Date&);
	bool operator < (const Date&);
	Date& operator = (const Date&);
	void print() const;
};

//----------------------------------------------------------------
inline bool
Date::operator == (const Date& x) {
	return (year == x.year) && (month == x.month) && (day == x.day);
}
//----------------------------------------------------------------
inline bool
Date::operator > (const Date& x) {
	if (year  > x.year)  return true;
	if (year  < x.year)  return false;
	if (month > x.month) return true;
	if (month < x.month) return false;
	return day > x.day;
}
//----------------------------------------------------------------
inline bool
Date::operator < (const Date& x) {
	return !((*this == x) || (*this > x));
}
//----------------------------------------------------------------
inline Date&
Date::operator = (const Date& x) {
	year = x.year;
	month = x.month;
	day = x.day;
	return *this;
}
//----------------------------------------------------------------
inline void
Date::print() const {
	cout << year << "-" << month << "-" << day << endl;
}
//----------------------------------------------------------------

#endif 
```

在Stack建立数组的方式

``` cpp
// date_test.cpp
#include "date.h"
#include <cstdlib>
#include <ctime>
#include <algorithm>
#include <iostream>
using namespace std;
const int DATE_NUMBER = 10;

void CreatePoints (Date* );
void Sort (Date* );
void Print (Date* );

//----------------------------------------------------------------
int main(int argc, char** argv) { 
	Date mydate[DATE_NUMBER];
	CreatePoints(mydate);
	cout << "========== before sort ==========" << endl;
	Print(mydate);
	Sort(mydate);
	cout << "========== after sort ==========" << endl;
	Print(mydate);
	return 0;
}
//----------------------------------------------------------------
void CreatePoints (Date* mydate) {
	int day[12] = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
	time_t t;
	srand((unsigned)time(&t));
	for (int i=0; i<DATE_NUMBER; ++i) {
		int yy, mm, dd;
		yy = 1970 + rand()%80;
		mm = rand()%12 + 1;
		day[1] = 28;
		if ((yy%4==0 && yy%100!=0) || yy%400==0) {
			day[1] = 29;
		}
		dd = rand()%day[mm-1] + 1;
		mydate[i] = Date(yy, mm, dd);
	}
}
//----------------------------------------------------------------
void Sort (Date* mydate) {
	sort(mydate, mydate+DATE_NUMBER);
}
//----------------------------------------------------------------
void Print (Date* mydate){
	for (int i=0; i<DATE_NUMBER; ++i) {
		mydate[i].print();
	}
}
```

在Heap内建立数组的方式

``` cpp
#include "date.h"
#include <cstdlib>
#include <ctime>
#include <algorithm>
#include <iostream>
using namespace std;
const int DATE_NUMBER = 10;

Date* CreatePoints ();
void Sort (Date* );
void Print (Date* );

//----------------------------------------------------------------
int main(int argc, char** argv) { 
	Date *mydate = CreatePoints();
	cout << "========== before sort ==========" << endl;
	Print(mydate);
	Sort(mydate);
	cout << "========== after sort ==========" << endl;
	Print(mydate);
	delete [] mydate;
	return 0;
}
//----------------------------------------------------------------
Date* CreatePoints () {
	Date *mydate = new Date[DATE_NUMBER];
	int day[12] = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
	time_t t;
	srand((unsigned)time(&t));
	for (int i=0; i<DATE_NUMBER; ++i) {
		int yy, mm, dd;
		yy = 1970 + rand()%80;
		mm = rand()%12 + 1;
		day[1] = 28;
		if ((yy%4==0 && yy%100!=0) || yy%400==0) {
			day[1] = 29;
		}
		dd = rand()%day[mm-1] + 1;
		mydate[i] = Date(yy, mm, dd);
	}
	return mydate;
}
//----------------------------------------------------------------
void Sort (Date* mydate) {
	sort(mydate, mydate+DATE_NUMBER);
}
//----------------------------------------------------------------
void Print (Date* mydate){
	for (int i=0; i<DATE_NUMBER; ++i) {
		mydate[i].print();
	}
}
```

v1文件夹里的代码是我写的第一份，与第二份（v2）不同的是`CreatePoints`的返回类型。v1是我认为比较合理的代码（尽管不符合题目要求，但更符合逻辑）。

在做作业的过程当中有以下几个思考：

* 一直让我很在意的是全局函数`CreatePoints`需要以数组（确切的说是指针）形式返回，这样势必要在`CreatePoints`动态创建一个数组，但目前还没讲到`new`的部分（我只知道跟C语言里面的`malloc`差不多），这样就存在`delete`时机的问题。假设`CreatePoints`是我设计好给别人用的一个接口，那么用户需要记得用一个指针去接收生成好的10个`Date`数据，并且还需要记得`delete`，这无疑会对使用者造成负担。所以最后我选择在`main`里面建立一个数组，并且将指针传入，在`CreatePoints`只是去重新用随机时间戳改写数组（这导致也需要重载一下赋值运算符）。当然用户也需要记得把数组作为参数。当然等后面学了动态内存分配后我想可以改进一下。
* 我本来想把`Date`的声明和定义分离，声明在`.h`文件里，定义在`.cpp`里面，但`.cpp`里一开始把每个函数`inline`，编译过不了，原来`inline`的需要写在`.h`里面，这是比较特殊的情况。
* 我在`print()`加了可以影响格式的`endl`，一般输出打印的是不会加影响格式的。但是如果不加怎么换行呢，也许还得重载`<<`，以流来输出，这样`print`就需要参数，但是要求里面的`print`是没有参数的，这也是待讨论的地方。
* 因为重载了`>`、`<`，要做排序，自然就想到了利用标准库里面的`sort`，所以直接调用了。但是我在Dev C++里面编译没有问题，mac内置的会过不了编译（报错信息好像是因为实参不是`const`，我没明白）。
* 最后是关于每个东西放哪里的问题，`date.h`里面应该只有对类的定义，以及这个类能够提供的一些功能，所以我把`CreatePoints`和`Sort`都放到了`date_test.cpp`里面，包括数量10，也并不属于类的一部分，都放到了`date_test.cpp`中。不知道一般是如何分配每个模块的，我这样的做法是否合理。

## 2016.01.19更新：

由于群里有不少人强调题目里面要求的返回数组这点，于是我按照上面第一点说的做了几行代码的修改，`date_test.cpp`里的第15、21、26、41行。（原始代码放在附件的v1文件夹，修改后的在v2文件夹），正像第一点说的，假如`CreatePoints`需要返回数组，那么需要在`CreatePoints`去申请内存，这样用户需要在主函数记得`delete`，个人认为并不合理。假如内存是在`main`里面申请的，那么和使用内置数组并没有区别。题目也没有需要非常大的数组（导致必须用动态申请内存的方法）。

对于这个`Date`类来说，是不需要重载赋值运算符的，直接使用默认生成的就可以（做第一周作业时我还以为只有构造函数和析构函数会自动生成默认的）。

第二，关于数组长度，对于目前程序来说，长度是唯一的，所以并没有以参数的形式传入`CreatePoints`、`Sort`、`Print`这三个函数里面。不过确实，假如这三个函数脱离限制，需要单独使用，以复用性角度讲，将长度以参数传入更加合理一些。

修改后的代码：

``` cpp
//date.h
#ifndef __DATE_H__
#define __DATE_H__

#include <iostream>
using namespace std;

class Date {
private:
	int year;
	int month;
	int day;
public:
	Date (int yy = 1970, int mm = 1, int dd = 1): year(yy), month(mm), day(dd) { }
	bool operator == (const Date&);
	bool operator > (const Date&);
	bool operator < (const Date&);
	void print() const;
};

//----------------------------------------------------------------
inline bool
Date::operator == (const Date& x) {
	return (year == x.year) && (month == x.month) && (day == x.day);
}
//----------------------------------------------------------------
inline bool
Date::operator > (const Date& x) {
	if (year  > x.year)  return true;
	if (year  < x.year)  return false;
	if (month > x.month) return true;
	if (month < x.month) return false;
	return day > x.day;
}
//----------------------------------------------------------------
inline bool
Date::operator < (const Date& x) {
	return !((*this == x) || (*this > x));
}
//----------------------------------------------------------------
inline void
Date::print() const {
	cout << year << "-" << month << "-" << day << endl;
}
//----------------------------------------------------------------

#endif 
```

``` cpp
//date_test.cpp
#include "date.h"
#include <cstdlib>
#include <ctime>
#include <algorithm>
#include <iostream>
using namespace std;

Date* CreatePoints (int);
void Sort (Date*, int);
void Print (Date*, int);

//----------------------------------------------------------------
int main(int argc, char** argv) { 
	const int DATE_NUMBER = 10;
	Date *mydate = CreatePoints(DATE_NUMBER);
	cout << "========== before sort ==========" << endl;
	Print(mydate, DATE_NUMBER);
	Sort(mydate, DATE_NUMBER);
	cout << "========== after sort ==========" << endl;
	Print(mydate, DATE_NUMBER);
	delete [] mydate;
	return 0;
}
//----------------------------------------------------------------
Date* CreatePoints (int count) {
	Date *mydate = new Date[count];
	int day[12] = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
	time_t t;
	srand((unsigned)time(&t));
	for (int i=0; i<count; ++i) {
		int yy, mm, dd;
		yy = 1970 + rand()%80;
		mm = rand()%12 + 1;
		day[1] = 28;
		if ((yy%4==0 && yy%100!=0) || yy%400==0) {
			day[1] = 29;
		}
		dd = rand()%day[mm-1] + 1;
		mydate[i] = Date(yy, mm, dd);
	}
	return mydate;
}
//----------------------------------------------------------------
void Sort (Date* mydate, int count) {
	sort(mydate, mydate+count);
}
//----------------------------------------------------------------
void Print (Date* mydate, int count){
	for (int i=0; i<count; ++i) {
		mydate[i].print();
	}
}
```