# std::optional

`std::optional` 是 C++17 引入的一个工具类，用来表示一个 **可选值**。它可以包含某个类型的值，也可以为空（即没有值）。这在函数返回值和状态表达上尤其有用，可以避免使用特殊标志值（如 `-1` 或 `nullptr`）来表示“无效”或“不存在”的情况。它本质上是对 **“可空值”** 概念的抽象：

- **有值 (engaged)**：存储一个 `T` 类型对象，保证其生命周期绑定到 `optional` 对象上。
- **无值 (disengaged)**：表示空状态，此时不存储任何对象。

相比于传统的返回 `nullptr` 或 `std::pair<T,bool>` 表示失败，`std::optional` 更加语义化、类型安全，尤其适合构造成本较高的对象。

其模板定义如下：

```cpp
template< class T >
class optional;
```

## 1. 引入

被包括在 `<optional>` 头文件中。

```cpp
#include <optional>
```

`T` 必须满足 **Destructible**（可析构）。

不允许是引用类型、数组类型、函数类型或 `void`。

从 C++26 起，`optional<T>` 还支持作为 **range view**。

## 2. `std::optional` 的工作原理

- `optional<T>` 内部通过一个 **布尔标志** + **可能存在的值存储区** 来实现：
  - 当 engaged 时，`optional` 内部构造并管理一个 `T` 对象。
  - 当 disengaged 时，标志为 false，存储区未构造对象。

这使得 `optional` 更接近于 “可能为空的对象”，而不是指针。
 即使提供了 `operator*` 和 `operator->`，它依旧是值语义而非指针语义。

### (1) 为什么需要 `std::optional`？

传统做法：

- C 风格函数通常用返回值来表示错误，用输出参数来返回数据。
- 某些场景需要用 `nullptr` 或 `-1` 表示“无效”，但这并不类型安全。

`std::optional` 提供了一个清晰、安全的方式：返回“要么有值，要么没有值”。

### (2) 基本使用

```cpp
std::optional<int> maybeInt;           // 默认构造：无值
std::optional<int> hasInt = 42;        // 构造：有值
std::optional<int> emptyInt = std::nullopt; // 显式无值

if (hasInt) {
    std::cout << "Value: " << *hasInt << "\n";  // 解引用访问
}
```

## 3. 使用 `std::optional` 的场景

### (1) 函数返回值

最常见的场景是函数可能“有值”或“无值”。

```cpp
std::optional<int> findValue(const std::vector<int>& vec, int target) {
    for (auto v : vec) {
        if (v == target) return v;
    }
    return std::nullopt;
}

auto result = findValue({1, 2, 3}, 2);
if (result) {
    std::cout << "Found: " << *result << "\n";
}
```

### (2) 延迟初始化

`std::optional` 可以用来延迟对象构造，仅在需要时才赋值。

```cpp
std::optional<std::string> cache;
if (!cache) {
    cache = "Hello";  // 仅在需要时才构造
}
```

### (3) 配合结构化绑定

```cpp
std::optional<std::pair<int, int>> divide(int a, int b) {
    if (b == 0) return std::nullopt;
    return std::make_pair(a / b, a % b);
}

if (auto res = divide(10, 3)) {
    auto [q, r] = *res;
    std::cout << "Quotient: " << q << " Remainder: " << r << "\n";
}
```

## 4. 常用操作

### (0) 构造与析构

```cpp
// 默认构造：无值
std::optional<int> a;

// 从值构造
std::optional<int> b = 42;
std::optional<std::string> c{"hello"};

// 拷贝构造 / 赋值
std::optional<int> d = b;

// 移动构造 / 赋值
std::optional<std::string> e = std::move(c);

// 使用 nullopt 构造空对象
std::optional<int> f = std::nullopt;

// 析构：生命周期结束时自动析构其中的值（如果有值）
{
    std::optional<std::string> g{"RAII"};
} // g 的析构会自动调用 std::string 的析构函数
```

### (1) `value()` 与 `value_or()`

```cpp
std::optional<int> x = 10;
std::cout << x.value() << "\n";         // 获取值，若为空抛出 std::bad_optional_access
std::cout << x.value_or(0) << "\n";     // 若为空返回默认值
```

### (2) `emplace()` 与 `reset()`

```cpp
std::optional<std::string> name;
name.emplace("Alice");   // 原地构造值
std::cout << *name << "\n";
name.reset();            // 清空，变为无值
```

### (3) 布尔上下文与 `has_value()`

```cpp
std::optional<int> maybe;
if (maybe) {             // 等价于 maybe.has_value()
    std::cout << "有值\n";
} else {
    std::cout << "无值\n";
}
```

### (4) 访问操作符 `operator*` 与 `operator->`

```cpp
std::optional<std::string> msg = "Hello";
std::cout << (*msg).size() << "\n";     // 解引用访问
std::cout << msg->size() << "\n";       // 箭头访问
```

### (5) `swap()`

```cpp
std::optional<int> a = 1, b = 2;
a.swap(b);                             // 交换内容
std::cout << *a << " " << *b << "\n";  // 输出 2 1
```

### (6) 工厂函数 `std::make_optional` 与 `std::nullopt`

```cpp
auto a = std::make_optional<int>(42);   // 创建有值 optional
std::optional<int> b = std::nullopt;    // 创建空 optional
```

### (7) 常见 `value_or` 模式

```cpp
std::optional<std::string> maybe_name;
std::string name = maybe_name.value_or("Default");  // 若无值，使用默认值
```

### (8) C++23 单子操作（Monadic operations）

```cpp
std::optional<int> num = 5;

// and_then: 仅在有值时继续计算
auto squared = num.and_then([](int x) { return std::optional<int>(x * x); });

// transform: 值存在时映射为新值
auto str = num.transform([](int x) { return std::to_string(x); });

// or_else: 若无值则提供替代
auto val = num.or_else([] { return std::optional<int>(100); });
```

## 5. `std::optional` 的常见误用

### (1) 滥用在“必然有值”的场景

如果某个值在逻辑上 **必然存在**，使用 `optional` 反而多此一举，降低可读性。

### (2) 忘记检查是否有值

错误示例：

```cpp
std::optional<int> x;
std::cout << *x;  // 未检查直接解引用，未定义行为
```

正确示例：

```cpp
if (x) std::cout << *x;
```

### (3) 不当替代指针或容器

`std::optional` 适合表示“零或一”的情况，但如果可能有多个值，应该用 `std::vector` 或其他容器，而不是 `optional<std::vector<T>>` 滥用。

## 6. 与 Rust 的类比

- Rust 的 `Option<T>` 与 C++ 的 `std::optional<T>` 类似，都是“要么有值（Some），要么无值（None）”。
- 不同点：Rust 强制模式匹配，保证空值情况不会被遗漏，而 C++ 中 `optional` 需要程序员手动检查。
