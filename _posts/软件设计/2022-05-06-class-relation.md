---
title: 类之间常见的关系
category: [软件设计]
tags: [uml, c++]
---

> 合理选择类的关系设计能提高代码的扩展性和可维护性。例如，组合优先于继承、依赖注入解耦等设计原则，都基于这些关系的特点。常见的类关系如下表：
{: .prompt-info }

关系类型	|耦合强度	|生命周期依赖	|示例场景
----------|---------|-------------|-------
继承	   |最强	  |无	        |动物 → 狗
实现	   |强	      |无	        |Runnable → Car
组合	   |强	      |是	        |汽车 → 发动机
聚合	   |中等	  |否	        |班级 → 学生
关联	   |中等	  |否	        |学生 ↔ 课程
依赖	   |最弱	  |否	        |方法参数中的临时对象

## 组合
在 UML 中，组合表示一个对象包含另一个对象，并且两者的生命周期是一样的。
```c++
#include <iostream>

class Network {
public:
    void connect() {
        std::cout << "Connecting" << std::endl;
    }
};

class App {
private:
    Network m_network; // 对象成员

public:
    void run() {
        m_network.connect();
        std::cout << "Runnig" << std::endl;
    }
};

int main() {
    App app;
    app.run();
    while (true) {
        sleep(1);
    }
    return 0;
}
```

## 聚合
在 UML 中，聚合表示一个对象包含另一个对象，但两者的生命周期可以独立存在。

```c++
#include <iostream>

class Network
{
private:
    Network *network_; // 指针成员

public:
    void connect()
    {
        std::cout << "Connecting" << std::endl;
    }
};

class App
{
public:
    void run()
    {
        network_ = new Network;
        network_->connect();
        std::cout << "Runnig" << std::endl;
        delete network_;
    }
};

int main()
{
    App app;
    app.run();

    return 0;
}
```

## TODO