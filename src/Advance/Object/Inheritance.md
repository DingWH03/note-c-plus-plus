# 继承

继承（Inheritance）是面向对象编程的三大特征之一（封装、继承、多态）之一，它使得我们可以在已有类的基础上创建新的类，从而实现代码复用与层次结构的抽象建模。

C++ 的继承机制极其灵活，既可以实现简单的单继承（Single Inheritance），也可以进行复杂的多继承（Multiple Inheritance），甚至支持通过**虚继承（Virtual Inheritance）**来解决多重继承带来的“菱形问题”。

## 一、继承的基本概念

继承是从一个已有的类（称为基类 / 父类 Base Class）派生出一个新的类（称为派生类 / 子类 Derived Class）。
派生类自动拥有基类的成员（数据与函数），并可以在此基础上新增成员或重写行为。

语法格式如下：

```cpp
class 派生类名 : 继承方式 基类名 {
    // 派生类成员
};
```

## 二、基类构造与派生类构造顺序

派生类对象中包含了基类子对象，因此在创建派生类实例时，必须先调用基类构造函数来完成基类部分的初始化。

调用顺序如下：

1. 按声明顺序调用所有基类构造函数（从上到下）。
2. 再调用派生类自身构造函数。

销毁顺序则相反：
先调用派生类析构函数，再调用基类析构函数。

示例：

```cpp
class Base {
public:
    Base() { std::cout << "Base constructed\n"; }
    ~Base() { std::cout << "Base destroyed\n"; }
};

class Derived : public Base {
public:
    Derived() { std::cout << "Derived constructed\n"; }
    ~Derived() { std::cout << "Derived destroyed\n"; }
};

int main() {
    Derived d;
}
```

输出结果为：

```text
Base constructed
Derived constructed
Derived destroyed
Base destroyed
```

若基类构造函数需要参数，必须在派生类的构造函数初始化列表中显式调用：

```cpp
class Base {
public:
    Base(int x) { std::cout << "Base(" << x << ")\n"; }
};

class Derived : public Base {
public:
    Derived(int x) : Base(x) {
        std::cout << "Derived(" << x << ")\n";
    }
};
```

> 构造函数调用顺序是从“上至下”，析构顺序是从“下至上”。
> 这种严格的顺序保证了对象的完整性和资源的正确释放。

## 三、访问控制与继承权限

访问控制是继承体系中非常重要的机制，它决定了哪些成员可以在派生类中被访问。

继承方式主要有三种：

|      继承方式      | 基类 `public` 成员在派生类中的访问属性 | 基类 `protected` 成员在派生类中的访问属性 | 特点说明                 |
| :------------: | :----------------------: | :-------------------------: | :------------------- |
|   `public` 继承  |    `public` → `public`   |  `protected` → `protected`  | 常用方式，保持原有访问级别        |
| `protected` 继承 |  `public` → `protected`  |  `protected` → `protected`  | 用于希望限制外部访问但允许派生使用的情况 |
|  `private` 继承  |   `public` → `private`   |   `protected` → `private`   | 派生类对外隐藏基类接口          |

1. public 成员

    * 在 `public` 继承中保持 `public`，外部依然可以访问。
    * 在 `protected/private` 继承中则变为不可外部访问。

2. protected 成员

    * 在 `public/protected` 继承中可被派生类访问。
    * 在 `private` 继承中仅在本类可访问。

3. private 成员

    * 永远不能被派生类直接访问。

```cpp
class Base {
public:
    int a;
protected:
    int b;
private:
    int c;
};

class Derived : public Base {
public:
    void show() {
        a = 1;   // 可访问（public继承下保持public）
        b = 2;   // 可访问（protected继承下保持protected）
        // c = 3; // 不可访问（private成员永远不能被继承访问）
    }
};
```

在继承中，private 成员虽然被继承，但不可直接访问，只能通过基类的 `public` 或 `protected` 接口间接访问。

## 四、单继承与多继承

### 1. 单继承

最常见的继承方式，一个派生类只有一个直接基类：

```cpp
class Animal {
public:
    void eat() { std::cout << "Eating\n"; }
};

class Dog : public Animal {
public:
    void bark() { std::cout << "Barking\n"; }
};
```

`Dog` 继承了 `Animal` 的所有非私有成员，因此：

```cpp
Dog d;
d.eat();  // 继承自 Animal
d.bark(); // 自身成员
```

### 2. 多继承

C++ 支持一个类继承自多个基类，从而组合多种功能：

```cpp
class A {
public:
    void funcA() { std::cout << "A\n"; }
};

class B {
public:
    void funcB() { std::cout << "B\n"; }
};

class C : public A, public B {
public:
    void funcC() { std::cout << "C\n"; }
};
```

`C` 同时拥有 `A` 和 `B` 的成员：

```cpp
C obj;
obj.funcA();
obj.funcB();
obj.funcC();
```

但多继承容易引入**命名冲突**：

```cpp
class A { public: void show() { std::cout << "A\n"; } };
class B { public: void show() { std::cout << "B\n"; } };
class C : public A, public B {};

C c;
c.show();  // 二义性错误：不知该调用 A::show 还是 B::show
```

需使用作用域限定符解决：

```cpp
c.A::show();
```

## 五、多重继承的菱形问题

多继承中最著名的问题是“菱形继承问题（Diamond Problem）”。

示例：

```cpp
class A {
public:
    int value = 1;
};

class B : public A {};
class C : public A {};
class D : public B, public C {};
```

继承关系如下：

```text
    A
   / \
  B   C
   \ /
    D
```

此时，`D` 同时继承了两份 `A`，因此 `value` 出现二义性：

```cpp
D d;
d.value = 10;   // 二义性：B::A::value 与 C::A::value 冲突
```

## 六、虚继承（Virtual Inheritance）

为解决菱形问题，C++ 引入了 虚继承（Virtual Inheritance）。

通过在继承声明前加上关键字 `virtual`，可以让共同的基类只保留一份共享副本。

修改上例：

```cpp
class A {
public:
    int value = 1;
};

class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {};
```

此时：

```cpp
D d;
d.value = 10;   // 不再二义性，A仅保留一份
```

在对象布局上，编译器会通过额外的“虚基表指针（vbptr）”来实现共享基类的唯一性。
因此，虚继承的对象模型更复杂，但解决了最棘手的多继承冲突问题。

````admonish tip title='虚基表指针(vbptr)如何实现共享基类的唯一性？', collapsible=true
在普通多继承中，如果一个基类被多个派生类重复继承（如“菱形继承”结构），最底层派生类对象中会包含多份相同的基类成员，造成 **数据冗余** 和 **二义性**。

为了解决这一问题，C++ 提供了 **虚继承（virtual inheritance）**。编译器通过在对象中引入一套特殊的指针机制——**虚基表指针（`vbptr`）** 和 **虚基表（`vbtable`）**，来确保所有派生路径最终共享同一个虚基类实例。

* 当一个类以 `virtual` 方式继承基类时，编译器会在该类的对象布局中添加一个隐藏成员：`vbptr`。
* `vbptr` 指向一张由编译器生成的 **虚基表（vbtable）**。
* `vbtable` 中记录了**从当前对象地址到虚基类子对象地址的偏移量**。
* 当程序访问虚基类的成员时，编译器通过 `vbptr` 查表计算出正确的虚基类位置，确保所有继承路径共享同一个虚基类实例。

```cpp
class A { int x; };
class B : virtual public A {};
class C : virtual public A {};
class D : public B, public C {};
```

在 `D` 的对象布局中：

```
D
├── B 子对象（含 vbptr → 指向 B 的 vbtable）
├── C 子对象（含 vbptr → 指向 C 的 vbtable）
└── A 虚基类子对象（唯一一份）
```

访问 `A::x` 时：

* 若通过 `B` 或 `C` 访问，编译器都会通过 `vbptr` 找到唯一的 `A` 子对象；
* 从而避免了重复继承带来的“二义性”与“多份拷贝”问题。
````

> 虚继承常用于框架或接口设计中，尤其当多个中间层类共享同一顶层基类时。
