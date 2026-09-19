# 类型转换与转换运算符

C++ 允许在类型之间自动转换，这带来了便利，也埋下了隐患。一个构造函数可能让编译器在你没意识到的情况下做转换；一个转换运算符可能让对象在意外的地方变成别的类型。

这一章讲清两件事：**怎么定义类型转换**，以及**怎么防止它们失控**。

## 一、转换构造函数

只接受**单个参数**的构造函数，会同时充当隐式转换的入口：

```c++
class Complex
{
public:
    Complex(double re, double im = 0) : re_(re), im_(im) {}

private:
    double re_, im_;
};

void print(const Complex& c);

print(3.14);   // 隐式转换：3.14 → Complex(3.14, 0)
```

`3.14` 是 `double`，但函数要 `Complex`，编译器发现 `Complex` 有个能接受 `double` 的构造函数，就自动调用了它。

这看起来方便，但会带来意外：

```c++
Complex a(1, 2);
Complex b = a + 3.0;   // 3.0 被隐式转成 Complex(3.0)
// 如果没定义 operator+，编译器会尝试各种转换组合
```

### explicit：禁止隐式转换

C++11 起，用 `explicit` 可以禁止这种自动转换：

```c++
class Complex
{
public:
    explicit Complex(double re, double im = 0) : re_(re), im_(im) {}
};

print(3.14);              // 编译错误
print(Complex(3.14));     // 显式构造，OK
Complex c = 3.14;         // 编译错误
Complex c{3.14};          // OK：直接初始化
```

**经验法则**：除非转换在语义上完全自然（比如 `std::string` 接受 `const char*`），否则构造函数都应该加 `explicit`。

C++17 起，`explicit` 还能用于**条件性显式**，配合类型转换运算符使用。

## 二、转换运算符

反过来，让对象能转换成别的类型：

```c++
class Fraction
{
public:
    Fraction(int num, int den) : num_(num), den_(den) {}

    // 转换运算符：Fraction → double
    operator double() const
    {
        return static_cast<double>(num_) / den_;
    }

private:
    int num_, den_;
};

Fraction f(1, 2);
double d = f;              // 隐式调用 operator double()
std::cout << f + 0.5;      // 1.0
```

转换运算符的语法是 `operator 目标类型()`：没有返回类型（返回类型就是目标类型），没有参数，通常加 `const`。

### explicit 转换运算符

同样可以加 `explicit` 防止隐式转换：

```c++
class SafeBool
{
public:
    explicit operator bool() const { return valid_; }

private:
    bool valid_;
};

SafeBool sb;
if (sb) { }              // OK：条件语句中允许显式转换
bool b = sb;             // 编译错误
bool b2 = static_cast<bool>(sb);   // OK
```

`operator bool` 是转换运算符中最常见的一个，用于让对象能用在条件判断中。

## 三、二义性问题

多个转换路径会导致编译器无法选择：

```c++
class A
{
public:
    A(int);
    operator int() const;
};

A a(1);
int x = a + 1;   // 歧义：是把 a 转成 int，还是把 1 转成 A？
```

编译器会报「ambiguous」错误。解决办法是**减少隐式转换的入口**——该加 `explicit` 就加。

另一个常见问题是有多个可行的转换序列：

```c++
class B
{
public:
    B(double);
    B(long);
};

B b = 42;   // 歧义：int → double 还是 int → long？
```

## 四、转换中的隐式步骤

一次隐式转换最多包含一个「用户定义转换」。但标准转换可以叠加：

```c++
class Widget
{
public:
    Widget(int);
};

void use(const Widget& w);

short s = 42;
use(s);   // short → int（标准转换）→ Widget（用户定义转换）
```

如果路径需要**两个**用户定义转换，编译器会拒绝：

```c++
class A { public: A(int); };
class B { public: B(const A&); };

void use(const B& b);

use(42);   // 错误：需要 int → A → B，两次用户定义转换
```

## 五、实际应用

### 1. 智能指针的 operator bool

```c++
auto p = std::make_unique<Widget>();

if (p)              // 调用 operator bool
    p->doSomething();

bool b = p;         // 编译错误：explicit 阻止了隐式转换
```

标准库的智能指针都用 `explicit operator bool`，这样既能用在条件判断中，又不会意外转换成整数。

### 2. string_view 的转换

```c++
class MyString
{
public:
    operator std::string_view() const   // 隐式转换到视图
    {
        return std::string_view(data_, size_);
    }

private:
    const char* data_;
    std::size_t size_;
};
```

这样 `MyString` 能直接传给接受 `string_view` 的函数，无需拷贝。

### 3. 数值类型的包装

```c++
class Meters
{
public:
    explicit Meters(double value) : value_(value) {}

    double value() const { return value_; }

private:
    double value_;
};

class Seconds
{
public:
    explicit Seconds(double value) : value_(value) {}
    double value() const { return value_; }

private:
    double value_;
};

// 用了 explicit，编译器不会允许这个：
// Meters m(10);
// Seconds s = m;   // 编译错误，防止单位混用
```

这是**强类型**的常见做法——用 `explicit` 防止不同单位的数值互相转换。

## 六、安全建议

**构造函数默认加 `explicit`**。除非转换在语义上完全自然，否则不要开放隐式转换。

**转换运算符也加 `explicit`**。特别是 `operator bool`，几乎所有情况下都该是 explicit。

**避免定义多个转换路径**。如果类既能从 `int` 构造，又能转成 `int`，很容易产生歧义。

**用命名函数代替转换运算符**。`toString()` 比 `operator std::string()` 更明确，也更容易搜索。

```c++
// 不推荐：隐式转换，调用点看不出发生了什么
std::string s = widget;

// 推荐：明确
std::string s = widget.toString();
```

**注意 `operator bool` 的特殊性**。它是唯一允许在「上下文转换」中使用的转换运算符：

```c++
if (obj) { }              // OK
while (obj) { }           // OK
bool b = obj;             // 仅当非 explicit 时 OK
int i = obj;              // 错误：不会转成 int
```

## 七、常见误区

**「单参数构造函数不会引起问题」** —— 它是最常见的隐式转换来源，应该默认加 `explicit`。

**「转换运算符比命名函数方便」** —— 方便的另一面是难以追踪。代码里突然出现类型转换，往往需要翻头文件才能搞清发生了什么。

**「`explicit` 会阻止所有转换」** —— 只阻止隐式转换。显式构造、`static_cast`、直接初始化都不受影响。

**「`operator bool` 不加 `explicit` 也没事」** —— 会导致对象能隐式转成 `int`，进而参与算术运算，产生难以发现的 bug。

**「转换运算符可以重载多个」** —— 可以，但多个转换目标很容易造成歧义。

**「隐式转换没有性能开销」** —— 转换构造函数会创建临时对象，可能有拷贝开销。`explicit` 不一定更快，但至少让开销可见。

## 八、相关章节

- [类型转换](../../Basis/Types/Type_Conversion.md)：`static_cast`、`dynamic_cast` 等类型转换运算符
- [运算符重载](./Operator_Overloading.md)：转换运算符的重载规则
- [拷贝控制](./Copy_Control.md)：转换产生的临时对象的生命周期
- [继承](./Inheritance.md)：派生类到基类的转换
