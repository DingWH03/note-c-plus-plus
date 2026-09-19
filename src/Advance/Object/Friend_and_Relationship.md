# 友元与类关系

封装是面向对象的基础——把数据藏起来，只暴露必要的接口。但有时两个类确实需要共享内部状态，比如容器和它的迭代器、对象和它的工厂函数。

`friend` 就是为这种情况准备的：它允许指定的函数或类访问私有成员。但它的代价是**打破封装**，所以用之前要想清楚是不是真的必要。

这一章还讨论类之间的几种关系——组合、聚合、依赖——它们决定了对象之间该怎么互相持有。

## 一、友元函数

友元函数不是成员函数，但能访问类的私有成员：

```c++
class Point
{
private:
    double x_, y_;

public:
    Point(double x, double y) : x_(x), y_(y) {}

    // 声明友元
    friend double distance(const Point& a, const Point& b);
};

// 定义（不需要 friend 关键字）
double distance(const Point& a, const Point& b)
{
    double dx = a.x_ - b.x_;   // 可以访问私有成员
    double dy = a.y_ - b.y_;
    return std::sqrt(dx * dx + dy * dy);
}
```

### 什么时候需要友元函数

**运算符重载**是最常见的场景。当运算符的左操作数不是类对象时，必须用非成员函数：

```c++
class Complex
{
    double re_, im_;

public:
    Complex(double re, double im) : re_(re), im_(im) {}

    // 需要访问私有成员，所以声明为友元
    friend std::ostream& operator<<(std::ostream& os, const Complex& c);
    friend Complex operator+(double lhs, const Complex& rhs);
};

std::ostream& operator<<(std::ostream& os, const Complex& c)
{
    return os << c.re_ << "+" << c.im_ << "i";
}

// 支持 2.0 + complex，而不仅仅是 complex + 2.0
Complex operator+(double lhs, const Complex& rhs)
{
    return Complex(lhs + rhs.re_, rhs.im_);
}
```

`operator<<` 的左操作数是 `ostream`，`double + Complex` 的左操作数是 `double`——都不是 `Complex` 对象，所以不能定义为成员函数。

### 友元的几个特性

- **友元关系不可传递**：A 是 B 的友元，B 是 C 的友元，不代表 A 是 C 的友元
- **友元关系不可继承**：基类的友元不能访问派生类的私有成员
- **友元声明不受访问控制影响**：可以放在 `private`、`protected` 或 `public` 区域，效果相同
- **友元不是成员**：不受 `this` 指针影响，调用方式与普通函数一致

## 二、友元类

整个类都可以成为友元：

```c++
class Matrix
{
private:
    double* data_;
    int rows_, cols_;

    friend class MatrixIterator;   // 迭代器需要访问内部数据
    friend class MatrixFactory;    // 工厂需要直接构造
};
```

友元类中的所有成员函数都能访问对方的私有成员。

一个典型场景是**迭代器**：

```c++
class Container
{
    int* data_;
    std::size_t size_;

public:
    class Iterator
    {
        int* ptr_;
    public:
        int& operator*() { return *ptr_; }
        Iterator& operator++() { ++ptr_; return *this; }
        // ...
    };

    Iterator begin() { return Iterator{data_}; }
    Iterator end()   { return Iterator{data_ + size_}; }
};
```

这里迭代器用嵌套类实现，作为成员类天然可以访问外部类的私有成员，不需要 `friend`。

### 友元成员函数

也可以只让某个类的特定成员函数成为友元：

```c++
class A;

class B
{
public:
    void accessA(A& a);
};

class A
{
    int secret_ = 42;

    // 只让 B::accessA 成为友元
    friend void B::accessA(A& a);
};
```

这种写法需要前置声明，且顺序敏感，实际中较少使用。

## 三、友元的代价

友元打破封装，这是它的本质。用之前先问几个问题：

**能不能通过公开接口实现？** 如果加个 `getX()` 就够了，就不需要友元。

**是不是设计有问题？** 如果两个类需要频繁互相访问私有成员，说明它们的职责可能没分清楚，也许应该合并成一个类，或者提取出共同的部分。

**能不能缩小范围？** 与其把整个类设为友元，不如只开放一两个成员函数。

不过友元也不是洪水猛兽。标准库中就有大量友元——`operator<<`、比较运算符、`swap` 函数通常都是友元。关键判断标准是：**这个外部函数在概念上是不是这个类的一部分**。

`operator<<` 显然是 `Complex` 的一部分（它就是用来打印 `Complex` 的），所以作为友元是合理的。反过来，如果某个业务函数需要访问类的私有数据，那就值得怀疑了。

## 四、类之间的关系

两个类之间可能有几种关系，选择合适的表达方式很重要。

### 1. 组合（Composition）—— 「是一个部分」

组合表示**强拥有**关系：整体销毁时部分也随之销毁。

```c++
class Engine
{
public:
    void start();
};

class Car
{
    Engine engine_;   // 直接持有对象
public:
    void start() { engine_.start(); }
};
```

特点：

- 成员是**值**，不是指针
- 生命周期完全绑定：`Car` 销毁，`Engine` 也销毁
- 通常用 `unique_ptr` 表示需要延迟构造或多态的组合

```c++
class Car
{
    std::unique_ptr<Engine> engine_;   // 独占所有权
public:
    Car() : engine_(std::make_unique<V8Engine>()) {}
};
```

### 2. 聚合（Aggregation）—— 「有一个」

聚合表示**弱拥有**关系：部分可以独立于整体存在。

```c++
class Department;

class University
{
    std::vector<Department*> departments_;   // 不拥有
public:
    void addDepartment(Department* d) { departments_.push_back(d); }
};
```

特点：

- 成员是指针或引用，**不负责生命周期**
- 被持有的对象可以在外部创建和销毁
- 需要调用者保证对象存活

如果生命周期不确定，可以用 `shared_ptr`；如果只是观察，用 `weak_ptr`。

### 3. 依赖（Dependency）—— 「用一下」

依赖是临时关系，通常通过函数参数体现：

```c++
class ReportGenerator
{
public:
    void generate(const Database& db, std::ostream& out);
};
```

`ReportGenerator` 不持有 `Database`，只是在调用时使用它。

### 4. 继承（Inheritance）—— 「是一个」

继承表示 **is-a** 关系：

```c++
class Shape { public: virtual void draw() = 0; };
class Circle : public Shape { public: void draw() override; };
```

继承的细节见 [继承](./Inheritance.md)。

### 关系对照

| 关系 | 语义 | 实现方式 | 生命周期 |
| :--- | :--- | :--- | :--- |
| 组合 | is-a-part-of | 值成员 / `unique_ptr` | 绑定 |
| 聚合 | has-a | 裸指针 / 引用 / `shared_ptr` | 独立 |
| 依赖 | uses-a | 函数参数 | 临时 |
| 继承 | is-a | 公有继承 | 绑定 |

## 五、设计建议

**优先用组合而非继承**。继承会带来强耦合，基类的改动会影响所有派生类。组合更灵活，也更容易测试。

**明确表达所有权**。用值成员表示「我拥有」，用裸指针表示「我不拥有」，用 `unique_ptr` 表示「独占但延迟构造」。不要让读者去猜。

**避免双向强引用**。A 持有 B，B 又持有 A，会导致循环引用和生命周期混乱。如果确实需要双向访问，让一方用弱引用。

**友元是最后手段**。先考虑公开接口、成员函数、嵌套类，都不行再用友元。

## 六、常见误区

**「友元破坏了封装，所以绝对不能用」** —— 友元是语言提供的工具，用在对的地方（如 `operator<<`）是合理的。关键是判断这个函数在概念上是否属于这个类。

**「友元关系可以继承」** —— 不能。派生类不会继承基类的友元关系。

**「友元关系可以传递」** —— 不能。A 是 B 的友元，B 是 C 的友元，A 不能访问 C 的私有成员。

**「友元声明的位置影响访问权限」** —— 不影响。放在 `private` 还是 `public` 区域都一样。

**「组合就是用一个指针成员」** —— 用指针不一定表示组合。裸指针通常表示不拥有（聚合），`unique_ptr` 才表示强拥有。

**「两个类互相是友元就没问题了」** —— 互相友元通常意味着设计有问题，两个类的职责可能没分清。

## 七、相关章节

- [继承](./Inheritance.md)：is-a 关系的细节
- [运算符重载](./Operator_Overloading.md)：为什么 `operator<<` 需要友元
- [拷贝控制](./Copy_Control.md)：成员对象的拷贝与移动
- [智能指针](../Memory/Smart_Pointer.md)：用智能指针表达所有权
