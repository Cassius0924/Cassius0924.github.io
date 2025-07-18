---
title: 'C++ delete ptr 和 ptr = nullptr 的区别'
draft: false
date: 2024-06-29T14:42:02+08:00
description: ''
author: 'Cassius0924'
tags: ["C++", "Pointer", "MemoryManagement", "Nullptr", "Delete"]
---

# C++ delete ptr 和 ptr = nullptr 的区别

## delete ptr 

`delete ptr` 是释放 `ptr` 所指向的对象资源，而 `ptr` 依然存在，且依然指向那片内存地址。

## ptr = nullptr

`ptr = nullptr` 是将 `ptr` 指向空指针，和其所指向的对象没关系。

![ptr](https://s2.loli.net/2024/06/29/z8RCgt6krxTPZjU.jpg)

## 试着实现一个 unique_ptr

``` c++
template <typename T>
class UniquePtr {
private:
    T *_ptr;

public:
    // 默认构造
    UniquePtr() : _ptr(nullptr) { }
    explicit UniquePtr(T *ptr) : _ptr(ptr) { }

    ~UniquePtr() {
        delete _ptr;
        // 无需置 nullptr，因为析构函数会被调用，_ptr 会被销毁
        // 置空无意义
    }

    // 拷贝构造 删除
    UniquePtr(const UniquePtr &) = delete;
    UniquePtr &operator=(const UniquePtr &) = delete;

    // 移动构造
    UniquePtr(UniquePtr &&p) noexcept : _ptr(p._ptr) {
        // 至于这里为什么不需要 delete _ptr
        // 是因为这是移动构造函数，是个构造函数！_ptr 本来就没有资源
        p._ptr = nullptr;
    }
    UniquePtr &operator=(UniquePtr &&p) noexcept {
        if (p != *this) {
            delete _ptr;        // 第一步，释放当前资源
            _ptr = p._ptr;      // 第二步，将当前指针指向新的资源
            p._ptr = nullptr;   // 第三步，将原来的指针置空
        }
        return *this;
    }

    T *get() const { // 返回指针
        return _ptr;
    }
    T *operator->() const { // 返回指针
        return _ptr;
    }
    T &operator*() const { // 解引用
        return *_ptr;
    }

    T *release() {
        // 这里不能 delete _ptr
        // 因为 release 只是解除 UniquePtr 对资源的所有权，但资源还是存在的
        T *tmp = _ptr;
        _ptr = nullptr;
        return tmp;
    }
    
    void reset(T *newptr = nullptr) {
        if (_ptr != newptr) {
            delete _ptr;        // 释放当前资源
            _ptr = newptr;      // 指向新资源
            // 这里不需要置空 newptr
            // 是否置空 new ptr 由用户决定
        }
    }
};
```

`UniquePtr &operator=(UniquePtr &&p)` 移动赋值运算符的原理如下图：

![](https://s2.loli.net/2024/06/29/SIOfTD1RgjmUXVJ.png)
