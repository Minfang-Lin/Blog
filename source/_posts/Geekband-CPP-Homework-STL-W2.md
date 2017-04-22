---
title: Geekband C++ STL 第二周作业
date: 2016-04-09 23:40
tags: Cpp
---

## 题目说明：

为以下Programmer对象提供一个基于Id并且升序的仿函数ProgrammerIdGreater，使得Programmer对象可以在set中以Id 排序存放。
(1) 将Programmer(1, L"Scott Meyers")、Programmer(2, L"MartinFowler")、Programmer(3, L"Bill Gates")、Programmer(4, L"P.J.Plaught")、Programmer(5, L"Stanley B. Lippman")、Programmer(6, L"Andrei Alexandrescu")插入到一个set 中；
(2) 通过for_each 遍历set，并且使用Programmer 对象的Print 方法打印出对象，结果应该如下所示；
``` bash
[6] : Andrei Alexandrescu
[5] : Stanley B. Lippman
[4] : P.J. Plaught
[3] : Bill Gates
[2] : Martin Fowler
[1] : Scott Meyers
```
(3) 在set 中查找Id 为3、名称为“Bill Gates”的对象；
(4) 如果(2)中找到目标对象，则将其名称改成“David Vandevoorde”，再通过for_each 重新打印set，查看是否真的修改成功了；
(5) 重新定义一个set(命名为set2)，将Programmer 在set2 中排序方式改为通过Name 来排序，为此还需重新定义一个仿函数用于比较Name，请写出该仿函数，名字可能类似ProgrammerNameComparer;
(6) 通过for_each 重打印set2，验证set2 中的元素是否是按照名字来排序的。
``` cpp
struct Programmer
{
    Programmer(const int id, const std::wstring name)
        : Id(id), Name(name)
    {
    }
    void Print() const
    {
        std::wcout << L"[" << Id << L"] : " << Name << std::endl;
    }
    int Id;
    std::wstring Name;
};
struct ProgrammerIdGreater { ... }
```


<!--more-->

## 解答

``` cpp
#ifndef __PROGRAMMER_H__
#define __PROGRAMMER_H__

#include <iostream>
#include <string>
using namespace std;
//----------------------------------------------------------------
struct Programmer {
	Programmer(const int id, const wstring name) :
			Id(id), Name(name) {
	}
	void Print() const {
		wcout << L"[" << Id << L"]:" << Name << endl;
	}
	const int getId() const {
		return Id;
	}
	const wstring& getName() const {
		return Name;
	}
	void setName(const wstring& name) {
		Name = name;
	}
private:
	int Id;
	wstring Name;
};
//----------------------------------------------------------------
struct ProgrammerIdGreater: public binary_function<Programmer, Programmer, bool> {
	bool operator()(const Programmer& p1, const Programmer& p2) const {
		return p1.getId() > p2.getId();
	}
};
//----------------------------------------------------------------
struct ProgrammerNameComparer: public binary_function<Programmer, Programmer,
		bool> {
	bool operator()(const Programmer& p1, const Programmer& p2) const {
		return p1.getName() < p2.getName();
	}
};
//----------------------------------------------------------------

#endif // programmer.h
```

``` cpp
#include "programmer.h" 
#include <iostream>
#include <set>
#include <algorithm>
#include <functional>
using namespace std;

//----------------------------------------------------------------
int main(int argc, char** argv) {
	// Step 1
	const int nSize = 6;
	const Programmer programmerArray[nSize] = { Programmer(1, L"Scrott Meyers"),
			Programmer(2, L"Martin Fowler"), Programmer(3, L"Bill Gates"),
			Programmer(4, L"P.J. Plaught"), Programmer(5,
					L"Stanley B. Lippman"), Programmer(6,
					L"Andrei Alexandrescu"), };
	set<Programmer, ProgrammerIdGreater> set1(programmerArray,
			programmerArray + nSize);

	// Step 2
	for_each(set1.begin(), set1.end(),
			bind(Programmer::Print, placeholders::_1));
	cout << "------------------------------" << endl;

	// Step 3
	set<Programmer, ProgrammerIdGreater>::iterator it = set1.find(
			Programmer(3, L"Bill Gates"));

	// Step 4
	if (it == set1.end()) {
		cout << "not found." << endl;
		exit(0);
	} else {
		const_cast<Programmer&>(*it).setName(L"David Vandevoorde");
	}
	for_each(set1.begin(), set1.end(), mem_fun_ref(&Programmer::Print));
	cout << "------------------------------" << endl;

	// Step 5
	set<Programmer, ProgrammerNameComparer> set2(set1.begin(), set1.end());

	// Step 6
	for_each(set2.begin(), set2.end(), mem_fn(&Programmer::Print));

	return 0;
}
```

可以通过直接拷贝的方式，而不是一个个元素insert，因为set每次insert会插入合适的位置（排序）。
在判断是否找到的时候，可以把更简单的处理放在前面，避免头重脚轻。