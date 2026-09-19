# 异常处理

**异常（Exception）** 是 C++ 报告错误的主要机制。当函数无法完成它承诺的任务时，可以**抛出**一个异常对象，控制权会沿着调用栈向上传递，直到遇到能处理它的 `catch` 块。

相比返回错误码，异常的优势在于：错误无法被忽略，且不需要在每一层函数中都写检查代码。代价是运行时有一定开销，且必须小心处理资源释放问题。

## 一、基本语法

异常处理由三个关键字组成：

| 关键字 | 作用 |
| :--- | :--- |
| `throw` | 抛出一个异常对象 |
| `try` | 标记一段可能抛出异常的代码 |
| `catch` | 捕获并处理特定类型的异常 |

```c++
#include <iostream>
#include <stdexcept>

double divide(double a, double b)
{
    if (b == 0)
        throw std::invalid_argument("除数不能为 0");
    return a / b;
}

int main()
{
    try
    {
        std::cout << divide(10, 2) << '\n';   // 5
        std::cout << divide(10, 0) << '\n';   // 抛出异常
    }
    catch (const std::invalid_argument& e)
    {
        std::cerr << "错误: " << e.what() << '\n';
    }

    return 0;
}
```

`throw` 之后的代码不会执行，控制权立即转移。

## 二、栈展开（Stack Unwinding）

异常抛出后，程序会沿着调用栈逐层退出，直到找到匹配的 `catch`。这个过程中，每一层作用域内的**局部对象都会被析构**，这一机制称为**栈展开**。

```c++
void inner()
{
    std::string s = "局部对象";
    throw std::runtime_error("出错了");
    // s 会在这里被析构
}

void outer()
{
    std::vector<int> v{1, 2, 3};
    inner();
    // v 也会被析构
}
```

栈展开是 **RAII**（Resource Acquisition Is Initialization）能够工作的基础：只要资源由对象持有，异常发生时析构函数就会自动释放它，不需要手写 `catch` 来清理。

```c++
void safe()
{
    std::ifstream file("data.txt");   // 构造时打开文件
    // ... 即使这里抛出异常
}   // file 的析构函数仍会被调用，文件自动关闭
```

> [!WARNING]
> **析构函数中不要抛出异常**
>
> 如果栈展开过程中某个析构函数又抛出异常，而此时已经有一个异常正在传播，程序会直接调用 `std::terminate` 终止。
>
> C++11 起，析构函数默认是 `noexcept` 的，在其中抛出异常会直接导致终止。

## 三、捕获异常

### 1. 按引用捕获

标准库抛出的都是**匿名对象**，应该**按引用捕获**：

```c++
try
{
    throw std::runtime_error("错误");
}
catch (const std::exception& e)   // 推荐：按 const 引用
{
    std::cerr << e.what() << '\n';
}
```

如果按值捕获，会发生**对象切片（object slicing）**——派生类异常被截断成基类，丢失派生部分的信息：

```c++
catch (std::exception e)   // 错误：切片，e 只是基类部分
{
    // e.what() 可能丢失派生类的信息
}
```

### 2. 捕获顺序

`catch` 块按**从上到下**的顺序匹配，第一个匹配的类型生效。因此**派生类必须写在基类之前**：

```c++
try
{
    // ...
}
catch (const std::out_of_range& e)     // 派生类在前
{
    // ...
}
catch (const std::exception& e)        // 基类在后
{
    // ...
}
```

顺序写反的话，基类的 `catch` 会先捕获所有派生类异常，后面的块永远不会执行。

### 3. 捕获所有异常

`catch (...)` 可以捕获任何类型的异常：

```c++
try
{
    // ...
}
catch (const std::exception& e)
{
    // 处理标准异常
}
catch (...)                            // 兜底
{
    // 处理其它未知异常
    throw;                             // 通常应该重新抛出
}
```

### 4. 重新抛出

在 `catch` 块内使用不带操作数的 `throw;` 可以把当前异常继续向上传播，同时保留原始类型：

```c++
try
{
    // ...
}
catch (const std::exception& e)
{
    log(e.what());   // 记录日志
    throw;           // 原样重新抛出，而不是 throw e;（后者会切片）
}
```

## 四、异常安全保证

标准库对每个操作都规定了异常安全等级。这四个等级是逐层包含的关系：

| 等级 | 含义 | 典型例子 |
| :--- | :--- | :--- |
| **Nothrow（不抛异常）** | 函数承诺不抛出异常 | 析构函数、`swap`、移动构造函数 |
| **Strong（强保证）** | 抛出异常时，状态**回滚**到调用前的样子 | `std::vector::push_back` |
| **Basic（基本保证）** | 抛出异常时，程序仍处于**有效状态**，不泄漏资源 | 大多数标准库操作 |
| **No guarantee（无保证）** | 可能泄漏资源或破坏不变量 | — |

此外还有 **exception-neutral（异常中立）**：函数自身不处理异常，但保证异常原样传播给调用者，且不破坏自身状态。模板库通常提供这种保证。

理解这几个等级有助于判断：调用某个操作失败后，对象还能不能继续使用。

```c++
std::vector<int> v{1, 2, 3};
try
{
    v.push_back(4);   // 强保证：失败时 v 仍为 {1,2,3}
}
catch (const std::bad_alloc&)
{
    // v 未被修改
}
```

## 五、noexcept

C++11 引入 `noexcept` 说明符，用于声明函数是否会抛出异常。

### 1. 说明符

```c++
void f() noexcept;              // 承诺不抛异常
void g() noexcept(true);        // 同上
void h() noexcept(false);       // 可能抛异常（默认行为）
```

如果声明为 `noexcept` 的函数真的抛出了异常，程序会直接调用 `std::terminate`：

```c++
void bad() noexcept
{
    throw std::runtime_error("糟糕");   // 编译通过，但运行时终止程序
}
```

### 2. `noexcept` 运算符

`noexcept(expr)` 是**运算符**，在编译期判断表达式是否会抛出异常，返回 `bool`：

```c++
std::vector<int> v;
std::cout << std::boolalpha << noexcept(v.push_back(1));   // false（可能抛 bad_alloc）
```

它常用于编写条件性的 `noexcept` 模板：

```c++
template <typename T>
void wrapper(T&& x) noexcept(noexcept(process(std::forward<T>(x))))
{
    process(std::forward<T>(x));
}
```

### 3. 为什么重要

`noexcept` 不只是文档说明，它会**影响容器的行为**。以 `std::vector` 扩容为例：

- 如果元素的**移动构造函数是 `noexcept`**，`vector` 会选择移动元素
- 否则，为了保持强异常安全保证，`vector` 只能选择**拷贝**元素

```c++
struct Good
{
    Good(Good&&) noexcept = default;   // vector 会移动
};

struct Bad
{
    Bad(Bad&&) { }                     // 未标记 noexcept，vector 会选择拷贝
};
```

因此，**自定义类型的移动构造函数应当尽可能标记为 `noexcept`**，否则会带来不必要的性能损失。

## 六、标准异常体系

标准库的所有异常都直接或间接继承自 `std::exception`，定义在 `<exception>` 中。它提供了虚函数 `what()` 返回错误描述：

```c++
namespace std
{
class exception
{
public:
    virtual const char* what() const noexcept;
    virtual ~exception();
};
}
```

主要派生类分两大类：

| 基类 | 含义 | 主要派生类 |
| :--- | :--- | :--- |
| `std::logic_error` | 程序逻辑错误，理论上可在运行前发现 | `invalid_argument`、`domain_error`、`length_error`、`out_of_range` |
| `std::runtime_error` | 运行时才能发现的错误 | `range_error`、`overflow_error`、`underflow_error` |

此外还有一些独立的异常类型：

| 异常 | 头文件 | 触发场景 |
| :--- | :--- | :--- |
| `std::bad_alloc` | `<new>` | `new` 分配内存失败 |
| `std::bad_cast` | `<typeinfo>` | `dynamic_cast` 引用转换失败 |
| `std::bad_typeid` | `<typeinfo>` | 对空指针解引用后取 `typeid` |
| `std::bad_weak_ptr` | `<memory>` | 用空的 `weak_ptr` 构造 `shared_ptr` |
| `std::bad_function_call` | `<functional>` | 调用空的 `std::function` |
| `std::bad_optional_access` | `<optional>` | 访问空的 `optional`（C++17） |
| `std::bad_variant_access` | `<variant>` | 访问 `variant` 中非当前类型（C++17） |
| `std::bad_any_cast` | `<any>` | `any_cast` 类型不匹配（C++17） |
| `std::system_error` | `<system_error>` | 操作系统或底层库错误（C++11） |

## 七、自定义异常

自定义异常通常继承自 `std::runtime_error` 或 `std::logic_error`，这样可以直接复用 `what()`：

```c++
#include <stdexcept>
#include <string>

class NetworkError : public std::runtime_error
{
public:
    explicit NetworkError(const std::string& msg, int code)
        : std::runtime_error(msg), code_(code) {}

    int code() const noexcept { return code_; }

private:
    int code_;
};
```

使用时按引用捕获基类即可统一处理，需要时再 `dynamic_cast` 取回具体类型。

## 八、注意事项

- **按值抛出，按引用捕获**。抛出时构造的是异常对象的副本，捕获时必须用引用以避免切片。
- **不要在析构函数中抛异常**。栈展开期间的二次异常会导致 `std::terminate`。
- **`catch (...)` 不要吞掉异常**。至少要记录日志，否则问题会被掩盖。
- **`throw;` 与 `throw e;` 不同**。前者原样重抛，后者会拷贝并切片。
- **异常不是控制流**。不要用异常实现正常的跳转逻辑，它的开销远高于普通的条件判断。
- **构造函数的失败必须用异常报告**。构造函数没有返回值，无法用错误码表示失败。
- **`noexcept` 是承诺而非检查**。编译器不会验证函数体是否真的不抛异常，违背承诺只在运行时报错。
- **性能考量**。异常在**不抛出**时几乎没有开销（零成本异常模型），但抛出时的栈展开代价较高。因此热路径上的正常情况不应依赖异常。

## 九、相关章节

- [RAII与资源管理](./Memory/RAII.md)：栈展开与资源自动释放
- [拷贝控制](./Object/Copy_Control.md)：移动构造函数与 `noexcept` 的关系
- [智能指针](./Memory/Smart_Pointer.md)：异常安全的资源管理
- [C++ 新特性](./Object/CPP_Modern_Features.md)：`noexcept` 在类设计中的应用
