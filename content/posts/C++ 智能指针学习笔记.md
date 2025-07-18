# C++ 智能指针学习笔记

## 智能指针简介

智能指针是一种 RAII（Resource Acquisition Is Initialization）技术，用于管理动态分配的内存。智能指针的优点是可以自动释放内存，避免内存泄漏。

C++11 标准引入了三种智能指针：`std::unique_ptr`、`std::shared_ptr` 和 `std::weak_ptr`。

它们都定义在头文件 `<memory>` 中。

## unique_ptr

`unique_ptr` 是一种独占所有权的智能指针，它保证同一时间只有一个指针可以指向对象。

### unique_ptr 的创建

```cpp
#include <iostream>
#include <memory>

int main() {
    // 使用 new 创建 unique_ptr
    std::unique_ptr<int> up1(new int(10));
    std::cout << *up1 << std::endl;

    // 使用裸指针创建 unique_ptr
    int count = 20;
    std::unique_ptr<int> up2(&count);
    std::cout << *up2 << std::endl;

    // 使用 make_unique 创建 unique_ptr
    auto up2 = std::make_unique<int>(20);
    std::cout << *up2 << std::endl;

    return 0;
}
```

### unique_ptr 的拷贝和赋值

`unique_ptr` 不能拷贝，但可以移动。

```cpp
#include <iostream>

int main() {
    std::unique_ptr<int> up1(new int(10));
    std::unique_ptr<int> up2 = std::move(up1);

    std::cout << *up2 << std::endl;

    return 0;
}
```

### unique_ptr 的释放

`unique_ptr` 会在离开作用域时自动释放内存。

```cpp
#include <iostream>

int main() {
    std::unique_ptr<int> up1(new int(10));
    std::cout << *up1 << std::endl;

    return 0;
}
```

### unique_ptr 的自定义删除器

`unique_ptr` 支持自定义删除器，可以用于释放动态分配的内存。

```cpp
#include <iostream>
#include <memory>

void deleter(int *p) {
    std::cout << "delete " << *p << std::endl;
    delete p;
}

int main() {
    std::unique_ptr<int, decltype(deleter) *> up(new int(10), deleter);
    std::cout << *up << std::endl;

    return 0;
}
```

### unique_ptr 作为函数参数

当 `unique_ptr` 作为函数参数时，分为两种情况：

1. Pass by value（传值）：需要使用 `std::move` 移动 `unique_ptr`，调用函数后原来的 `unique_ptr` 就会失效。

2. Pass by reference（传引用）：`unique_ptr` 不会被移动，调用函数后原来的 `unique_ptr` 仍然有效。如果声明为 `const` 引用，则不能改变 `unique_ptr` 的指向（如调用 `reset` 函数）。

```cpp
#include <iostream>
#include <memory>

void foo(std::unique_ptr<int> up) {
    std::cout << *up << std::endl;
}

void bar(std::unique_ptr<int> &up) {
    std::cout << *up << std::endl;
}

void baz(const std::unique_ptr<int> &up) {
    std::cout << *up << std::endl;
}

int main() {
    auto up = std::make_unique<int>(10);
    foo(std::move(up));  // up 失效
    // std::cout << *up << std::endl;  // 运行时错误

    auto up2 = std::make_unique<int>(20);
    bar(up2);  // up2 仍然有效
    std::cout << *up2 << std::endl;

    auto up3 = std::make_unique<int>(30);
    baz(up3);  // up3 仍然有效
    std::cout << *up3 << std::endl;

    return 0;
}
```

### unique_ptr 作为函数返回值

`unique_ptr` 作为函数返回值时，只有一种情况：

1. Return by value（返回值）：需要使用 `std::move` 移动 `unique_ptr`，调用函数后原来的 `unique_ptr` 就会失效。

```cpp
#include <iostream>
#include <memory>

std::unique_ptr<int> foo() {
    return std::make_unique<int>(10);
}

int main() {
    auto up = foo();
    std::cout << *up << std::endl;

    return 0;
}
```

## shared_ptr

`shared_ptr` 是一种共享所有权的智能指针，它可以被多个指针共享，当最后一个指针离开作用域时，会自动释放内存。

### shared_ptr 的创建

```cpp
#include <iostream>
#include <memory>

int main() {
    // 使用 new 创建 shared_ptr
    std::shared_ptr<int> sp1(new int(10));
    std::cout << *sp1 << std::endl;

    // 使用裸指针创建 shared_ptr
    int count = 20;
    std::shared_ptr<int> sp2(&count);
    std::cout << *sp2 << std::endl;

    // 使用 make_shared 创建 shared_ptr
    auto sp3 = std::make_shared<int>(30);
    std::cout << *sp3 << std::endl;

    // 拷贝 shared_ptr
    std::shared_ptr<int> sp4 = sp3;
    std::cout << *sp4 << std::endl;

    return 0;
}
```

### shared_ptr 的释放

`shared_ptr` 会在最后一个指针离开作用域时自动释放内存。

```cpp
#include <iostream>
#include <memory>

int main() {
    std::shared_ptr<int> sp1(new int(10));
    std::cout << *sp1 << std::endl;

    {
        std::shared_ptr<int> sp2 = sp1;
        std::cout << *sp2 << std::endl;
    }   // sp2 离开作用域，内存不会被释放

    std::cout << *sp1 << std::endl; // sp1 仍然有效

    return 0;
    // sp1 离开作用域，内存被释放
}
```

### shared_ptr 的自定义删除器

`shared_ptr` 支持自定义删除器，可以用于释放动态分配的内存。

```cpp
#include <iostream>
#include <memory>

void deleter(int *p) {
    std::cout << "delete " << *p << std::endl;
    delete p;
}

int main() {
    std::shared_ptr<int> sp(new int(10), deleter);
    std::cout << *sp << std::endl;

    return 0;
}
```

### shared_ptr 作为函数参数

当 `shared_ptr` 作为函数参数时，分为两种情况：

1. Pass by value（传值）：可以直接传递 `shared_ptr`，函数内部会自动增加引用计数，调用函数后原来的 `shared_ptr` 仍然有效。

2. Pass by reference（传引用）：与 `unique_ptr` 类似，如果声明为 `const` 引用，则不能改变 `shared_ptr` 的指向（如调用 `reset` 函数）。

```cpp
#include <iostream>
#include <memory>

void foo(std::shared_ptr<int> sp) {
    std::cout << *sp << std::endl;
    std::cout << sp.use_count() << std::endl;
}

void bar(const std::shared_ptr<int> &sp) {
    std::cout << *sp << std::endl;
    std::cout << sp.use_count() << std::endl;
}

int main() {
    auto sp = std::make_shared<int>(10);
    foo(sp);  // sp 仍然有效
    std::cout << sp.use_count() << std::endl;

    auto sp2 = std::make_shared<int>(20);
    bar(sp2);  // sp2 仍然有效
    std::cout << sp2.use_count() << std::endl;

    return 0;
}
```

### shared_ptr 作为函数返回值

与 `unique_ptr` 类似同理。

## unique_ptr 和 shared_ptr 的转换

`unique_ptr` 能够转换为 `shared_ptr`，但 `shared_ptr` 不能转换为 `unique_ptr`。

```cpp
#include <iostream>
#include <memory>

int main() {
    std::unique_ptr<int> up(new int(10));
    std::shared_ptr<int> sp = std::move(up);

    std::cout << *sp << std::endl;
    return 0;
}
```

## weak_ptr

`weak_ptr` 是一种弱引用的智能指针，它不会增加引用计数，当最后一个指向对象的 `shared_ptr` 离开作用域时，即使还有 `weak_ptr` 指向对象，对象也会被释放。

`weak_ptr` 用于解决 `shared_ptr` 的循环引用问题。

```cpp
#include <iostream>
#include <memory>

int main() {
    std::shared_ptr<int> sp(new int(10));
    std::weak_ptr<int> wp = sp;

    std::cout << *wp.lock() << std::endl; // lock() 返回 shared_ptr

    sp.reset();
    if (wp.expired()) {
        std::cout << "wp is expired" << std::endl;
    }

    return 0;
}
```

## 智能指针和管理的对象分别在哪个区域

智能指针和管理的对象并在同一个区域，C++ 中的智能指针是在栈上分配的，而管理的对象是在堆上分配的。之所以能够自动释放内存，是因为智能指针在析构时会调用 `delete` 函数。

记住，智能指针本身是在 **栈上** 分配的，而其管理的对象是在 **堆上** 分配的。

## 参考

- [C++现代实用教程:智能指针](https://www.bilibili.com/video/BV18B4y187uL)（视频）
