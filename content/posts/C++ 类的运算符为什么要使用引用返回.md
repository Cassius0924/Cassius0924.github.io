---
title: 'C++ 类的运算符为什么要使用引用返回'
draft: false
date: 2024-06-25T22:41:04+08:00
description: ''
author: 'Cassius0924'
tags: ["C++", "Operator Overloading", "Reference", "Best Practice"]
---

# C++ 类的运算符为什么要使用引用返回

## 代码

``` c++
class AClass {
private:
    int _count;

public:
    AClass() : _count(0) {
        std::cout << "Default constructor called\n";
    }

    // 赋值运算符，返回引用
    AClass &operator=(int cnt) {
        _count = cnt;k
        return *this;
    }

    // 后置自增运算符，返回引用
    int &operator++(int) {
        ++_count;
        return *this;
    }
};
```

如果不使用引用返回，其实也是可以运行的，只不过会在返回时调用拷贝构造函数，生成临时对象，然后再调用析构函数释放临时对象，这样会多出一次拷贝构造和析构的开销。而使用引用返回，可以直接返回对象的引用，避免了这个开销。

需要注意的是，如果我们返回值类型，我们是不能直接修改返回值的：

``` c++
class AClass {
private:
    int _count;

public:
    // 省略构造函数
    // 赋值运算符，返回引用
    AClass operator=(int cnt) {
        _count = cnt;
        return *this;
    }

    // 后置自增运算符，返回引用
    int operator++(int) {
        ++_count;
        return *this;
    }

    void print() const {
        std::cout << "AClass: count = " << _count << '\n';
    }
};
int main() {
    AClass a;
    a.print();  // 输出 AClass: count = 0
    (a++) = 10;
    a.print();  // 输出 AClass: count = 1
    (a++)++;
    a.print();  // 输出 AClass: count = 2
}
```

可以看到，如果我们返回值类型，我们是不能直接修改返回值的。虽然 `a++` 已经修改了 `a` 的值，但是 `a++` 返回的是一个修改后的 `a` 对象的拷贝，所以 `(a++) = 10;` 或者 `(a++)++;` 修改的是这个拷贝对象，而不是原对象 `a`。
