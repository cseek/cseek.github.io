---
title: 组合
category: [软件设计]
tags: [uml, c++]
---

> 在 UML 中，组合表示一个对象包含另一个对象，并且两者的生命周期是紧密相关的。
{: .prompt-info }

## 示例

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