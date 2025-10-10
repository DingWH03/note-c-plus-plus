# 类型转换

类型转换是将一个数据类型的值转换为另一种数据类型的值。在 C++ 中，类型转换主要有四种方式：静态转换、动态转换、常量转换和重新解释转换。每种转换方式有其特定的应用场景和注意事项。

## 静态转换（Static Cast）

静态转换用于将一个数据类型的值强制转换为另一种类型，通常适用于类型之间存在一定的相似性。例如，可以将 `int` 类型转换为 `float` 类型。静态转换并不会进行运行时的类型检查，因此在某些情况下，它可能会导致运行时错误，尤其是在转换较为复杂的对象时。

例如：

```cpp
int i = 10;
float f = static_cast<float>(i);  // 将 int 转换为 float
```

## 动态转换（Dynamic Cast）

动态转换是 C++ 中用于在继承体系中进行类型转换的方式。它通常用于将基类指针或引用转换为派生类指针或引用，特别是在涉及多态的场景中。与静态转换不同，动态转换会在运行时检查转换是否合法，确保程序的类型安全。如果转换失败，对于指针类型，它会返回 `nullptr`，而对于引用类型，则会抛出 `std::bad_cast` 异常。

例如：

```cpp
#include <iostream>

class Base {
public:
    virtual ~Base() = default;  // 基类必须有虚函数
};

class Derived : public Base {
public:
    void show() {
        std::cout << "Derived class method" << std::endl;
    }
};

int main() {
    Base* ptr_base = new Derived;  // 基类指针指向派生类对象

    // 将基类指针转换为派生类指针
    Derived* ptr_derived = dynamic_cast<Derived*>(ptr_base);

    if (ptr_derived) {
        ptr_derived->show();  // 成功转换，调用派生类方法
    } else {
        std::cout << "Dynamic cast failed!" << std::endl;
    }

    delete ptr_base;
    return 0;
}
```

输出：

```text
Derived class method
```

在转换引用时，也可以使用 `dynamic_cast`，不过如果转换失败，它会抛出异常：

```cpp
#include <iostream>
#include <typeinfo>

class Base {
public:
    virtual ~Base() = default;
};

class Derived : public Base {
public:
    void show() {
        std::cout << "Derived class method" << std::endl;
    }
};

int main() {
    Derived derived_obj;
    Base& ref_base = derived_obj;  // 基类引用绑定到派生类对象

    try {
        Derived& ref_derived = dynamic_cast<Derived&>(ref_base);
        ref_derived.show();  // 成功转换，调用派生类方法
    } catch (const std::bad_cast& e) {
        std::cout << "Dynamic cast failed: " << e.what() << std::endl;
    }

    return 0;
}
```

输出：

```text
Derived class method
```

动态转换提供了更高的安全性，确保在程序运行时可以正确地进行类型检查。尽管如此，它相对于静态转换而言，性能开销较高，因为需要进行类型验证。

## 常量转换（Const Cast）

常量转换（`const_cast`）用于去除对象的 `const` 限定符，从而使得一个 `const` 对象能够被修改。它并不会改变对象的实际类型，只是去掉了 `const` 属性。这种转换通常用于需要修改原本被声明为常量的对象时，尽管这种做法应谨慎使用。

例如：

```cpp
const int i = 10;
int& r = const_cast<int&>(i);  // 去除 const 限定符
```

此代码将 `const int` 转换为普通的 `int`，使得对 `i` 的修改变得可能。但请注意，修改一个原本是常量的对象可能导致未定义的行为。

## 重新解释转换（Reinterpret Cast）

重新解释转换（`reinterpret_cast`）允许将一个数据类型的值重新解释为另一种数据类型的值。它通常用于在指针类型之间进行转换，不会进行任何类型检查，因此可能会导致未定义的行为。这种转换应谨慎使用，特别是在不同类型之间进行转换时，必须确保数据在内存中的表示是兼容的。

例如：

```cpp
int i = 10;
float f = reinterpret_cast<float&>(i);  // 将 int 转换为 float
```

这段代码通过 `reinterpret_cast` 将 `int` 类型的数据重新解释为 `float` 类型。然而，重新解释转换的使用非常危险，因为它不会检查类型是否真正兼容。如果目标类型与原类型不兼容，可能会导致程序崩溃或者数据损坏。因此，`reinterpret_cast` 应该仅在非常确定数据类型兼容的情况下使用。
