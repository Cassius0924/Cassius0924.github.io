# C++ Copy&Swap 惯用法指南

## Copy&Swap 是什么

Copy&Swap 是一种 C++ 中常用的编程技巧，用于实现类的赋值运算符（`operator=`）。

## 实现

### 传统写法

先看看未使用 Copy&Swap 的赋值运算符写法：

``` c++
#include <iostream>
#include <vector>

class OldAClass {
private:
    int _count;
    std::string _str;
    std::vector<int> _vec;

public:
    OldAClass() : _count(0), _vec(10) {}

    // 拷贝构造函数 和 拷贝赋值运算符
    OldAClass(OldAClass &a) : _count(a._count), _str(a._str), _vec(a._vec) {
        std::cout << "Copy constructor called\n";
    }

    OldAClass &operator=(OldAClass &a) {
        std::cout << "Copy Assignment operator called\n";
        if (this != &a) { //判断传入的 a 是否是自己
            _count = a._count;
            _str = a._str;
            _vec = a._vec;
        }
        return *this;
    }

    // 移动构造函数 和 移动赋值运算符
    OldAClass(OldAClass &&a) noexcept : _count(a._count), _str(std::move(a._str)), _vec(std::move(a._vec)) {
        std::cout << "Move constructor called\n";
    }

    OldAClass &operator=(OldAClass &&a) noexcept {
        std::cout << "Move Assignment operator called\n";
        if (this != &a) {
            _count = a._count;
            _str = std::move(a._str);
            _vec = std::move(a._vec);
        }
        return *this;
    }
};

```


可以看到，这种写法需要重复写两次赋值运算符，并且每次都需要判断传入的参数是否是自己，而且代码重复度高。

### Copy&Swap 写法

``` c++
class AClass {
private:
    int _count;
    std::string _str;
    std::vector<int> _vec;

public:
    AClass() : _count(0), _vec(10) {}

    static void swap(AClass &a, AClass &b) {
        std::swap(a._count, b._count);
        std::swap(a._str, b._str);
        std::swap(a._vec, b._vec);
    }

    // 拷贝构造函数
    AClass(AClass &a) : _count(a._count), _str(a._str), _vec(a._vec) {
        std::cout << "Copy constructor called\n";
    }

    // 移动构造函数
    AClass(AClass &&a) noexcept {
        std::cout << "Move constructor called\n";
        swap(*this, a);
    }

    // 赋值运算符
    AClass &operator=(AClass a) { // 注意这里的参数是值传递，会调用拷贝构造函数
        std::cout << "Assignment operator called\n";
        swap(*this, a);
        return *this;
    }
};
```

这种写法只需要写一次赋值运算符，代码更简洁，而且不需要判断传入的参数是否是自己。

至于为什么要这样写，我们先看看拷贝赋值运算符和移动赋值运算符的本质，他们都是为了将自己的值改变为另一个对象的值。区别只在于是否保留原对象的值。

- 拷贝赋值运算符（copy）：修改自己的值为另一个对象的值，但是保留原对象的值。
- 移动赋值运算符（move）：修改自己的值为另一个对象的值，不保留原对象的值，或者不关心原对象的值。

对于移动赋值运算符，由于传进来的是一个右值引用，也就是一个将亡值，所以我们可以直接使用 `swap` 函数，交换他们的值。

那么对于拷贝赋值运算符是否也可以用 `swap` 函数呢？如果直接修改函数实现当然是不可以的，因为拷贝赋值运算符传入的是一个引用类型的参数，如果直接交换，那么会导致原对象被修改。所以我们可以将参数改为值传递，先调用拷贝构造器生成一个临时对象，然后再调用 `swap` 函数，这样就可以实现拷贝赋值运算符的功能。

## 更好的 `swap` 函数定义

``` c++
class BetterAClass {
private:
    int _count;
    std::string _str;
    std::vector<int> _vec;

public:
    BetterAClass() : _count(0), _vec(10) {
        std::cout << "Default constructor called\n";
        _str = "Hello";
        _vec.assign(10, 1);
    }

    friend void swap(BetterAClass &a, BetterAClass &b) {    //定义成友元是为了访问私有成员
        using std::swap;    //开启 ADL （Argument-Dependent Lookup 参数依赖查找）
        swap(a._count, b._count);
        swap(a._str, b._str);
        swap(a._vec, b._vec);
    }

    BetterAClass(BetterAClass &a) : _count(a._count), _str(a._str), _vec(a._vec) {
        std::cout << "Copy constructor called\n";
    }

    BetterAClass(BetterAClass &&a) noexcept {
        using std::swap;
        std::cout << "Move constructor called\n";
        swap(*this, a);
    }

    BetterAClass &operator=(BetterAClass a) {
        using std::swap;
        std::cout << "Assignment operator called\n";
        swap(*this, a);
        return *this;
    }

    friend std::ostream &operator<<(std::ostream &os, const BetterAClass &a) {
        os << "AClass: count = " << a._count << ", str = " << a._str << ", vec size = " << a._vec.size();
        return os;
    }
};

class TestClass {
public:
    AClass a;
    BetterAClass ba;

    TestClass() = default;

    friend void swap(TestClass &a, TestClass &b) {
        AClass::swap(a.a, b.a);
        using std::swap;    // 开启 ADL，保证 std::swap 被调用
        swap(a.ba, b.ba);
    }
};
```

如果我们的类需要被其他类作为成员变量，那么我们的 `swap` 函数就需要调用这些类的 `swap` 函数，如果按照原来的写法调用静态成员函数，如果不想多写 `AClass::`，那么就需要将 `swap` 函数定义为普通函数，但是普通函数并不能访问类的私有成员，所以我们需要将 `swap` 函数定义为友元函数。

并且这样做有一个额外的好处，我们可以开启 ADL（Argument-Dependent Lookup 参数依赖查找），这样我们就可以直接调用 `swap` 函数，无论传入的是什么类型，都会调用对应的最匹配的 `swap` 函数。如果找不到对应的 `swap` 函数，那么就会调用 `std::swap` 函数。

## 参考

- [[C++] 经典的 Copy & Swap](https://www.bilibili.com/video/BV1WU4y1w7Vq)（视频）

- [[C++] Copy & Swap 续：更好的 swap 定义](https://www.bilibili.com/video/BV1qv411g7YS)（视频）