---
title: Geekband C++面向对象高级编程 第三周作业
date: 2016-02-08 10:02
tags: Cpp
---

## 题目说明：

为下面的 Rectangle 和 Circle 类重写 getArea 虚函数。然后创建一个数组。使用一个循环,生成 10 个 Rectangle、10 个 Circle,根据循环遍历顺序为它们设置 no 编号,位置、长、宽、半径等其他信息取随机 1~10 之间的整数值,然后将它们加入到创建好的数组中。最后,将这个长度为 20 的数组中所有面积小于 50 的形状删除。将剩下的形状组成一个新的数组返回。					
注意: 1. 补齐任务所需的其他函数。2. 考虑正确的内存管理。3. 使用原生数组,不使用 vector 等容器。				
``` cpp
class Shape
{
    int no;
    public:
    virtual int getArea()=0;
};
class Point
{
    int x;
    int y;
};
class Rectangle: public Shape
{
    int width;
    int height;
    Point leftUp;
};
class Circle: public Shape
{
    Point center;
    int radius;
};
```

<!--more-->

## 解答

原始提交代码：[Geekband-cpp-w3.rar](\Geekband-cpp-w3.rar)

1.31更新：我重新制作了第3份代码（v3），估计这是本身题目想要的->返回数组，所以可以先看v3文件夹的代码。

由于对“将这个长度为 20 的数组中所有面积小于 50 的形状删除。将剩下的形状组成一个新的数组返回。”这句话的理解问题，我制作了两份代码，分别放在v1和v2文件夹中。

关于这道题目的一些思考：
* 关于v1：我将那句话理解为建立一个新的数组，并且把符合条件的元素拷贝过去（因为有的时候原始数据是需要保留的）。此时困扰我最久的是在筛选出面积不小于50的Shape后如何拷贝到新的数组里，一开始我用的是比较暴力的方式，遍历0-9的Shape，调用`Rectangle`，遍历10-19，调用`Circle`。这样存在一个麻烦的事情，假如需要再添加一个`Shape`，就必须要改写`filterShape`这个函数。而且如果题目需要再在这选出来的里面挑面积大于60的，由于不知道新数组中每个子类还有多少个元素，这样的写法修改起来就非常困难了。我尝试过重载赋值运算符（`Shape`类里面定义为虚函数），但编译器在`setShape`函数里面报错不能`new`一个抽象类，不清楚原因。最后终于想到了，因为`myshape[i]`自己知道自己是`Rectangle`还是`Circle`，可以写一个`copy()`的纯虚函数来解决这个问题。
* 关于v2：这是在看到微信群大家说其实是要删掉数组中不符合条件的元素，让它变成一个新数组。显然直接删掉会导致数组变成不连续的坑（也不能用链表），所以就想到了把所有符合条件的元素往前移的方法（本质上来说是改变了指针的指向）。相较v1，因为复制的是指针，效率比拷贝高，而且也不需要那几个`copy`函数了。假如题目的意思是原数组要删除不符合条件的，并且拷贝一份到新数组，那么可以结合v1的copy函数，这里就不做第三份代码了。
* <s>看翁恺老师C/C++课时24的时候，知道了如果父类里面有virtual函数，那么析构函数也必须virtual，</s>/应该说是因为需要用父类去管理子类（upcast）才需要析构函数是virtual。这里暂时让它们都是默认的。调试结果来看确实会先调用子类析构函数，再调用父类析构函数。
* 理论上来说Circle类的getArea应该返回double类型，但题目给的返回类型是int，纯虚函数在子类的实现返回类型需要相同，暂时还没想到解决方法。
* 题目说用一个循环生成10个Rectangle和10个Circle，但这样需要判断循环计数器i需要以10作为分界判断，并没有这个必要，所以我使用了两个循环。假如生成的是Rectangle还是Circle也通过随机的方式，那么要求一个循环到是合理的。
* 关于v3：经过1.30的直播，我知道了其实作业故意设计成不太合理的样子，其实是想我们按照这样的做法去做，从而发现一些问题。所以我写了一份v3。首先对于3份代码，从复用性角度讲，我把`cnt`（`shape`对象数量）作为一个参数传递进去。回到题目，最后的要求是返回一个数组，所以需要用到`**`（指针的指针），注意点2说注意内存管理，我大概就明白了，其实如何释放内存是这题的一个考察点，v1和v2都避开了这个问题。而且，filterShape函数需要返回`**`，就不可能同时返回`ncnt`（新数组的元素个数），所以`ncnt`需要作为参数传入，并且设置为引用传递（呼应第四周讲的`reference`专题）。另外，如果硬拗“小于50的形状要删掉”这句话的话，在`filterShape`函数里面`if`后面（52行）加上`else { delete myshape[i]; myshape[i] = nullptr;}` 这样做只是提前回收内存而已。 
* 面积小于50的这个50也可以设置成`filter_num`提高复用性。
* 我一开始的时候把三个析构函数都写成了`=default`，但是LLVM报错了：
``` bash
shape.h:19:21: warning: defaulted function definitions are a C++11 extension
      [-Wc++11-extensions]
        virtual ~Shape() = default;
                           ^
shape.h:40:10: error: exception specification of overriding function is more lax
      than base version
        virtual ~Rectangle() = default;
                ^
shape.h:19:10: note: overridden virtual function is here
        virtual ~Shape() = default;
                ^
shape.h:40:25: warning: defaulted function definitions are a C++11 extension
      [-Wc++11-extensions]
        virtual ~Rectangle() = default;
                               ^
shape.h:52:10: error: exception specification of overriding function is more lax
      than base version
        virtual ~Circle() = default;
                ^
shape.h:19:10: note: overridden virtual function is here
        virtual ~Shape() = default;
                ^
shape.h:52:22: warning: defaulted function definitions are a C++11 extension
      [-Wc++11-extensions]
        virtual ~Circle() = default;
```
	意思是它希望子类应该比基类更宽松吗？后来我都改成{}了，因为这道题目里面析构函数不需要特别做什么处理。


点评作业提到筛选合适面积时候，可以先用一次循环计算出符合条件的元素个数，再建立恰好大小的数组。这样就是牺牲时间换空间。而我的做法是牺牲的空间换时间。当然了，假如原始数据不需要保留的话，V2也是不错的办法。各有利弊，采用哪一种还是根据具体情况具体分析了。