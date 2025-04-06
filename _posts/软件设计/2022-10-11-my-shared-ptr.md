---
title: 自己实现一个简易版共享指针
category: [软件设计]
tags: [shared_ptr, 智能指针]
---

> C++ 中的共享指针通过引用计数机制实现自动内存管理，下面是一个简约版的共享指针实现方式，用于学习和理解共享指针的底层实现。
{: .prompt-info }


```c++
#include <iostream>

template <typename T>
class SharedPtr {
private:
    T* m_ptr;
    unsigned* m_count;
    void release() {
        if (--(*m_count) == 0) {
            delete m_ptr;
            delete m_count;
        }
    }
    
public:
    explicit SharedPtr(T* ptr = nullptr)
        : m_ptr(ptr)
        , m_count(new unsigned(1)) {
        }
    SharedPtr(const SharedPtr& other)
        : m_ptr(other.m_ptr)
        , m_count(other.m_count) {
        ++(*m_count);
    }
    SharedPtr& operator=(const SharedPtr& other) {
        if (this != &other) {
            release();
            m_ptr = other.m_ptr;
            m_count = other.m_count;
            ++(*m_count);
        }
        return *this;
    }
    T& operator*() const {
        return *m_ptr;
    }
    T* operator->() const {
        return m_ptr;
    }
    unsigned use_count() const {
        return *m_count;
    }
    ~SharedPtr() {
        release();
    }
};

// 使用示例
struct MyRes {
    MyRes() {
        std::cout << "Resource created\n";
    }
    ~MyRes() {
        std::cout << "Resource destroyed\n";
    }
};

int main() {
    SharedPtr<MyRes> p1(new MyRes());
    {
        SharedPtr<MyRes> p2 = p1;
        std::cout << "Use count: " << p2.use_count() << std::endl; // 2
    }
    std::cout << "Use count: " << p1.use_count() << std::endl; // 1
}
```