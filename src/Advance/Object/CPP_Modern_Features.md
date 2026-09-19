# C++新特性

C++11 是语言历史上最大的一次修订。它引入的**移动语义**、**右值引用**、`constexpr`、`noexcept` 等特性，彻底改变了 C++ 的类设计方式——从「如何正确管理资源」转向「如何让编译器自动帮你管理资源」。

本节聚焦这些特性在**面向对象设计**中的应用。关于拷贝构造、赋值运算符、析构函数的完整语义，见 [拷贝控制](./Copy_Control.md)。

## 一、值类别：左值与右值

理解移动语义的前提是理解**值类别（value category）**。

| 类别 | 含义 | 例子 |
| :--- | :--- | :--- |
| **左值** (lvalue) | 有名字、有确定地址、可取址 | 变量名、`*p`、`arr[0]` |
| **纯右值** (prvalue) | 临时对象、字面量，无名字 | `42`、`a + b`、`func()` 的返回值 |
| **将亡值** (xvalue) | 即将被销毁的对象，可被移动 | `std::move(x)` 的结果 |

左值和将亡值合称**泛左值**（glvalue），纯右值和将亡值合称**右值**（rvalue）。

关键区别：**左值持久，右值短暂**。右值通常是即将销毁的临时对象，因此可以安全地「窃取」它的资源。

```c++
int a = 10;          // a 是左值
int&& r = 10;        // r 是右值引用，绑定到临时值
int&& r2 = std::move(a);   // 把左值 a 转成右值引用
```

## 二、右值引用

右值引用用 `T&&` 表示，它只能绑定到右值：

```c++
int x = 1;
int& lref = x;        // 左值引用，绑定左值
int&& rref = 42;      // 右值引用，绑定右值
// int&& bad = x;     // 错误：右值引用不能绑定左值
```

右值引用的意义在于：**让函数能够区分「参数是临时对象」还是「参数是持久对象」**，从而对临时对象采取更高效的操作。

> [!TIP]
> **引用折叠与完美转发**
>
> 在模板中，`T&&` 的含义取决于 `T` 的推导结果，这被称为**转发引用**（forwarding reference）：
>
> - 传入左值 → `T` 推导为 `T&`，经引用折叠后 `T&&` 变成 `T&`
> - 传入右值 → `T` 推导为 `T`，`T&&` 保持为 `T&&`
>
> 引用折叠规则：`& + & = &`，`& + && = &`，`&& + && = &&`。
>
> 配合 `std::forward<T>(x)` 即可把参数原样转发，保持其值类别不变。

## 三、移动语义

### 1. 移动构造函数与移动赋值

移动构造函数接受右值引用，直接**接管**源对象的资源，而不是复制一份：

```c++
class Buffer
{
public:
    explicit Buffer(std::size_t n)
        : data_(new int[n]), size_(n) {}

    ~Buffer() { delete[] data_; }

    // 拷贝构造：深拷贝
    Buffer(const Buffer& other)
        : data_(new int[other.size_]), size_(other.size_)
    {
        std::copy(other.data_, other.data_ + size_, data_);
    }

    // 移动构造：接管资源，不分配内存
    Buffer(Buffer&& other) noexcept
        : data_(other.data_), size_(other.size_)
    {
        other.data_ = nullptr;   // 关键：源对象置空，避免析构时重复释放
        other.size_ = 0;
    }

    Buffer& operator=(Buffer&& other) noexcept
    {
        if (this != &other)
        {
            delete[] data_;          // 释放自己的资源
            data_ = other.data_;     // 接管对方的资源
            size_ = other.size_;
            other.data_ = nullptr;
            other.size_ = 0;
        }
        return *this;
    }

private:
    int* data_;
    std::size_t size_;
};
```

**移动的本质是「指针交换」**：把源对象的内部指针直接赋给目标对象，再把源对象的指针置空。这样既避免了深拷贝的开销，又保证了源对象析构时不会重复释放。

### 2. 源对象的状态

被移动后的对象处于**有效但未指定**（valid but unspecified）的状态：

- 可以安全地析构
- 可以重新赋值
- **不应该**假设它的值是什么（比如假设它一定为空）

### 3. 何时触发移动

移动在以下情况自动发生：

| 场景 | 说明 |
| :--- | :--- |
| 用临时对象初始化 | `Buffer b = makeBuffer();` |
| 用 `std::move` 显式转换 | `Buffer b2 = std::move(b1);` |
| 容器扩容时搬移元素 | `vector` 重新分配内存 |
| 返回局部对象 | NRVO 未生效时 |

```c++
std::vector<std::string> v;
std::string s = "hello";
v.push_back(s);              // 拷贝：s 是左值
v.push_back(std::move(s));   // 移动：s 被移空
v.push_back("world");        // 移动：临时对象
```

## 四、std::move 与 std::forward

### 1. std::move

`std::move` **不移动任何东西**，它只是一个类型转换，把左值转成右值引用：

```c++
// 大致实现
template <typename T>
constexpr std::remove_reference_t<T>&& move(T&& x) noexcept
{
    return static_cast<std::remove_reference_t<T>&&>(x);
}
```

它相当于告诉编译器：「这个对象我不再需要了，可以当作临时对象处理」。

```c++
std::string a = "hello";
std::string b = std::move(a);   // 触发移动构造
// a 现在是空字符串（有效但未指定）
```

> [!WARNING]
> **`std::move` 之后不要再使用源对象**
>
> `std::move` 只是转换，真正的移动发生在构造/赋值时。但一旦移动完成，源对象的内容就不可依赖了。
>
> 常见错误：
> ```c++
> std::string s = "hello";
> use(std::move(s));   // s 被移空
> log(s);              // 错误：s 的内容已不可依赖
> ```

### 2. std::forward

`std::forward` 用于**完美转发**：在模板中把参数原样传递给下一层函数，保持其值类别不变。

```c++
template <typename T>
void wrapper(T&& x)
{
    // 如果传入左值，x 以左值传递；传入右值，以右值传递
    process(std::forward<T>(x));
}
```

如果写成 `process(x)`，右值参数会被当作左值，退化为拷贝；如果写成 `process(std::move(x))`，左值参数会被错误地移动。

### 3. 对比

| | `std::move` | `std::forward` |
| :--- | :--- | :--- |
| 作用 | 无条件转为右值 | 有条件保持原值类别 |
| 使用场景 | 明确不再需要该对象 | 模板中的参数转发 |
| 参数 | 普通对象 | 转发引用 `T&&` |

## 五、constexpr

`constexpr` 表示「可在编译期求值」。它比 `const` 更强：`const` 只保证运行期不可修改，`constexpr` 则要求编译期就能算出结果。

```c++
const int a = 10;              // 运行期常量
constexpr int b = 10;          // 编译期常量

constexpr int square(int x) { return x * x; }
constexpr int c = square(5);   // 编译期算出 25

int arr[square(3)];            // 合法：数组大小是编译期常量
```

### 在类设计中的应用

```c++
class Circle
{
public:
    constexpr Circle(double r) : radius_(r) {}

    constexpr double area() const
    {
        return 3.141592653589793 * radius_ * radius_;
    }

private:
    double radius_;
};

constexpr Circle c(2.0);
constexpr double a = c.area();   // 编译期计算
```

**要点**：

- `constexpr` 函数在 C++11 中只能有一条 `return` 语句，C++14 起放宽为允许循环和分支
- `constexpr` 隐含 `inline`，定义需放在头文件中
- `constexpr` 构造函数使对象可以成为编译期常量
- C++20 起还有 `consteval`（强制编译期求值）和 `constinit`（强制编译期初始化）

## 六、noexcept 与类设计

`noexcept` 不只是文档说明，它会**直接影响容器的行为**。

`std::vector` 扩容时需要把旧元素搬到新内存。如果元素的移动构造函数**可能抛异常**，一旦搬运途中抛异常，旧内存已被破坏、新内存尚未完成，无法回滚——强异常安全保证就会被破坏。因此：

- 移动构造函数标记 `noexcept` → `vector` **移动**元素
- 移动构造函数未标记 `noexcept` → `vector` 退化为**拷贝**元素

```c++
class Good
{
public:
    Good(Good&&) noexcept = default;   // vector 会移动
};

class Bad
{
public:
    Bad(Bad&&) { /* 未标记 noexcept */ }   // vector 会选择拷贝
};
```

**实践建议**：

- 移动构造函数和移动赋值运算符**应尽可能标记 `noexcept`**
- 析构函数默认就是 `noexcept`（C++11 起），不要在其中抛异常
- `swap` 通常也应标记 `noexcept`，许多算法依赖这一点

## 七、其它现代特性

### 1. override 与 final

```c++
class Base
{
public:
    virtual void f();
    virtual void g();
};

class Derived : public Base
{
public:
    void f() override;        // 明确表示覆盖基类虚函数，签名不匹配会报错
    void g() final;           // 禁止派生类继续覆盖
};
```

`override` 能在编译期发现「本想覆盖却写错签名」的问题，应始终使用。

### 2. = default 与 = delete

```c++
class NonCopyable
{
public:
    NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete;             // 禁止拷贝
    NonCopyable& operator=(const NonCopyable&) = delete;
};
```

`= default` 让编译器生成默认实现，`= delete` 明确禁用某个函数。

### 3. 委托构造与继承构造

```c++
class Widget
{
public:
    Widget() : Widget(0, 0) {}          // 委托构造：调用另一个构造函数
    Widget(int w, int h) : w_(w), h_(h) {}

private:
    int w_, h_;
};

class Derived : public Base
{
public:
    using Base::Base;   // 继承基类的构造函数
};
```

### 4. 类内成员初始化

```c++
class Config
{
    int timeout_ = 30;               // 类内默认值
    std::string name_ = "default";
};
```

这样多个构造函数不必重复写相同的初始化。

### 5. 结构化绑定（C++17）

```c++
std::map<std::string, int> m{{"a", 1}};
for (const auto& [key, value] : m)
    std::cout << key << ": " << value << '\n';
```

### 6. 三路比较运算符 `<=>`（C++20）

```c++
struct Point
{
    int x, y;
    auto operator<=>(const Point&) const = default;   // 自动生成全部比较运算符
};
```

一行代码即可获得 `==`、`!=`、`<`、`<=`、`>`、`>=`。

## 八、注意事项

- **`std::move` 不是移动，只是转换**。它不会改变对象，真正的移动发生在构造或赋值时。
- **移动后不要再使用源对象**。它的状态是「有效但未指定」，只能析构或重新赋值。
- **移动构造函数要置空源对象的指针**。否则源对象析构时会重复释放资源。
- **`noexcept` 影响容器性能**。未标记 `noexcept` 的移动构造会让 `vector` 退化为拷贝。
- **不要对 `const` 对象用 `std::move`**。`const T&&` 无法绑定到移动构造的 `T&&`，结果仍是拷贝。
- **返回局部对象不需要 `std::move`**。编译器会做 NRVO（返回值优化），写 `std::move` 反而可能阻止优化。
- **`constexpr` 函数要放在头文件**。它隐含 `inline`，否则会违反 ODR。

## 九、相关章节

- [拷贝控制](./Copy_Control.md)：拷贝构造、赋值运算符与 Rule of Three/Five/Zero
- [异常处理](../Exception.md)：`noexcept` 说明符与异常安全保证
- [智能指针](../Memory/Smart_Pointer.md)：移动语义在资源管理中的应用
- [RAII与资源管理](../Memory/RAII.md)：资源所有权的转移
