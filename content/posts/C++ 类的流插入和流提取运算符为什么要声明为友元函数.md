---
title: 'C++ 类的流插入和流提取运算符为什么要声明为友元函数'
draft: false
date: 2024-06-25T21:35:40+08:00
description: ''
author: 'Cassius0924'
tags: ["C++", "OOP", "Friend Function", "Operator Overloading"]
---

# C++ 类的流插入和流提取运算符为什么要声明为友元函数

## 友元函数版代码

``` c++
class AClass {
private:
    int _count;
    std::string _str;
    std::vector<int> _vec;

public:
    friend std::ostream &operator<<(std::ostream &os, const AClass &a) {
        os << "AClass: count = " << a._count << ", str = " << a._str << ", vec size = " << a._vec.size();
        return os;
    }
};
```

## 为什么要声明为友元函数

先理解一下友元函数，它实际上是一个普通函数，不属于类成员，但它又是一个特殊的普通函数，因为它可以访问类的私有成员。因此 `operator<<` 和 `operator>>` 声明为友元函数的目的很明显，就是为了能够访问类的私有成员。

实际上，如果它们不声明为友元函数，也是可以实现的，例如下面代码：

``` c++
class AClass {
private:
    int _count;
    std::string _str;
    std::vector<int> _vec;

public:
    std::ostream &operator<<(std::ostream &os) {
        os << "AClass: count = " << _count << ", str = " << _str << ", vec size = " << _vec.size();
        return os;
    }
};
```

但是这样就需要特殊的方法来调用这个 `operator<<` 函数，因为它不再是一个普通函数，而是一个类成员函数：

``` c++
int main() {
    AClass a;
    a << std::cout;    // 错误，不能这样调用
    a.operator<<(std::cout);    // 正确
    return 0;
}
```

这样显然不够直观，不是一个正常人类写的代码：）

所以，为了代码的可读性和可维护性，我们将 `operator<<` 和 `operator>>` 声明为友元函数，这样就可以直接使用 `<<` 和 `>>` 运算符来操作类的对象了。
