---
title: 设计模式之单例模式
category: [软件设计]
tags: [设计模式, 单例模式]
---

> 单例模式是 23 种设计模式中的一种，用于确保一个类只有一个实例，并提供一个全局访问点。在 c++ 中，可以通过使用静态成员变量和静态成员函数来实现单例模式。通过将构造函数设置为私有，限制了类的实例化，然后通过静态成员函数返回类的唯一实例，确保在整个程序中只有一个实例被创建和访问。这种模式在需要全局共享的资源、配置信息或对象管理等场景下非常有用，比如 windows 系统的串口管理器就是采用的单例模式来实现的。单例模式分为懒汉式和饿汉式, 本人一般情况懒汉式单例用得多一些，下面逐一介绍。
{: .prompt-info } 

## 懒汉式单例
单例在第一次获取实例的时候才实例化。
```c++
#include <string>

template<class T>
class Singleton {
protected:
    Singleton() = default;
    ~Singleton() = default;

public:

    Singleton(const Singleton &) = delete;
    Singleton &operator=(const Singleton &) = delete;

    template<typename... Args>
    static T &instance(Args &&...args) {
        static T Singleton(std::forward<Args>(args)...);
        return instance;
    }
};

```
用法如下：
```c++
#include <iostream>

class MyClass {
public:
    void say_hello() {
        std::cout << "Hello" << std::endl;
    }
}

int main() {
    auto &my_class = Singleton<MyClass>::instance();
    my_class.say_hello();

    // 也可以使用 MyClass 直接继承 Singleton 来使用
    // auto my_class = MyClass::instance();
    // my_class.say_hello();
}
```

## 饿汉式单例
单例在进入 main 函数之前就已经实例化了。

```c++
#include <iostream>
#include <memory>
#include <mutex>
#include <string>

template<class T>
class Singleton {
protected:
    static std::unique_ptr<T> m_instance;
    static std::once_flag m_flag; // 保证多线程调用时只make_unique一次

protected:
    Singleton() = default;
    ~Singleton() = default;

public:
    template<typename... Args>
    static std::unique_ptr<T> &instance(Args &&...args) {
        std::call_once(m_flag, [&]() {
            m_instance = std::make_unique<T>(std::forward<Args>(args)...);
        });
        return m_instance;
    }

    Singleton(const Singleton &) = delete;
    Singleton &operator=(const Singleton &) = delete;
};

template<class T>
std::unique_ptr<T> Singleton<T>::m_instance = nullptr;
template<class T>
std::once_flag Singleton<T>::m_flag;
```

用法如下：

```c++
class Person : public Singleton<Person> {
public:
    Person(const std::string &name, int age)
        :m_name(name)
        ,m_age(age) {
    }

    void show_info() {
        std::cout << "姓名: " << m_name << std::endl;
        std::cout << "年龄: " << m_age << std::endl;
    }

private:
    std::string m_name;
    int m_age;
};

int main() {
    Person::instance("xxx", 15)->show_info();
    return 0;
}
```