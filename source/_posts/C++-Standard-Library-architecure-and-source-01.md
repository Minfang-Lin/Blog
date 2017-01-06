---
title: C++ STL体系结构与内核分析（一）
date: 2016-12-01 20:00
tags: [C++,STL]
---

标准库是以header files形式呈现。

* C++标准库的header files不带副档名(`.h`)，例如`#inlcude <vector>`
* 新式C header files不带副档名`.h`，例如`#include <cstdio>`
* 旧式C header files（带副档名`.h`），仍然可用，例如`#include <stdio.h>`
* 新式headers内的组件封装于`namespace std`
* 旧式headers内的组件**不被**封装于`namespace std`

重要的网站

* [cplusplus.com](http://www.cplusplus.com)
* [cppreference.com](http://www.cppreference.com)
* [gcc.gnu.org](http://gcc.gnu.org)

STL六大部件（Components）

* 容器（Containers）
* 分配器（Allocators）
* 算法（Algorithms）
* 迭代器（Iterators）
* 适配器（Adapters)
* 仿函数（Functions）

