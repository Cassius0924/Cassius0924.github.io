---
title: 'C++ STL 常用容器和迭代器学习笔记'
draft: false
date: 2024-06-28T19:50:50+08:00
description: ''
author: 'Cassius0924'
tags: ["C++", "STL", "Data Structure", "Algorithm", "Notes"]
---

# C++ STL 常用容器和迭代器学习笔记

## 常用容器

1. 序列容器

    - `vector`: 动态数组，随机插入/删除 `O(n)`，随机访问 `O(1)`，尾插 `O(1)`
    
    - `array`: 静态数组，不支持插入/删除，随机访问 `O(1)`

    - `deque`: 双端队列，头尾插入/删除 `O(1)`，随机访问 `O(1)`，中间插入/删除 `O(n)`

    - `list`: 双向链表，插入/删除 `O(1)`，不支持随机访问

    - `forward_list`: 单向链表，插入/删除 `O(1)`，不支持随机访问

2. 关联容器（底层实现为 **红黑树** ）

    - `set`: 有序集合，插入/删除/查找 `O(logn)`

    - `map`: 有序映射，插入/删除/查找 `O(logn)`

    - `multiset`: 有序多重集合，插入/删除/查找 `O(logn)`

    - `multimap`: 有序多重映射，插入/删除/查找 `O(logn)`

3. 无序容器（底层实现为 **哈希表** ）

    - `unordered_set`: 无序集合，插入/删除/查找 `O(1)`

    - `unordered_map`: 无序映射，插入/删除/查找 `O(1)`

    - `unordered_multiset`: 无序多重集合，插入/删除/查找 `O(1)`

    - `unordered_multimap`: 无序多重映射，插入/删除/查找 `O(1)`

4. 容器适配器

    - `stack`: 栈，后进先出，只能在栈顶插入/删除元素

    - `queue`: 队列，先进先出，只能在队尾插入，在队头删除元素

    - `priority_queue`: 优先队列，元素按照一定规则排序，每次取出的是最大/最小元素，底层实现为堆

## vector

``` c++
#include <iostream>
#include <vector>
using namespace std;

// vector使用示例
int main() {
    vector<int> vec = {1, 2, 3, 4, 5};

    // 尾部插入元素：复杂度为O(1)
    vec.push_back(6);
    // 尾部删除元素：复杂度为O(1)
    vec.pop_back();
    // 随机插入和删除元素：复杂度为O(n)
    vec.insert(vec.begin() + 1, 3);
    vec.erase(vec.begin() + 1);

    // vector的大小
    cout << vec.size() << endl;
    // 获取vector的容量
    cout << vec.capacity() << endl;
    // 判断vector是否为空
    cout << vec.empty() << endl;

    // 获取vector的第一个元素和最后一个元素
    cout << vec.front() << endl;
    cout << vec.back() << endl;
    // 访问指定位置的元素
    cout << vec[2] << endl;
    cout << vec.at(2) << endl; // at函数会检查索引是否越界，更安全

    vector<int> vec2 = {7, 8, 9, 10};
    vec.swap(vec2); // 交换两个vector的元素

    // 清空vector
    vec.clear();
}
```

vector 常用的成员函数：

- 元素访问
    - `front()`: 返回第一个元素

    - `back()`: 返回最后一个元素

    - `at()`: 访问指定位置的元素，会检查索引是否越界

- 迭代器
    - `begin()`: 返回指向第一个元素的迭代器

    - `end()`: 返回指向最后一个元素的迭代器

    - `rbegin()`: 返回指向最后一个元素的逆向迭代器

    - `rend()`: 返回指向第一个元素的逆向迭代器   

    - `cbegin()`、`cend()`、`crbegin()`、`crend()`: 返回常量迭代器，不允许修改元素

- 容量

    - `size()`: 返回vector中元素的个数

    - `max_size()`: 返回当前程序中的vector最大可以容纳的元素个数，通常是一个很大的值

    - `capacity()`: 返回vector的容量

    - `empty()`: 判断vector是否为空

    - `reserve()`: 为vector预留空间

    - `shrink_to_fit()`: 将vector的容量缩小到和元素个数相同

- 元素修改

    - `assign()`: 为vector赋值

    - `assign_range()`: 为vector赋值一个范围内的元素（C++23）

    - `push_back()`: 尾部插入元素

    - `emplace_back()`: 原地构造并插入元素到尾部

    - `pop_back()`: 尾部删除元素

    - `insert()`: 插入元素

    - `emplace()`: 原地构造并插入元素

    - `insert_range()`: 插入元素一个范围内的元素（C++23）

    - `append_range()`: 尾部插入元素一个范围内的元素（C++23）

    - `erase()`: 删除元素

    - `resize()`: 改变vector的大小

    - `swap()`: 交换两个vector的元素

    - `clear()`: 清空vector


### vector 的底层实现

vector 的底层实现是一个动态数组，当元素个数超过容量时，会重新分配内存，将原来的元素拷贝到新的内存空间中。

值得一提的是，无论 vector 是直接创建还是通过 `new` 关键词创建，vector 的 **元素** 都是在 **堆上** 分配的。

![](https://s2.loli.net/2024/06/28/FgIQcELldJroWq5.jpg)

### vector 的扩容机制

vector 的扩容取决于编译器的实现，在 GCC、Clang 中，vector 是以 **2 倍** 扩容的，即当元素个数超过容量时，会重新分配一个 2 倍大小的内存空间。在 MSVC 中，vector 是以 **1.5 倍** 扩容的。

假设我们有一个 A 类，如果我们在 vector 中存储 A 类的对象，当 vector 扩容时，A 类的对象会被拷贝到新的内存空间中，这是一笔不小的开销！

``` c++
int main() {
    vector<A> vec{1, 2, 3, 4, 5};
    cout << vec.data() << endl; // 输出：0x15be05f20
    vec.reserve(100);   // 扩容
    cout << vec.data() << endl; // 输出：0x15be06100
    // 从输出结果可以看出，vector 元素地址在内存的位置发生了变化 5f20 -> 6100
}
```

![](https://s2.loli.net/2024/06/28/Otu9nvzW85DCpGj.jpg)

### vector 迭代器失效

如果在插入元素时引起了内存的重新分配，那么原来的迭代器就会失效。例如下面代码：

``` c++
int main() {
    vector<int> vec = {1, 2, 3, 4, 5};  //此时 size 和 capacity 都是 5

    auto it = vec.begin();
    cout << *it << endl; // 正常输出 1

    vec.push_back(6);   // vector 扩容，capacity 增加到 10，内存重新分配

    cout << *it << endl; // 这里会出现未定义行为
}
```

### vector 的时间复杂度

| 操作 | 时间复杂度 |
| --- | --- |
| `push_back()`、`pop_back()`  | `O(1)` |
| `insert()`、`erase()`  | `O(n)` |
| `at()` | `O(1)` |

> [!NOTE]
> 
> **emplace_back() 和 push_back() 的区别**
>
> 不废话看代码：
>
> ``` c++
> class A {
> private:
>     int _count;
> public:
>     A(int count): _count(count){
>         cout << "default" << endl;
>     }
>     A(const A& a): _count(a._count) {
>         cout << "copy" << endl;
>     }
>     A(A&& a): _count(a._count) {
>         cout << "move" << endl;
>     }
> };
> 
> int main() {
>     vector<A> vec;
>     vec.reserve(100);
> 
>     // 相同点：
>     A a(1);
>     // 传入左值
>     vec.push_back(a);       //copy
>     vec.emplace_back(a);    //copy
>     A b(1);
>     A c(1);
>     // 传入右值
>     vec.push_back(std::move(b));    //move
>     vec.emplace_back(std::move(c)); //move
> 
>     // 不同点：
>     // 传入参数
>     vec.push_back(1);       //default move
>     vec.emplace_back(1);    //default
> }
> ```
> 
> **resize() 和 assign() 的区别**
> 
> - `resize()`: 改变vector的大小，如果新的大小比原来的大，则新的元素用默认值填充，如果新的大小比原来的小，则删除多余的元素
> 
> - `assign()`: 为vector赋值，会清空原来的元素，然后用新的元素填充
> 
> ![](https://s2.loli.net/2024/06/28/n5lWdFQCciyzKA9.jpg)

## array

``` c++
#include <iostream>
#include <array>
using namespace std;

// array使用示例
int main() {
    array<int, 5> arr = {10, 20, 30, 40, 50};

    // 获取array的大小
    cout << arr.size() << endl;

    // 访问指定位置的元素，复杂度为O(1)
    arr[2] = 100;
    cout << arr[2] << endl;

    // 获取array的第一个元素和最后一个元素，复杂度为O(1)
    cout << arr.front() << endl;
    cout << arr.back() << endl;

    // 直接获取array的数据指针
    int *p = arr.data();
    cout << *(p + 2) << endl;
}
```

array 常用的成员函数：

- 元素访问

    - `front()`: 返回第一个元素

    - `back()`: 返回最后一个元素

    - `at()`: 访问指定位置的元素，会检查索引是否越界

    - `data()`: 直接获取array的数据指针

- 迭代器

    - `begin()`: 返回指向第一个元素的迭代器，

    - `end()`: 返回指向最后一个元素的迭代器

    - `rbegin()`: 返回指向最后一个元素的逆向迭代器

    - `rend()`: 返回指向第一个元素的逆向迭代器

    - `cbegin()`、`cend()`、`crbegin()`、`crend()`: 返回常量迭代器，不允许修改元素

- 容量

    - `size()`: 返回array中元素的个数

    - `empty()`: 判断array是否为空

    - `max_size()`: 与 `size()` 相同，只是为了与其他容器保持一致

- 容器操作

    - `fill()`: 用指定的值填充array

    - `swap()`: 交换两个array的元素

### array 和 vector 的区别

- `array` 是 **静态数组**，大小是 **固定** 的，不支持插入和删除操作，支持随机访问，元素在 **栈上** 分配。

- `vector` 是 **动态数组**，大小是 **可变** 的，支持插入和删除操作，支持随机访问，元素在 **堆上** 分配。

``` c++
int main() {
    std::array<int, 100> arr = {1,2,3};
    cout << &arr << endl;   // 栈上：0x7fff92971320
    cout << arr.data() << endl;     //栈上：0x7fff92971320

    vector<int> vec {1, 2, 3};
    cout << &vec << endl;   // 栈上：0x7fff92971320
    cout << vec.data() << endl; // 堆上：0x7ad2c0
}
```

### array 的时间复杂度

| 操作 | 时间复杂度 |
| --- | --- |
| `at()` | `O(1)` |
| `front()`、`back()` | `O(1)` |

## deque

``` c++
#include<iostream>
#include<deque>
using namespace std;

// deque使用示例
int main() {
    deque<int> deq(10, 1); // 初始化一个大小为10，元素值为0的deque

    // 在deque的尾部和头部插入元素：复杂度为O(1)
    deq.push_back(99);
    deq.push_back(99);
    deq.push_front(100);

    // 在deque的尾部和头部删除元素：复杂度为O(1)
    deq.pop_back();
    deq.pop_front();

    // 随机插入和删除元素：复杂度为O(n)
    deq.insert(deq.begin() + 1, 3);
    deq.erase(deq.begin() + 1);

    // deque的大小
    cout << deq.size() << endl;

    //  获取deque的第一个元素和最后一个元素
    cout << deq.front() << endl;
    cout << deq.back() << endl;

    // 访问指定位置的元素
    cout << deq[5] << endl;
    cout << deq.at(5) << endl;  // at函数会检查索引是否越界，更安全
}
```

deque 常用的成员函数：

- 元素访问

    - `front()`: 返回第一个元素

    - `back()`: 返回最后一个元素

    - `at()`: 访问指定位置的元素，会检查索引是否越界

- 迭代器

    - `begin()`、`end()`、`rbegin()`、`rend()`、`cbegin()`、`cend()`、`crbegin()`、`crend()`: 与 vector 相同

- 容量

    - `size()`: 返回deque中元素的个数

    - `max_size()`: 返回当前程序中的deque最大可以容纳的元素个数，通常是一个很大的值

    - `empty()`: 判断deque是否为空

- 元素修改

    - `push_back()`: 尾部插入元素

    - `emplace_back()`: 原地构造并插入元素到尾部

    - `pop_back()`: 尾部删除元素

    - `append_range()`: 尾部插入元素一个范围内的元素（C++23）

    - `push_front()`: 头部插入元素

    - `emplace_front()`: 原地构造并插入元素到头部

    - `pop_front()`: 头部删除元素

    - `prepend_range()`: 头部插入元素一个范围内的元素（C++23）

    - `insert()`: 插入元素

    - `emplace()`: 原地构造并插入元素

    - `erase()`: 删除元素

    - `clear()`: 清空deque

    - `resize()`: 改变deque的大小

    - `swap()`: 交换两个deque的元素

### deque 的底层实现

deque 的底层实现是由多段 **等长的连续空间** 组成，各段不一定连续，它们被一块 map 数组所控制，map 数组的每个元素都是一个指针，指向一段连续空间。

由于 deque 这种特殊的结构，deque 的迭代器不是普通的指针，而是一个复杂的结构体，它包含了四个指针，分别指向当前段的当前元素、当前段头、当前段尾、当前段的 map 指针。

![](https://s2.loli.net/2024/06/28/J8usGEVgkah5ifT.jpg)

### deque 的时间复杂度

| 操作 | 时间复杂度 |
| --- | --- |
| `push_back()`、`pop_back()`、`push_front()`、`pop_front()` | `O(1)` |
| `insert()`、`erase()` | `O(n)` |
| `at()` | `O(1)` |

## list

``` c++
#include <iostream>
#include <list>
using namespace std;

// list使用示例
int main() {
    list<int> ls = {1, 2, 3, 4, 5};

    // 在list的头部和尾部插入元素：复杂度为O(1)
    ls.push_back(6);
    ls.push_front(0);

    // 在list的头部和尾部删除元素：复杂度为O(1)
    ls.pop_back();
    ls.pop_front();

    // 随机插入和删除元素：复杂度为O(1)
    ls.insert(ls.begin(), 3);
    ls.erase(ls.begin());

    // list的大小
    cout << ls.size() << endl;

    // 获取list的第一个元素和最后一个元素
    cout << ls.front() << endl;
    cout << ls.back() << endl;

    // 访问指定位置的元素，使用迭代器
    list<int>::iterator it = ls.begin();
    advance(it, 2); // advance函数用于移动迭代器
    cout << *it << endl;
}
```

list 常用的成员函数：

- 元素访问

    - `front()`: 返回第一个元素

    - `back()`: 返回最后一个元素

    - list 不支持随机访问，因此没有 `at()` 函数，如果需要访问指定位置的元素，需要使用迭代器

- 迭代器

    - `begin()`、`end()`、`rbegin()`、`rend()`、`cbegin()`、`cend()`、`crbegin()`、`crend()`: 与 vector 相同

- 容量

    - `size()`: 返回list中元素的个数

    - `max_size()`: 返回当前程序中的list最大可以容纳的元素个数，通常是一个很大的值

    - `empty()`: 判断list是否为空

- 元素修改

    - `push_back()`: 尾部插入元素

    - `emplace_back()`: 原地构造并插入元素到尾部

    - `pop_back()`: 尾部删除元素

    - `push_front()`: 头部插入元素

    - `emplace_front()`: 原地构造并插入元素到头部

    - `pop_front()`: 头部删除元素

    - `insert()`: 插入元素

    - `emplace()`: 原地构造并插入元素

    - `erase()`: 删除元素

    - `clear()`: 清空list

    - `resize()`: 改变list的大小

    - `swap()`: 交换两个list的元素

- 容器操作

    - `splice()`: 将一个 list 中的元素插入到另一个 list 中

    - `merge()`: 合并两个有序 list

    - `remove()`: 删除 list 中值为指定值的元素，不是按照迭代器删除

    - `remove_if()`: 删除 list 中满足条件的元素

    - `reverse()`: 反转 list

    - `unique()`: 删除 list 中相邻的重复元素

    - `sort()`: 对 list 进行排序

### list 的时间复杂度

list 是一个插入删除效率高的容器，但是访问效率低；vector 是一个访问效率高的容器，但是插入删除效率低；

| 操作 | 时间复杂度 |
| --- | --- |
| `push_back()`、`pop_back()`、`push_front()`、`pop_front()` | `O(1)` |
| `insert()`、`erase()` | `O(1)` |
| `front()`、`back()` | `O(1)` |

## forward_list

forward_list 是一个单向链表，它只支持单向遍历，不支持双向遍历，因此没有 `back()` 函数。

``` c++
#include <iostream>
#include <forward_list>
using namespace std;

// forward_list使用示例
int main() {
    forward_list<int> fl = {1, 2, 3, 4, 5};

    // 在forward_list的头部插入元素：复杂度为O(1)
    fl.push_front(0);

    // 在forward_list的头部删除元素：复杂度为O(1)
    fl.pop_front();

    // 随机插入和删除元素：复杂度为O(1)
    fl.insert_after(fl.before_begin(), 3);
    fl.erase_after(fl.before_begin());

    // forward_list的大小
    cout << fl.size() << endl;

    // 获取forward_list的第一个元素
    cout << fl.front() << endl;

    // 访问指定位置的元素，使用迭代器
    forward_list<int>::iterator it = fl.begin();
    advance(it, 2); // advance函数用于移动迭代器
    cout << *it << endl;
}
```

forward_list 常用的成员函数：

- 元素访问

    - `front()`: 返回第一个元素

- 迭代器

    - `begin()`: 返回指向第一个元素的迭代器

    - `end()`: 返回指向最后一个元素的迭代器

    - `before_begin()`: 返回指向第一个元素之前的迭代器，这是一个特殊的迭代器，用于插入到第一个元素的位置

    - `cbegin()`、`cend()`、`cbefore_begin()`: 返回常量迭代器，不允许修改元素

    - forward_list 不支持反向迭代器，因此没有 `rbegin()`、`rend()`、`crbegin()`、`crend()` 函数

- 容量

    - `empty()`: 判断forward_list是否为空

    - `max_size()`: 返回当前程序中的forward_list最大可以容纳的元素个数，通常是一个很大的值

- 元素修改

    - `push_front()`、`emplace_front()`: 头部插入元素

    - `pop_front()`: 头部删除元素

    - `insert_after()`、`emplace_after()`: 插入元素

    - `erase_after()`: 删除元素

    - `clear()`: 清空forward_list

    - `resize()`: 改变forward_list的大小

    - `insert_range_after()`: 插入元素一个范围内的元素（C++23）

    - `prepend_range()`: 头部插入元素一个范围内的元素（C++23）

    - `swap()`: 交换两个forward_list的元素

- 容器操作

    - `splice_after()`: 将一个 forward_list 中的元素插入到另一个 forward_list 中

    - `merge()`: 合并两个有序 forward_list

    - `remove()`、`remove_if()`: 删除 forward_list 中满足条件的元素

    - `reverse()`: 反转 forward_list

    - `unique()`: 删除 forward_list 中相邻的重复元素

    - `sort()`: 对 forward_list 进行排序

### forward_list 的时间复杂度

forward_list 是一个插入删除效率高的容器，但是访问效率低，但相比于 list，forward_list 的空间效率更高，性能开销更小。

| 操作 | 时间复杂度 |
| --- | --- |
| `push_front()`、`pop_front()` | `O(1)` |
| `insert_after()`、`erase_after()` | `O(1)` |
| `front()` | `O(1)` |

## set

``` c++
#include <iostream>
#include <set>
using namespace std;

// set使用示例
int main() {
    set<int> s = {1, 2, 3, 4, 5};

    // 插入元素：复杂度为O(logn)
    s.insert(6);

    // 删除元素：复杂度为O(logn)
    s.erase(3);

    // 查找元素：复杂度为O(logn)
    auto it = s.find(4);
    if (it != s.end()) {
        cout << *it << endl;
    }

    // set的大小
    cout << s.size() << endl;

    // 判断set是否为空
    cout << s.empty() << endl;
}
```

set 常用的成员函数：

- 迭代器

    - `begin()`、`end()`、`rbegin()`、`rend()`、`cbegin()`、`cend()`、`crbegin()`、`crend()`: 该有的都有

- 容量

    - `size()`、`max_size()`、`empty()`: 该有的都有

- 元素修改

    - `insert()`、`emplace()`、`insert_range()`: 插入元素

    - `emplace_hint()`: 通过迭代器提示插入元素，可以提高插入效率

    - `erase()`: 删除元素

    - `clear()`: 清空set

    - `swap()`: 交换两个set的元素

    - `extract()`: 提取并删除元素（C++17）
    
    - `merge()`: 合并两个有序 set（C++17）

- 查找

    - `find()`: 查找元素

    - `count()`: 统计元素个数

    - `lower_bound()`: 返回第一个不小于指定值的元素的迭代器

    - `upper_bound()`: 返回第一个大于指定值的元素的迭代器

    - `equal_range()`: 返回指定值的区间

    - `contains()`: 判断是否包含指定值（C++20）

### set、map、multi_set、multi_map 的特点

- `set`：有序集合，不允许重复元素，插入元素时会自动排序

- `map`：有序映射，不允许重复键，插入元素时根据键自动排序

- `multi_set`：有序多重集合，允许重复元素，插入元素时会自动排序

- `multi_map`：有序多重映射，允许重复键，插入元素时根据键自动排序

### set 的底层实现

set、map、multi_set、multi_map 的底层实现是 **红黑树**，它是一种自平衡的二叉查找树，可以保证插入、删除、查找的时间复杂度都是 `O(logn)`。至于为什么不选择 **AVL 树**，是因为红黑树的旋转操作比 AVL 树的旋转操作更少，AVL 树的平衡策略太严格了。综合来看，红黑树是一种性能和平衡性都比较好的二叉查找树。

### multiset 的底层实现

multiset 的底层实现也是红黑树，但是 multiset 允许重复元素，因此 

### set 的时间复杂度

| 操作 | 时间复杂度 |
| --- | --- |
| `insert()`、`erase()` | `O(logn)` |
| `find()` | `O(logn)` |

## unordered_set

使用方法和 set 类似，只是 unordered_set 是无序集合，底层实现是 **哈希表**，插入、删除、查找的时间复杂度都是 `O(1)`。是一种空间换时间的策略。

unordered_set 常用的成员函数：

- 迭代器

    - `begin()`、`end()`、`cbegin()`、`cend()`: unordered_set 不支持反向迭代器

- 容量

    - `size()`、`max_size()`、`empty()`: 该有的都有

- 元素修改

    - 和 set 一致

- 查找

    - 和 set 一致，但是 unordered_set 不支持 `lower_bound()`、`upper_bound()`、`equal_range()` 函数

### unordered_set 的底层实现

unordered_set、unordered_map、unordered_multiset、unordered_multimap 的底层实现是 **哈希表**，哈希表是一种以键值对形式存储数据的数据结构，通过哈希函数将键映射到哈希表中的一个位置，然后将值存储在该位置。

哈希表的优点是插入、删除、查找的时间复杂度都是 `O(1)`，但是哈希表的缺点是空间利用率低，哈希冲突的处理比较复杂。

### unordered_set 的时间复杂度

| 操作 | 时间复杂度 |
| --- | --- |
| `insert()`、`erase()` | `O(1)` |
| `find()` | `O(1)` |

## map 

不想写了

> [!NOTE]
>
> **map中[]和find的区别**
>
> - `[]`：返回的是该 key 对应的 value，如果 key 不存在，会插入一个键值对，值为默认值
>
> - `find()`：返回的是该 key 的迭代器，如果 key 不存在，返回 `end()` 迭代器

## unordered_map

不想写了

## stack

stack 和 queue 是容器适配器，它们是在底层容器的基础上提供了一些操作，使得底层容器的操作更加方便。

``` c++
#include <iostream>
#include <stack>
using namespace std;

// stack使用示例
int main() {
    stack<int> s;

    // 入栈
    s.push(1);
    s.push(2);
    s.push(3);

    // 出栈
    s.pop();

    // 获取栈顶元素
    cout << s.top() << endl;

    // 判断栈是否为空
    cout << s.empty() << endl;

    // 获取栈的大小
    cout << s.size() << endl;
}
```

stack 常用的成员函数：

- 元素访问

    - `top()`: 返回栈顶元素

- 容量

    - `size()`: 返回栈中元素的个数

    - `empty()`: 判断栈是否为空

- 元素修改

    - `push()`: 入栈

    - `emplace()`: 原地构造并入栈

    - `pop()`: 出栈

    - `swap()`: 交换两个 stack 的元素

    - `emplace()`: 原地构造并入栈

### stack 的底层实现

stack 的底层实现默认是由 deque 实现的，deque 是一个双端队列，支持在队头和队尾插入和删除元素，因此 stack 也支持在栈顶插入和删除元素。

### stack 的时间复杂度

| 操作 | 时间复杂度 |
| --- | --- |
| `push()`、`pop()`、`top()` | `O(1)` |

## queue

``` c++
#include <iostream>
#include <queue>
using namespace std;

// queue使用示例
int main() {
    queue<int> q;

    // 入队
    q.push(1);
    q.push(2);
    q.push(3);

    // 出队
    q.pop();

    // 获取队头元素
    cout << q.front() << endl;

    // 获取队尾元素
    cout << q.back() << endl;

    // 判断队列是否为空
    cout << q.empty() << endl;

    // 获取队列的大小
    cout << q.size() << endl;
}
```

queue 常用的成员函数：

- 元素访问

    - `front()`: 返回队头元素

    - `back()`: 返回队尾元素

- 容量

    - `size()`: 返回队列中元素的个数

    - `empty()`: 判断队列是否为空

- 元素修改

    - `push()`: 入队

    - `emplace()`: 原地构造并入队

    - `pop()`: 出队

    - `swap()`: 交换两个 queue 的元素

## 迭代器

STL 常用迭代器操作函数：

- `advance()`：移动迭代器

- `distance()`：计算两个迭代器之间的距离

- `next()`：返回迭代器的下一个或下 n 个位置

- `prev()`：返回迭代器的上一个或上 n 个位置

- `begin()`：返回指向第一个元素的迭代器

- `end()`：返回指向最后一个元素的迭代器

- `rbegin()`：返回指向最后一个元素的逆向迭代器

- `rend()`：返回指向第一个元素的逆向迭代器

- `cbegin()`、`cend()`、`crbegin()`、`crend()`：返回常量迭代器，不允许修改元素

### 迭代器底层实现

迭代器实际上是一种泛化的指针，它是一个类，重载了 `*`、`->`、`++`、`--`、`==`、`!=` 等操作符，使得迭代器可以像指针一样操作容器中的元素。

### 迭代器和反向迭代器的关系

![](https://s2.loli.net/2024/06/28/RBn7Sa8QqVKpYrh.jpg)
