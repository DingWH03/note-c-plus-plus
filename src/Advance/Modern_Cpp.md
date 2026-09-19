# 现代 C++ 演进

C++ 从 2011 年起进入三年一版的快速演进节奏。理解各版本引入了什么，有助于判断手头的代码库能用哪些特性，也有助于理解为什么某些写法会成为「现代 C++」的推荐风格。

本节按版本梳理主要特性。具体用法见各专题章节，本节只做脉络梳理与速查。

## 一、版本时间线

| 标准 | 俗称 | 发布时间 | 核心主题 |
| :--- | :--- | :--- | :--- |
| C++98 | C++98 | 1998 | 第一个 ISO 标准，STL 正式纳入 |
| C++03 | C++03 | 2003 | 修订版，主要是缺陷修复 |
| **C++11** | **C++0x** | 2011 | **现代 C++ 起点**：移动语义、lambda、并发、智能指针 |
| C++14 | C++1y | 2014 | 对 C++11 的完善与放宽 |
| C++17 | C++1z | 2017 | 结构化绑定、`if constexpr`、`optional`/`variant`、并行算法 |
| **C++20** | **C++2a** | 2020 | **第二次大修订**：Concepts、Ranges、协程、模块、三路比较 |
| C++23 | C++2b | 2023 | `expected`、`print`、`mdspan`、Ranges 扩展 |
| C++26 | C++2c | 2026 | 反射、契约、并行 Ranges、`std::execution` |

## 二、C++11：现代 C++ 的起点

C++11 是语言历史上最大的一次修订，它改变了 C++ 的编程范式。

### 语言特性

| 特性 | 说明 |
| :--- | :--- |
| **`auto`** | 类型自动推导 |
| **范围 `for`** | `for (auto& x : container)` |
| **右值引用 `&&`** | 移动语义的基础 |
| **移动构造/赋值** | 避免不必要的深拷贝 |
| **Lambda 表达式** | 就地定义匿名函数对象 |
| **`nullptr`** | 类型安全的空指针 |
| **`enum class`** | 强类型枚举，不隐式转换 |
| **`override` / `final`** | 显式标注虚函数覆盖 |
| **`= default` / `= delete`** | 显式控制特殊成员函数 |
| **变参模板** | `template<typename... Args>` |
| **`constexpr`** | 编译期求值 |
| **`noexcept`** | 声明不抛异常 |
| **`static_assert`** | 编译期断言 |
| **委托构造 / 继承构造** | 构造函数复用 |
| **初始化列表 `{}`** | 统一的初始化语法 |
| **`using` 别名** | `using IntVec = std::vector<int>;` |

### 标准库特性

| 特性 | 说明 |
| :--- | :--- |
| **智能指针** | `unique_ptr`、`shared_ptr`、`weak_ptr` |
| **`std::thread`** | 线程支持 |
| **`std::mutex`** | 互斥量 |
| **`std::atomic`** | 原子操作 |
| **`std::chrono`** | 时间库 |
| **`std::random`** | 随机数库 |
| **`std::unordered_*`** | 哈希容器 |
| **`std::array`** | 固定大小数组 |
| **`std::tuple`** | 异质容器 |
| **`std::function`** | 通用函数包装器 |

```c++
// C++11 的典型现代写法
auto v = std::vector<int>{1, 2, 3, 4, 5};

auto sum = 0;
for (const auto& x : v)
    sum += x;

auto even_count = std::count_if(v.begin(), v.end(),
                                [](int x) { return x % 2 == 0; });

auto ptr = std::make_unique<int>(42);
```

## 三、C++14：完善与放宽

C++14 是一个小版本，主要是对 C++11 的补全。

| 特性 | 说明 |
| :--- | :--- |
| **泛型 lambda** | `[](auto x) { return x; }` |
| **初始化捕获** | `[x = std::move(y)] {}` |
| **`constexpr` 放宽** | 允许循环、分支、多语句 |
| **函数返回类型推导** | `auto f() { return 42; }` |
| **变量模板** | `template<typename T> constexpr T pi = ...;` |
| **`std::make_unique`** | C++11 遗漏的补全 |

```c++
// 泛型 lambda
auto print = [](const auto& x) { std::cout << x << '\n'; };

// 初始化捕获（移动捕获）
auto ptr = std::make_unique<int>(42);
auto f = [p = std::move(ptr)] { return *p; };
```

## 四、C++17：实用主义

C++17 引入了大量实用特性，是日常开发中最常用的一版。

### 语言特性

| 特性 | 说明 |
| :--- | :--- |
| **结构化绑定** | `auto [a, b] = pair;` |
| **`if` 初始化语句** | `if (auto it = m.find(k); it != m.end())` |
| **`if constexpr`** | 编译期分支，替代 SFINAE |
| **折叠表达式** | `(args + ...)` |
| **内联变量** | 头文件中定义变量 |
| **`[[nodiscard]]`** | 警告忽略返回值 |
| **`[[maybe_unused]]`** | 抑制未使用警告 |
| **类模板参数推导** | `std::pair p(1, 2);` |
| **`*this` 捕获** | `[*this] {}` |
| **`constexpr` lambda** | lambda 可用于编译期 |
| **嵌套命名空间** | `namespace a::b::c {}` |

### 标准库特性

| 特性 | 说明 |
| :--- | :--- |
| **`std::optional`** | 可选值 |
| **`std::variant`** | 类型安全联合体 |
| **`std::any`** | 任意类型容器 |
| **`std::string_view`** | 字符串视图 |
| **`std::filesystem`** | 文件系统操作 |
| **`std::byte`** | 字节类型 |
| **并行算法** | `std::execution::par` |
| **`std::scoped_lock`** | 多互斥量锁定 |
| **`std::apply`** | 展开 tuple 调用函数 |

```c++
// 结构化绑定 + if 初始化
if (auto [it, inserted] = map.try_emplace(key, value); inserted)
    std::cout << "插入成功\n";

// if constexpr
template <typename T>
auto stringify(T value)
{
    if constexpr (std::is_arithmetic_v<T>)
        return std::to_string(value);
    else
        return std::string(value);
}

// optional
std::optional<int> find_value(const std::string& key);
if (auto v = find_value("key"))
    std::cout << *v << '\n';
```

## 五、C++20：第二次大修订

C++20 引入了四大特性（Concepts、Ranges、协程、模块），是继 C++11 之后最重要的版本。

### 语言特性

| 特性 | 说明 |
| :--- | :--- |
| **Concepts** | 对模板参数的显式约束 |
| **Ranges** | 可组合的范围算法与视图 |
| **协程** | `co_await` / `co_yield` / `co_return` |
| **模块** | `import` 取代 `#include` |
| **三路比较 `<=>`** | 一行定义全部比较运算符 |
| **指定初始化** | `Point{.x = 1, .y = 2}` |
| **`consteval`** | 强制编译期求值 |
| **`constinit`** | 强制编译期初始化 |
| **`using enum`** | 引入枚举成员到作用域 |
| **lambda 模板参数** | `[]<typename T>(T x) {}` |
| **`[[likely]]` / `[[unlikely]]`** | 分支预测提示 |

### 标准库特性

| 特性 | 说明 |
| :--- | :--- |
| **`std::span`** | 连续序列的非拥有视图 |
| **`std::format`** | 格式化输出 |
| **`std::jthread`** | 自动 join 的线程 |
| **`std::latch` / `barrier`** | 线程同步原语 |
| **`std::semaphore`** | 信号量 |
| **`std::bit_cast`** | 类型安全的位转换 |
| **`std::midpoint`** | 中点计算（避免溢出） |
| **`std::numbers`** | 数学常量 |
| **`contains`** | 关联容器的成员函数 |
| **`std::source_location`** | 源码位置信息 |

```c++
// Concepts
template <std::integral T>
T add(T a, T b) { return a + b; }

// Ranges
auto result = data
    | std::views::filter([](int x) { return x > 0; })
    | std::views::transform([](int x) { return x * 2; })
    | std::views::take(5);

// 三路比较
struct Point {
    int x, y;
    auto operator<=>(const Point&) const = default;
};

// span
void process(std::span<const int> data);

// format
std::string s = std::format("x = {}, y = {}", 1, 2);
```

## 六、C++23：稳步推进

| 特性 | 说明 |
| :--- | :--- |
| **`std::expected`** | 错误值或结果值 |
| **`std::print`** | 直接打印，取代 `cout` |
| **`std::mdspan`** | 多维数组视图 |
| **`std::flat_map` / `flat_set`** | 基于有序 vector 的容器 |
| **Ranges 扩展** | `zip`、`enumerate`、`chunk`、`slide`、`stride` |
| **`std::ranges::to`** | 视图转容器 |
| **`std::stacktrace`** | 调用栈信息 |
| **`if consteval`** | 区分编译期与运行期 |
| **多维下标 `[]`** | `arr[i, j]` 语法 |
| **`std::byteswap`** | 字节序转换 |
| **`import std;`** | 标准库模块 |

```c++
// expected
std::expected<int, std::string> parse(const std::string& s)
{
    try {
        return std::stoi(s);
    } catch (...) {
        return std::unexpected("解析失败");
    }
}

// print
std::print("Hello, {}!\n", "World");

// Ranges 扩展
for (auto [i, x] : std::views::enumerate(data))
    std::cout << i << ": " << x << '\n';
```

## 七、C++26：即将到来

C++26 已于 2026 年发布，主要特性包括：

| 特性 | 说明 |
| :--- | :--- |
| **反射（静态）** | 编译期检查与生成代码 |
| **契约** | `pre` / `post` / `contract_assert` |
| **并行 Ranges** | `ranges::` 算法接受执行策略 |
| **`std::execution`** | 发送者/接收者异步模型 |
| **`std::hive`** | 新容器类型 |
| **`std::inplace_vector`** | 固定容量、栈上存储的动态数组 |
| **`std::linalg`** | 线性代数算法 |
| **`std::simd`** | 数据并行类型 |
| **`std::rcu` / 危险指针** | 安全内存回收 |
| **`=)` 占位符** | 结构化绑定的未使用标记 |

## 八、如何选择特性版本

| 场景 | 建议 |
| :--- | :--- |
| 新项目、工具链可控 | 直接用 C++20 或 C++23 |
| 需要广泛兼容 | C++17 是当前最稳妥的选择 |
| 嵌入式、老编译器 | 可能只能用到 C++11/14 |
| 学习 | 从 C++11 基础特性入手，逐步掌握 C++17/20 |

**判断编译器支持情况**：

```c++
#include <version>   // C++20

#if __cpp_lib_optional >= 201603L
    // 支持 std::optional
#endif

#if __cpp_concepts >= 201907L
    // 支持 Concepts
#endif
```

也可以用 `__cplusplus` 宏判断语言版本：

```c++
#if __cplusplus >= 202002L
    // C++20 或更高
#endif
```

## 九、注意事项

- **不要为了用而用**。新特性应服务于可读性和正确性，而非炫技。
- **注意编译器支持**。不同编译器对同一特性的支持进度不同，尤其是 C++20/23 的部分特性。
- **模块尚未普及**。虽然 C++20 就引入了模块，但工具链支持仍在完善中，实际项目大多仍用头文件。
- **协程需要库支持**。标准库只提供了语言机制，实际使用需要第三方库（如 cppcoro）或自行封装。
- **ABI 兼容性**。升级标准可能影响二进制兼容，尤其是涉及标准库类型的接口。
- **`constexpr` 能力持续增强**。C++14 允许循环，C++17 允许 lambda，C++20 允许 `try`（C++26 起允许抛异常），编译器支持度需确认。

## 十、相关章节

- [C++ 新特性](../Advance/Object/CPP_Modern_Features.md)：移动语义、右值引用、`constexpr`、`noexcept` 的详细用法
- [Ranges](../STL/Ranges.md)：Ranges 视图与管道语法
- [Iterators](../STL/Iterators.md)：C++20 迭代器概念
- [Algorithms](../STL/Algorithms.md)：C++20 Ranges 算法与 C++26 并行范围算法
- [异常处理](../Advance/Exception.md)：`noexcept` 与异常安全
