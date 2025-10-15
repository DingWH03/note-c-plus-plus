# 运算符重载（Operator Overloading）

## 一、概述

C++ 是一种支持面向对象编程的语言，在其类型系统中，除了内置类型（如 `int`、`double`）外，用户还可以自定义类（Class）来表示复杂数据结构。
然而，类的对象默认并不能像内置类型那样使用 `+`、`==`、`<<` 等运算符。为了让自定义类型具备直观、自然的操作方式，C++ 允许程序员**重定义（overload）运算符的行为**，使得运算符能作用于类的对象。

这种机制称为**运算符重载（Operator Overloading）**。它是 C++ 多态性的重要体现之一，使自定义类型可以拥有与内置类型一致的使用体验。

## 二、运算符重载的定义形式

运算符重载的本质是一个**函数定义**，它的函数名以 `operator` 开头，后接要重载的运算符符号。

基本形式如下：

```cpp
返回类型 operator运算符(参数列表);
```

运算符重载函数既可以定义为类的**成员函数**，也可以定义为**友元函数（friend function）**。
对于成员函数形式，函数的第一个操作数是当前对象本身（通过隐式的 `this` 指针传递）；
对于友元函数形式，则需要显式地接收两个参数。

## 三、可重载与不可重载的运算符

并非所有运算符都可以被重载。C++ 出于语言安全性和可维护性的考虑，对部分运算符做出了限制。

| 可重载的运算符                | 不可重载的运算符                           |
| :--------------------- | :--------------------------------- |
| `+  -  *  /  %`        | `.` （成员访问）                         |
| `==  !=  <  >  <=  >=` | `::` （作用域解析）                       |
| `+=  -=  *=  /=`       | `sizeof`                           |
| `[]  ()  ->`           | `?:` （条件运算符）                       |
| `<<  >>` （常用于输入输出）     | 类型转换运算符（如 `typeid`、`dynamic_cast`） |

此外，重载不会改变运算符的**优先级**和**结合性**，也不能引入新的运算符符号。

## 四、成员函数与友元函数重载的区别

1. 成员函数形式
   适用于当左操作数是类对象时。例如：

   ```cpp
   Complex a, b, c;
   c = a + b;     // 调用 a.operator+(b)
   ```

   在这种情况下，函数原型为：

   ```cpp
   Complex operator+(const Complex& rhs) const;
   ```

2. 友元函数形式
   当运算符左边不是该类的对象，或者需要访问私有成员时，使用友元函数更合适。

   ```cpp
   friend Complex operator+(const Complex& lhs, const Complex& rhs);
   ```

3. 主要区别

| 项目   | 成员函数             | 友元函数                |
| ---- | ---------------- | ------------------- |
| 左操作数 | 必须是类对象           | 可以不是类对象             |
| 调用形式 | `a.operator+(b)` | `operator+(a, b)`   |
| 访问权限 | 可直接访问成员          | 若声明为 friend，可访问私有成员 |
| 适用场景 | 一般用于类内操作         | 用于输入输出、双操作数等场合      |

这两种形式在使用时完全一致，可以直接使用`a+b`调用重载的符号。

## 五、算术运算符的重载

算术运算符（如 `+`、`-`、`*`、`/`）是最常见的重载形式之一。
下面以复数类 `Complex` 为例，说明如何重载加法运算符 `+`。

```cpp
#include <iostream>
class Complex {
private:
    double real, imag;

public:
    Complex(double r=0, double i=0) : real(r), imag(i) {}

    Complex operator+(const Complex& rhs) const {
        return Complex(real + rhs.real, imag + rhs.imag);
    }

    void display() const {
        std::cout << real << " + " << imag << "i" << std::endl;
    }
};

int main() {
    Complex a(2.0, 3.0), b(1.5, 2.0);
    Complex c = a + b;      // 调用 a.operator+(b)
    c.display();            // 输出 3.5 + 5i
}
```

该示例中，运算符函数返回一个新的对象，不改变原操作数的内容。

## 六、比较运算符的重载

对象之间的比较并不会自动发生。若要实现“对象是否相等”的判断，需要重载比较运算符。

示例：

```cpp
class Point {
private:
    int x, y;

public:
    Point(int a, int b) : x(a), y(b) {}

    bool operator==(const Point& rhs) const {
        return x == rhs.x && y == rhs.y;
    }

    bool operator!=(const Point& rhs) const {
        return !(*this == rhs);
    }
};

int main() {
    Point p1(1, 2), p2(1, 2), p3(2, 3);
    if (p1 == p2) std::cout << "p1 and p2 are equal" << std::endl;
    if (p1 != p3) std::cout << "p1 and p3 are not equal" << std::endl;
}
```

比较运算符通常返回布尔值，并应保证逻辑自洽性（如 `==` 和 `!=` 应互为补充）。

## 七、赋值运算符的重载

C++ 默认会生成一个“浅拷贝”的赋值运算符，这在涉及动态内存管理的类中往往会引发问题。
因此，通常需要手动重载 `operator=`，以实现**深拷贝（deep copy）**。

```cpp
#include <cstring>

class String {
private:
    char* data;

public:
    String(const char* s = "") {
        data = new char[strlen(s) + 1];
        strcpy(data, s);
    }

    String& operator=(const String& rhs) {
        if (this != &rhs) {                 // 防止自赋值
            delete[] data;
            data = new char[strlen(rhs.data) + 1];
            strcpy(data, rhs.data);
        }
        return *this;                       // 返回自身引用
    }

    ~String() { delete[] data; }
};
```

重载赋值运算符的注意事项：

1. 检查自赋值（防止对象赋给自己）。
2. 正确释放旧内存并重新分配。
3. 返回 `*this` 的引用以支持链式赋值。

## 八、输入输出运算符的重载

流插入运算符 `<<` 和流提取运算符 `>>` 无法作为成员函数重载，因为左操作数通常是标准流对象（如 `std::cout` 或 `std::cin`）。
因此，通常将它们重载为友元函数。

```cpp
#include <iostream>
using namespace std;

class Complex {
private:
    double real, imag;

public:
    Complex(double r=0, double i=0) : real(r), imag(i) {}

    friend ostream& operator<<(ostream& os, const Complex& c) {
        os << c.real << " + " << c.imag << "i";
        return os;
    }

    friend istream& operator>>(istream& is, Complex& c) {
        is >> c.real >> c.imag;
        return is;
    }
};

int main() {
    Complex c1;
    cin >> c1;
    cout << c1 << endl;
}
```

这种方式使得自定义类对象可以与标准输入输出流无缝交互。

## 九、自增与自减运算符的重载

C++ 区分前置与后置两种形式：

* 前置 `++a`：函数无形参；
* 后置 `a++`：函数带一个虚拟的 `int` 形参，用于区分。

```cpp
class Counter {
private:
    int value;

public:
    Counter(int v = 0) : value(v) {}

    Counter& operator++() {          // 前置 ++
        ++value;
        return *this;
    }

    Counter operator++(int) {        // 后置 ++
        Counter temp = *this;
        ++value;
        return temp;
    }

    int get() const { return value; }
};
```
