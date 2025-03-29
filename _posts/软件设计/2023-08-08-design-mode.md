---
title: 23种设计模式示例
category: [软件设计]
tags: [设计模式]
---

> 设计模式是解决软件设计中常见问题的可复用方案，分为三大类：创建型模式（5种）关注对象的高效创建与初始化，如单例、工厂；结构型模式（7种）处理对象间的组合与接口适配，如代理、装饰器；行为型模式（11种）管理对象间的通信与协作，如观察者、策略。这些模式通过解耦代码、提高复用性和增强扩展性，帮助开发者构建灵活、可维护的系统，例如通过适配器整合旧接口，利用责任链处理多级请求，或通过状态模式实现行为动态切换，是应对复杂软件架构挑战的经典工具箱。
{: .prompt-info } 


## 创建型（5种）
### 1. 单例模式（Singleton）

**场景**：全局配置管理、日志记录器、数据库连接池

懒汉式单例: 单例在第一次获取实例的时候才实例化。

```c++
#include <iostream>
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

饿汉式单例: 单例在进入 main 函数之前就已经实例化了。

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

### 2. 工厂方法模式（Factory Method）

**场景**：跨平台文件系统处理、文档格式转换器

```c++
#include <iostream>

// 产品接口
class Document {
public:
    virtual void Open() = 0;
    virtual ~Document() {}
};

class TextDocument : public Document {
public:
    void Open() override {
        std::cout << "Opening text document" << std::endl;
    }
};

// 创建者抽象类
class Application {
public:
    virtual Document* CreateDocument() = 0;
    
    void NewDocument() {
        Document* doc = CreateDocument();
        doc->Open();
        delete doc;
    }
};

class TextApplication : public Application {
public:
    Document* CreateDocument() override {
        return new TextDocument();
    }
};

int main() {
    Application* app = new TextApplication();
    app->NewDocument();  // Output: Opening text document
    delete app;
    return 0;
}
```

### 3. 抽象工厂模式（Abstract Factory）

**场景**：跨平台UI组件、游戏装备套装

```c++
#include <iostream>

// 抽象产品
class Button {
public:
    virtual void Paint() = 0;
    virtual ~Button() {}
};

class WinButton : public Button {
public:
    void Paint() override {
        std::cout << "Windows style button" << std::endl;
    }
};

// 抽象工厂
class GUIFactory {
public:
    virtual Button* CreateButton() = 0;
    virtual ~GUIFactory() {}
};

class WinFactory : public GUIFactory {
public:
    Button* CreateButton() override {
        return new WinButton();
    }
};

int main() {
    GUIFactory* factory = new WinFactory();
    Button* btn = factory->CreateButton();
    btn->Paint();  // Output: Windows style button
    delete btn;
    delete factory;
    return 0;
}
```

### 4. 建造者模式（Builder）

**场景**：复杂报表生成、餐饮套餐组合

```c++
#include <iostream>
#include <string>

class Pizza {
public:
    void SetDough(const std::string& dough) { dough_ = dough; }
    void SetSauce(const std::string& sauce) { sauce_ = sauce; }
    void ShowPizza() {
        std::cout << "Pizza with " << dough_ << " dough and "
                  << sauce_ << " sauce" << std::endl;
    }

private:
    std::string dough_;
    std::string sauce_;
};

class PizzaBuilder {
public:
    virtual void BuildDough() = 0;
    virtual void BuildSauce() = 0;
    virtual Pizza* GetPizza() = 0;
    virtual ~PizzaBuilder() {}
};

class HawaiianPizzaBuilder : public PizzaBuilder {
public:
    HawaiianPizzaBuilder() { pizza_ = new Pizza(); }
    
    void BuildDough() override { pizza_->SetDough("cross"); }
    void BuildSauce() override { pizza_->SetSauce("mild"); }
    Pizza* GetPizza() override { return pizza_; }

private:
    Pizza* pizza_;
};

class Cook {
public:
    void MakePizza(PizzaBuilder* builder) {
        builder->BuildDough();
        builder->BuildSauce();
    }
};

int main() {
    Cook cook;
    HawaiianPizzaBuilder builder;
    cook.MakePizza(&builder);
    Pizza* pizza = builder.GetPizza();
    pizza->ShowPizza();  // Output: Pizza with cross dough and mild sauce
    delete pizza;
    return 0;
}
```

### 5. 原型模式（Prototype）

**场景**：游戏对象克隆、复杂对象初始化

```c++
#include <iostream>
#include <memory>

class Prototype {
public:
    virtual std::unique_ptr<Prototype> Clone() const = 0;
    virtual void PrintID() const = 0;
    virtual ~Prototype() {}
};

class ConcretePrototype : public Prototype {
public:
    ConcretePrototype(int id) : id_(id) {}
    
    std::unique_ptr<Prototype> Clone() const override {
        return std::make_unique<ConcretePrototype>(*this);
    }

    void PrintID() const override {
        std::cout << "Instance ID: " << id_ << std::endl;
    }

private:
    int id_;
};

int main() {
    ConcretePrototype original(1001);
    auto clone = original.Clone();
    original.PrintID();  // Output: Instance ID: 1001
    clone->PrintID();    // Output: Instance ID: 1001
    return 0;
}
```

## 结构型（7种）
### 1. 适配器模式（Adapter）

**场景**：集成旧系统接口、第三方库兼容

```c++
#include <iostream>

// 旧式圆形接口
class LegacyCircle {
public:
    void Draw(int x, int y, int r) {
        std::cout << "Legacy circle at (" << x << "," << y << ") radius " << r << std::endl;
    }
};

// 新标准图形接口
class Shape {
public:
    virtual void Draw(float x, float y, float radius) = 0;
    virtual ~Shape() {}
};

// 适配器类
class CircleAdapter : public Shape {
public:
    void Draw(float x, float y, float radius) override {
        legacyCircle_.Draw(static_cast<int>(x), static_cast<int>(y), static_cast<int>(radius));
    }

private:
    LegacyCircle legacyCircle_;
};

int main() {
    Shape* shape = new CircleAdapter();
    shape->Draw(5.5f, 10.2f, 3.0f);  // Output: Legacy circle at (5,10) radius 3
    delete shape;
    return 0;
}
```

### 2. 桥接模式（Bridge）

**场景**：跨平台渲染引擎、设备抽象层

```c++
#include <iostream>

// 实现接口
class Renderer {
public:
    virtual void RenderCircle(float x, float y, float radius) = 0;
    virtual ~Renderer() {}
};

// 具体实现：OpenGL
class OpenGLRenderer : public Renderer {
public:
    void RenderCircle(float x, float y, float radius) override {
        std::cout << "OpenGL rendering circle at (" << x << "," << y << ") r=" << radius << std::endl;
    }
};

// 抽象接口
class Shape {
protected:
    Renderer& renderer_;
public:
    Shape(Renderer& renderer) : renderer_(renderer) {}
    virtual void Draw() = 0;
    virtual ~Shape() {}
};

class Circle : public Shape {
    float x_, y_, radius_;
public:
    Circle(Renderer& renderer, float x, float y, float r)
        : Shape(renderer), x_(x), y_(y), radius_(r) {}
    
    void Draw() override {
        renderer_.RenderCircle(x_, y_, radius_);
    }
};

int main() {
    OpenGLRenderer renderer;
    Circle circle(renderer, 5, 10, 3);
    circle.Draw();  // Output: OpenGL rendering circle at (5,10) r=3
    return 0;
}
```

### 3. 组合模式（Composite）

**场景**：文件系统管理、GUI容器组件

```c++
#include <iostream>
#include <vector>

class Graphic {
public:
    virtual void Draw() = 0;
    virtual void Add(Graphic* g) {}
    virtual ~Graphic() {}
};

class Circle : public Graphic {
public:
    void Draw() override {
        std::cout << "Drawing Circle" << std::endl;
    }
};

class CompositeGraphic : public Graphic {
    std::vector<Graphic*> children_;
public:
    void Draw() override {
        for (auto& child : children_) {
            child->Draw();
        }
    }
    
    void Add(Graphic* g) override {
        children_.push_back(g);
    }
};

int main() {
    CompositeGraphic container;
    container.Add(new Circle());
    container.Add(new Circle());
    
    container.Draw();
    // Output:
    // Drawing Circle
    // Drawing Circle
    return 0;
}
```

### 4. 装饰器模式（Decorator）

**场景**：动态添加功能、数据流处理链

```c++
#include <iostream>
#include <string>

class DataSource {
public:
    virtual void WriteData(const std::string& data) = 0;
    virtual ~DataSource() {}
};

class FileDataSource : public DataSource {
public:
    void WriteData(const std::string& data) override {
        std::cout << "Writing raw data: " << data << std::endl;
    }
};

class DataSourceDecorator : public DataSource {
protected:
    DataSource* wrappee_;
public:
    DataSourceDecorator(DataSource* source) : wrappee_(source) {}
};

class EncryptionDecorator : public DataSourceDecorator {
public:
    using DataSourceDecorator::DataSourceDecorator;
    
    void WriteData(const std::string& data) override {
        std::string encrypted = "Encrypted[" + data + "]";
        wrappee_->WriteData(encrypted);
    }
};

int main() {
    DataSource* source = new EncryptionDecorator(new FileDataSource());
    source->WriteData("Secret");  // Output: Writing raw data: Encrypted[Secret]
    delete source;
    return 0;
}
```

### 5. 外观模式（Facade）

**场景**：简化复杂子系统调用、API封装

```c++
#include <iostream>

class CPU {
public:
    void Boot() { std::cout << "CPU booting...\n"; }
};

class Memory {
public:
    void Check() { std::cout << "Memory check...\n"; }
};

class ComputerFacade {
    CPU cpu_;
    Memory memory_;
public:
    void Start() {
        memory_.Check();
        cpu_.Boot();
        std::cout << "System ready\n";
    }
};

int main() {
    ComputerFacade computer;
    computer.Start();
    // Output:
    // Memory check...
    // CPU booting...
    // System ready
    return 0;
}
```

### 6. 享元模式（Flyweight）

**场景**：文本编辑器字符格式管理、游戏粒子系统

```c++
#include <iostream>
#include <unordered_map>

class CharacterStyle {
    std::string font_;
    int size_;
    bool bold_;
public:
    CharacterStyle(const std::string& font, int size, bool bold)
        : font_(font), size_(size), bold_(bold) {}
    
    void ApplyStyle() const {
        std::cout << "Apply style: " << font_ << " " << size_
                  << (bold_ ? " bold" : "") << std::endl;
    }
};

class StyleFactory {
    std::unordered_map<std::string, CharacterStyle*> styles_;
public:
    CharacterStyle* GetStyle(const std::string& font, int size, bool bold) {
        std::string key = font + std::to_string(size) + (bold ? "1" : "0");
        if (!styles_.count(key)) {
            styles_[key] = new CharacterStyle(font, size, bold);
        }
        return styles_[key];
    }
    
    ~StyleFactory() {
        for (auto& pair : styles_) delete pair.second;
    }
};

int main() {
    StyleFactory factory;
    auto style1 = factory.GetStyle("Arial", 12, false);
    auto style2 = factory.GetStyle("Arial", 12, false);  // 复用对象
    
    style1->ApplyStyle();  // Output: Apply style: Arial 12
    return 0;
}
```

### 7. 代理模式（Proxy）

**场景**：延迟加载、访问控制、远程服务调用

```c++
#include <iostream>
#include <string>

class Image {
public:
    virtual void Display() = 0;
    virtual ~Image() {}
};

class RealImage : public Image {
    std::string filename_;
public:
    RealImage(const std::string& filename) : filename_(filename) {
        LoadFromDisk();
    }
    
    void Display() override {
        std::cout << "Displaying " << filename_ << std::endl;
    }
    
private:
    void LoadFromDisk() {
        std::cout << "Loading " << filename_ << " from disk..." << std::endl;
    }
};

class ImageProxy : public Image {
    RealImage* realImage_ = nullptr;
    std::string filename_;
public:
    ImageProxy(const std::string& filename) : filename_(filename) {}
    
    void Display() override {
        if (!realImage_) {
            realImage_ = new RealImage(filename_);
        }
        realImage_->Display();
    }
    
    ~ImageProxy() {
        delete realImage_;
    }
};

int main() {
    Image* image = new ImageProxy("photo.jpg");
    // 此时尚未加载真实图片
    image->Display();  // 第一次调用时加载
    // Output:
    // Loading photo.jpg from disk...
    // Displaying photo.jpg
    delete image;
    return 0;
}
```

## 行为型（11种）
### 1. 责任链模式（Chain of Responsibility）

**场景**：审批流程、异常处理链

```c++
#include <iostream>
#include <string>

class Handler {
protected:
    Handler* next_ = nullptr;
public:
    virtual void HandleRequest(const std::string& request) = 0;
    void SetNext(Handler* next) { next_ = next; }
    virtual ~Handler() {}
};

class ConcreteHandlerA : public Handler {
public:
    void HandleRequest(const std::string& request) override {
        if (request == "A") {
            std::cout << "Handler A processed request\n";
        } else if (next_) {
            next_->HandleRequest(request);
        }
    }
};

class ConcreteHandlerB : public Handler {
public:
    void HandleRequest(const std::string& request) override {
        if (request == "B") {
            std::cout << "Handler B processed request\n";
        } else if (next_) {
            next_->HandleRequest(request);
        }
    }
};

int main() {
    Handler* chain = new ConcreteHandlerA();
    chain->SetNext(new ConcreteHandlerB());
    
    chain->HandleRequest("B");  // Output: Handler B processed request
    delete chain;
    return 0;
}
```

### 2. 命令模式（Command）

**场景**：GUI操作、事务管理

```c++
#include <iostream>

class Light {
public:
    void On() { std::cout << "Light is on\n"; }
    void Off() { std::cout << "Light is off\n"; }
};

class Command {
public:
    virtual void Execute() = 0;
    virtual ~Command() {}
};

class LightOnCommand : public Command {
    Light& light_;
public:
    LightOnCommand(Light& light) : light_(light) {}
    void Execute() override { light_.On(); }
};

class RemoteControl {
    Command* command_;
public:
    void SetCommand(Command* cmd) { command_ = cmd; }
    void PressButton() { command_->Execute(); }
};

int main() {
    Light light;
    RemoteControl remote;
    remote.SetCommand(new LightOnCommand(light));
    remote.PressButton();  // Output: Light is on
    return 0;
}
```

### 3. 解释器模式（Interpreter）

**场景**：SQL解析、规则引擎

```c++
#include <iostream>
#include <string>
#include <unordered_map>

class Context {
    std::unordered_map<std::string, bool> variables_;
public:
    void SetVariable(const std::string& var, bool value) {
        variables_[var] = value;
    }
    bool GetVariable(const std::string& var) {
        return variables_[var];
    }
};

class Expression {
public:
    virtual bool Interpret(Context& context) = 0;
    virtual ~Expression() {}
};

class Variable : public Expression {
    std::string name_;
public:
    Variable(const std::string& name) : name_(name) {}
    bool Interpret(Context& context) override {
        return context.GetVariable(name_);
    }
};

int main() {
    Context context;
    context.SetVariable("X", true);
    Expression* expr = new Variable("X");
    std::cout << std::boolalpha << expr->Interpret(context);  // Output: true
    delete expr;
    return 0;
}
```

### 4. 迭代器模式（Iterator）

**场景**：集合遍历、文件系统导航

```c++
#include <iostream>
#include <vector>

template <typename T>
class Iterator {
public:
    virtual T Next() = 0;
    virtual bool HasNext() = 0;
    virtual ~Iterator() {}
};

template <typename T>
class VectorIterator : public Iterator<T> {
    std::vector<T> data_;
    size_t index_ = 0;
public:
    VectorIterator(const std::vector<T>& data) : data_(data) {}
    T Next() override { return data_[index_++]; }
    bool HasNext() override { return index_ < data_.size(); }
};

int main() {
    std::vector<int> numbers{1, 2, 3};
    Iterator<int>* it = new VectorIterator<int>(numbers);
    while (it->HasNext()) {
        std::cout << it->Next() << " ";  // Output: 1 2 3
    }
    delete it;
    return 0;
}
```

### 5. 中介者模式（Mediator）

**场景**：聊天室、GUI组件协调

```c++
#include <iostream>
#include <string>
#include <vector>

class Colleague;

class Mediator {
public:
    virtual void Notify(Colleague* sender, const std::string& msg) = 0;
};

class Colleague {
protected:
    Mediator* mediator_;
public:
    Colleague(Mediator* mediator) : mediator_(mediator) {}
    virtual ~Colleague() {}
};

class ConcreteColleagueA : public Colleague {
public:
    using Colleague::Colleague;
    void Send(const std::string& msg) {
        mediator_->Notify(this, msg);
    }
    void Receive(const std::string& msg) {
        std::cout << "Colleague A received: " << msg << "\n";
    }
};

class ConcreteMediator : public Mediator {
    std::vector<Colleague*> colleagues_;
public:
    void AddColleague(Colleague* col) {
        colleagues_.push_back(col);
    }
    void Notify(Colleague* sender, const std::string& msg) override {
        for (auto col : colleagues_) {
            if (col != sender) {
                if (auto a = dynamic_cast<ConcreteColleagueA*>(col)) {
                    a->Receive(msg);
                }
            }
        }
    }
};

int main() {
    ConcreteMediator mediator;
    ConcreteColleagueA c1(&mediator);
    ConcreteColleagueA c2(&mediator);
    mediator.AddColleague(&c1);
    mediator.AddColleague(&c2);
    
    c1.Send("Hello");  // Output: Colleague A received: Hello
    return 0;
}
```

### 6. 备忘录模式（Memento）

**场景**：撤销操作、游戏存档

```c++
#include <iostream>
#include <string>

class EditorMemento {
    std::string content_;
public:
    EditorMemento(const std::string& content) : content_(content) {}
    std::string GetContent() const { return content_; }
};

class Editor {
    std::string content_;
public:
    void Write(const std::string& text) { content_ += text; }
    EditorMemento* Save() { return new EditorMemento(content_); }
    void Restore(EditorMemento* memento) { content_ = memento->GetContent(); }
    void Show() { std::cout << "Content: " << content_ << "\n"; }
};

int main() {
    Editor editor;
    editor.Write("Hello ");
    auto save = editor.Save();
    editor.Write("World");
    editor.Show();  // Output: Content: Hello World
    
    editor.Restore(save);
    editor.Show();  // Output: Content: Hello 
    delete save;
    return 0;
}
```

### 7. 观察者模式（Observer）

**场景**：事件通知、数据监控

```c++
#include <iostream>
#include <vector>

class Observer {
public:
    virtual void Update(float temp) = 0;
    virtual ~Observer() {}
};

class Subject {
    std::vector<Observer*> observers_;
    float temperature_;
public:
    void Attach(Observer* obs) { observers_.push_back(obs); }
    void SetTemperature(float temp) {
        temperature_ = temp;
        Notify();
    }
    void Notify() {
        for (auto obs : observers_) {
            obs->Update(temperature_);
        }
    }
};

class Display : public Observer {
public:
    void Update(float temp) override {
        std::cout << "Temperature updated: " << temp << "℃\n";
    }
};

int main() {
    Subject weatherStation;
    Display display;
    weatherStation.Attach(&display);
    weatherStation.SetTemperature(25.5f);  // Output: Temperature updated: 25.5℃
    return 0;
}
```

### 8. 状态模式（State）

**场景**：订单状态机、游戏角色状态

```c++
#include <iostream>

class State {
public:
    virtual void Handle() = 0;
    virtual ~State() {}
};

class ConcreteStateA : public State {
public:
    void Handle() override { std::cout << "State A handling\n"; }
};

class Context {
    State* state_ = nullptr;
public:
    void SetState(State* state) { state_ = state; }
    void Request() { state_->Handle(); }
};

int main() {
    Context context;
    context.SetState(new ConcreteStateA());
    context.Request();  // Output: State A handling
    return 0;
}
```

### 9. 策略模式（Strategy）

**场景**：支付方式选择、排序算法

```c++
#include <iostream>

class PaymentStrategy {
public:
    virtual void Pay(int amount) = 0;
    virtual ~PaymentStrategy() {}
};

class CreditCardPayment : public PaymentStrategy {
public:
    void Pay(int amount) override {
        std::cout << "Paid " << amount << " via Credit Card\n";
    }
};

class PaymentContext {
    PaymentStrategy* strategy_;
public:
    PaymentContext(PaymentStrategy* strategy) : strategy_(strategy) {}
    void ExecutePayment(int amount) { strategy_->Pay(amount); }
};

int main() {
    PaymentContext context(new CreditCardPayment());
    context.ExecutePayment(100);  // Output: Paid 100 via Credit Card
    return 0;
}
```

### 10. 模板方法模式（Template Method）

**场景**：算法框架定义、流程标准化

```c++
#include <iostream>

class DataProcessor {
public:
    void Process() {
        LoadData();
        AnalyzeData();
        SaveResult();
    }
    virtual ~DataProcessor() {}
protected:
    virtual void LoadData() = 0;
    virtual void AnalyzeData() { std::cout << "Default analysis\n"; }
    virtual void SaveResult() = 0;
};

class CSVProcessor : public DataProcessor {
protected:
    void LoadData() override { std::cout << "Loading CSV\n"; }
    void SaveResult() override { std::cout << "Saving CSV results\n"; }
};

int main() {
    DataProcessor* processor = new CSVProcessor();
    processor->Process();
    // Output:
    // Loading CSV
    // Default analysis
    // Saving CSV results
    delete processor;
    return 0;
}
```

### 11. 访问者模式（Visitor）

**场景**：编译器AST处理、复杂结构操作

```c++
#include <iostream>

class Element;
class Visitor {
public:
    virtual void Visit(Element& element) = 0;
    virtual ~Visitor() {}
};

class Element {
public:
    virtual void Accept(Visitor& visitor) = 0;
    virtual ~Element() {}
};

class ConcreteElement : public Element {
public:
    void Accept(Visitor& visitor) override { visitor.Visit(*this); }
    void Operation() { std::cout << "Element operation\n"; }
};

class ConcreteVisitor : public Visitor {
public:
    void Visit(Element& element) override {
        std::cout << "Visitor processing element\n";
        dynamic_cast<ConcreteElement&>(element).Operation();
    }
};

int main() {
    ConcreteElement element;
    ConcreteVisitor visitor;
    element.Accept(visitor);
    // Output:
    // Visitor processing element
    // Element operation
    return 0;
}
```