# 虚函数与多态

多态（Polymorphism）是面向对象的核心特征之一，它让我们可以通过**统一的接口**来操作不同类型的对象。C++ 的多态是通过 **虚函数（virtual function）机制** 实现的，它使得函数调用在运行时（而非编译时）才确定具体执行哪个版本——这被称为 **动态绑定（Dynamic Binding）**。

## 一、虚函数表机制（vtable）

理解虚函数，必须先理解它背后的实现机制：**虚函数表（Virtual Table）**。

当一个类中包含虚函数时，编译器会为该类自动生成一张“虚函数表”，记录所有虚函数的地址。
每个含虚函数的对象中，会隐式包含一个指向这张表的指针（称为 vptr）。

1. 生成虚函数表（`vtable`）： 为该类自动生成一张只读表，其中记录了所有虚函数的地址。
2. 植入虚指针（`vptr`）： 每个含虚函数的对象中，会**隐式**包含一个指向这张表的指针（`vptr`）。在主流编译器中，`vptr` 通常是对象内存布局中的**第一个成员**。

> 换句话说，每个对象在内存中都携带一张“函数地址目录”，在调用虚函数时，程序会通过这张表找到该对象对应的函数实现。

一个典型的过程如下：

1. 为该类生成一张虚函数表（vtable）。
2. 每个对象内部会有一个指针（vptr），在构造函数中初始化，指向所属类的 vtable。
3. 当通过基类指针或引用调用虚函数时，程序会查找 vtable 来决定实际执行哪个函数。

例如：

```cpp
class Base {
public:
    virtual void show() { std::cout << "Base\n"; }
};

class Derived : public Base {
public:
    void show() override { std::cout << "Derived\n"; } // C++11 推荐使用 override
};

Base* p = new Derived();
p->show();  // 输出：Derived
```

编译器生成的伪逻辑相当于：

```cpp
(*(p->vptr[0]))(p);  // 从 vtable[0] 取出函数指针并调用
```

这就是 **动态绑定** 的本质。

## 二、纯虚函数与抽象类

有时，基类只定义接口而不提供具体实现，这时就需要 **纯虚函数（pure virtual function）**。

纯虚函数的定义方式是在声明末尾加上 `= 0`：

```cpp
class Shape {
public:
    // 纯虚函数，只提供接口，没有实现体
    virtual void draw() = 0;
};
```

当类中含有至少一个纯虚函数时，该类就成为 **抽象类（Abstract Class）**。

抽象类不能被实例化，只能被继承。

它的作用是定义一个统一的接口，让派生类去实现具体行为。

例如：

```cpp
class Shape {
public:
    virtual void draw() = 0;
};

class Circle : public Shape {
public:
    void draw() override { std::cout << "Drawing Circle\n"; } // 必须实现 draw()
};

// Shape s; // 错误：不能实例化抽象类
Circle c;    // 正确：Circle 实现了所有纯虚函数
```

`Shape` 不能直接创建对象，但 `Circle` 可以，因为它实现了 `draw()`。

这种方式让我们能通过“基类接口”统一管理不同对象，实现真正意义上的多态。

## 三、动态绑定 vs 静态绑定

**静态绑定（Static Binding）** 是在编译阶段就决定函数调用的目标，依据是**变量的声明类型**。
**动态绑定（Dynamic Binding）** 则在运行时根据**对象的实际类型**决定调用哪个函数。

| 绑定类型 | 发生时机 | 调用依据   | 示例          |
| :---- | :---- | :------ | :----------- |
| 静态绑定 | 编译期  | 变量声明类型 | 非虚函数调用、重载函数 |
| 动态绑定 | 运行期  | 对象实际类型 | 虚函数调用       |

示例：

```cpp
class Base {
public:
    void func() { std::cout << "Base::func\n"; }
    virtual void vfunc() { std::cout << "Base::vfunc\n"; }
};

class Derived : public Base {
public:
    void func() { std::cout << "Derived::func\n"; }
    void vfunc() override { std::cout << "Derived::vfunc\n"; }
};

Base* p = new Derived();
p->func();   // 静态绑定 -> 调用 Base::func (基类指针只能看到基类声明的非虚函数)
p->vfunc();  // 动态绑定 -> 调用 Derived::vfunc (通过 vtable 确定实际类型)
```

````admonish tip
强烈建议在派生类中重写虚函数时使用 `override` 关键字。
`override` 显式告知编译器该函数意图是重写基类的虚函数。如果签名（函数名、参数、const 性）与基类不匹配，编译器将报错，从而避免了重写失败导致的行为错误。
````

## 四、析构函数的虚函数问题

一个常见的陷阱：基类的析构函数必须声明为虚函数。

否则当通过**基类指针** `delete` 派生类对象时，如果析构函数不是虚函数，将发生静态绑定，只会调用基类的析构函数。
会导致派生类中特有的资源（如动态分配的内存、文件句柄等）将不会被释放，导致内存泄漏或资源泄漏。

```cpp
class Base {
public:
    // 必须是虚函数，确保通过基类指针删除时能触发动态绑定
    virtual ~Base() { std::cout << "Base destroyed\n"; }
};

class Derived : public Base {
private:
    int* data;
public:
    Derived() : data(new int[10]) {}
    ~Derived() override {
        delete[] data; // 派生类特有的资源释放
        std::cout << "Derived destroyed\n";
    }
};

Base* p = new Derived();
delete p;  // 1. 动态绑定调用 Derived::~Derived() 2. 调用 Base::~Base()
// 保证了所有资源的正确释放
```

## 五、虚函数的开销

虚函数机制提供了强大的灵活性（多态），但它不是零代价的。使用虚函数会带来一定的性能和内存开销：

1. 空间开销 (对象层面)： 每个含有虚函数的对象，都会增加一个 `vptr` 指针的存储空间（在 64 位系统上通常是 8 字节）。
2. 空间开销 (类层面)： 每个含有虚函数的类都需要生成一张 `vtable` 来存储函数指针。
3. 时间开销： 虚函数的调用比普通函数（静态绑定）多了一步**间接寻址**（查找 `vptr` \\(\to\\) 访问 `vtable` \\(\to\\) 调用函数）。虽然开销微小，但在高度性能敏感的循环中，可能会产生可见的影响。

因此，只有当确实需要多态特性时，才应该将函数声明为虚函数。
