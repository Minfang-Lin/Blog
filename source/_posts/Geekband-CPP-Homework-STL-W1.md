---
title: Geekband C++ STL 第一周作业
date: 2016-03-06 10:00
tags: Cpp
---

## 题目说明：

给定一个 vector:v1 = [0, 0, 30, 20, 0, 0, 0, 0, 10, 0]，希望通过not_equal_to 算法找到到不为零的元素，并复制到另一个 vector: v2。


<!--more-->

## 解答

在查了`not_equal_to`用法后，写下的第一个版本：
``` cpp
#include <vector>
#include <functional>
#include <iostream>
using namespace std;

int main(int argc, char** argv) {
  vector<int> v1 { 0, 0, 30, 20, 0, 0, 0, 0, 10, 0 };
  vector<int> v2;
  const int FIND_NUM = 0;
  for (int i : v1) {
    if (not_equal_to<int>()(i, FIND_NUM)){
      v2.push_back(i);
    }
  }
  for (int i : v2) 
    cout << i << endl;
  return 0;
}
```
也就是把`i!=FIND_NUM`改成了`not_equal_to<int>()(i, FIND_NUM)`这样的判断条件，虽然符合题目要求，但不符合题目初衷。

进一步的，我使用了find_if算法进行不为0元素的查找，由于每次都只能找到第一个不为0，所以迭代器在每次找到一个元素后需要移动到下一个元素的位置。代码如下：
``` cpp
#include <vector>
#include <functional>
#include <algorithm>
#include <iostream>
using namespace std;

int main(int argc, char** argv) {
  vector<int> v1 { 0, 0, 30, 20, 0, 0, 0, 0, 10, 0 };
  vector<int> v2;
  const int FIND_NUM = 0;
  vector<int>::iterator it = find_if(v1.begin(), v1.end(), bind1st(not_equal_to<int>(), FIND_NUM));
  while (it != v1.end()) {
    v2.push_back(*it);
    it = find_if(it+1, v1.end(), bind1st(not_equal_to<int>(), FIND_NUM));
  }
  for (int i : v2) 
    cout << i << endl;
  return 0;
}
```

也是符合题目要求，但不符合题目意图。我想这题是希望通过算法组合，直接一行代码搞定需求。所以像`find_if`这样只能寻找到第一个符合条件元素的也不适用于这题。继而我找到了`copy_if`。代码如下：
``` cpp
#include <vector>
#include <functional>
#include <algorithm>
#include <iostream>
using namespace std;

int main(int argc, char** argv) {
  vector<int> v1 { 0, 0, 30, 20, 0, 0, 0, 0, 10, 0 };
  vector<int> v2 (v1.size());
  const int FIND_NUM = 0;
  vector<int>::iterator it = copy_if(v1.begin(), v1.end(), v2.begin(), bind1st(not_equal_to<int>(), FIND_NUM));
  v2.resize(distance(v2.begin(), it));
  for (int i : v2) 
    cout << i << endl;
  return 0;
}
```

其中的问题显而易见，由于无法使用`push_back`，必须先让v2拥有v1一样的大小，才能保证不溢出，并且需要一个迭代器记录copy后v2中符合条件的最后一个元素位置，进行resize。

尽管vector在内部处理的时候会向系统申请一块空间，本质上来说v2初始化为v1相同的大小不一定就浪费空间了，但是这样的做法还是过于麻烦。

上面的做法只差了一步，如何每次插入到v2后面一个位置，通过back_inserter可以解决。代码如下：
``` cpp
#include <vector>
#include <functional>
#include <algorithm>
#include <iterator>
#include <iostream>
using namespace std;

int main(int argc, char** argv) {
  vector<int> v1 { 0, 0, 30, 20, 0, 0, 0, 0, 10, 0 };
  vector<int> v2;
  const int FIND_NUM = 0;
  copy_if(v1.begin(), v1.end(), back_inserter(v2), bind1st(not_equal_to<int>(), FIND_NUM));
  for (int i : v2) 
    cout << i << endl;
  return 0;
}
```
*注：去掉`#include <iterator\>`也是可以编译运行的。

由于`copy_if`是C++11才有的，之前可以用`remove_copy_if`替代，且筛选条件也需要做下修改，代码如下：
``` cpp
#include <vector>
#include <functional>
#include <algorithm>
#include <iterator>
#include <iostream>
using namespace std;

int main(int argc, char** argv) {
  vector<int> v1 { 0, 0, 30, 20, 0, 0, 0, 0, 10, 0 };
  vector<int> v2;
  const int FIND_NUM = 0;
  remove_copy_if(v1.begin(), v1.end(), back_inserter(v2), not1(bind1st(not_equal_to<int>(), FIND_NUM)));
  for (int i : v2) 
    cout << i << endl;
  return 0;
}
```

用课上讲的`erase`+`remove_if`也可以，但需要事先拷贝v1所有元素到v2，或者先修改了v1的数据，留下的拷贝到v2，代码如下：
``` cpp
#include <vector>
#include <functional>
#include <algorithm>
#include <iterator>
#include <iostream>
using namespace std;

int main(int argc, char** argv) {
  vector<int> v1 { 0, 0, 30, 20, 0, 0, 0, 0, 10, 0 };
  const int FIND_NUM = 0;
  v1.erase(remove_if(v1.begin(), v1.end(), not1(bind1st(not_equal_to<int>(), FIND_NUM))), v1.end());
  vector<int> v2(v1);
  for (int i : v2) 
    cout << i << endl;
  return 0;
}
```

改变原数组的方法并不可取。另外也可以使用`partition_copy`将v1分成0和非0元素两组，代码如下：
``` cpp
#include <vector>
#include <functional>
#include <algorithm>
#include <iterator>
#include <iostream>
using namespace std;

int main(int argc, char** argv) {
  vector<int> v1 { 0, 0, 30, 20, 0, 0, 0, 0, 10, 0 };
  vector<int> v2, v3;
  const int FIND_NUM = 0;
  partition_copy(v1.begin(), v1.end(), back_inserter(v2), back_inserter(v3), bind1st(not_equal_to<int>(), FIND_NUM));
  for (int i : v2) 
    cout << i << endl;
  return 0;
}
```

以上所有代码当中的
``` cpp
for (int i : v2)
  cout << i << endl;
```
也可以改成
``` cpp
copy(v2.begin(), v2.end(), ostream_iterator<int>(cout, "\n"));
```

在有算法可以使用的时候，尽量使用算法，比如将v1有效的数据拷贝入v2的时候，不考虑用`push_back/insert`，因为后者相对来说效率低。

C++11标准里有`bind()`可以替代原来的`bind1st()`和`bind2nd()`，所以也可以改写成下面这样：
``` cpp
#include <vector>
#include <functional>
#include <algorithm>
#include <iterator>
#include <iostream>
using namespace std;
using namespace std::placeholders;

int main(int argc, char** argv) {
  vector<int> v1 { 0, 0, 30, 20, 0, 0, 0, 0, 10, 0 };
  vector<int> v2;
  const int FIND_NUM = 0;
  auto func = bind(not_equal_to<int>(), _1, FIND_NUM);
  copy_if(v1.begin(), v1.end(), back_inserter(v2), func);
  for (int i : v2) 
    cout << i << endl;
  return 0;
}
```