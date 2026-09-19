# Lambda 表达式

**Lambda 表达式**（C++11 引入）用于在调用点就地定义一个匿名函数对象。它让「把函数作为参数传递」变得极为简洁，是标准库算法、回调、异步任务中最常用的语法之一。

Lambda 的本质是编译器生成的一个**闭包类型**（closure type）——一个重载了 `operator()` 的匿名类。因此它和手写的函数对象在行为上完全等价，只是写法更紧凑。

## 一、基本语法

```c++
[capture](parameters) -> return_type { body }
```

| 部分 | 说明 |
| :--- | :--- |
| `[capture]` | 捕获列表，决定如何访问外部变量（不可省略） |
| `(parameters)` | 参数列表，无参数时可省略 |
| `-> return_type` | 返回类型，可省略（编译器自动推导） |
| `{ body }` | 函数体 |

最简单的形式：

```c++
auto add = [](int a, int b) { return a + b; };
std::cout << add(3, 4);   // 7
```

返回类型通常可以省略。但如果函数体中有多个 `return` 语句且类型不一致，就必须显式指定：

```c++
auto f = [](int x) -> double {
    if (x > 0) return x * 1.0;
    return 0.0;
};
```

## 二、捕获列表

捕获列表决定 lambda 内部能访问哪些外部变量，以及访问方式。

### 1. 按值捕获与按引用捕获

```c++
int a = 10, b = 20;

auto byValue = [a] { return a; };        // 拷贝 a
auto byRef   = [&b] { return b; };       // 引用 b

a = 100;
b = 200;

std::cout << byValue();   // 10（捕获时的副本）
std::cout << byRef();     // 200（引用的是原变量）
```

| 捕获方式 | 写法 | 特点 |
| :--- | :--- | :--- |
| 按值 | `[x]` | 拷贝一份，lambda 内默认只读 |
| 按引用 | `[&x]` | 引用原变量，lambda 内可修改 |
| 全部按值 | `[=]` | 隐式按值捕获所有用到的变量 |
| 全部按引用 | `[&]` | 隐式按引用捕获所有用到的变量 |
| 混合 | `[=, &x]` | 默认按值，`x` 按引用 |
| 混合 | `[&, x]` | 默认按引用，`x` 按值 |

### 2. mutable

按值捕获的变量在 lambda 内默认是 `const` 的，要修改副本必须加 `mutable`：

```c++
int count = 0;

auto counter = [count]() mutable {
    return ++count;   // 修改的是副本
};

std::cout << counter();   // 1
std::cout << counter();   // 2
std::cout << count;       // 0（原变量未变）
```

> [!WARNING]
> **按引用捕获的生命周期陷阱**
>
> lambda 不会延长被引用对象的生命周期。如果 lambda 的生命周期超过了被引用变量，就会产生悬垂引用：
>
> ```c++
> auto makeLambda()
> {
>     int local = 42;
>     return [&local] { return local; };   // 错误：local 已销毁
> }
>
> auto f = makeLambda();
> f();   // 未定义行为
> ```
>
> 需要返回 lambda 时，应改为按值捕获。

### 3. 初始化捕获（C++14）

初始化捕获（init-capture）允许在捕获时执行表达式，并给捕获的变量起新名字：

```c++
// 移动捕获：把 move-only 对象移入 lambda
auto ptr = std::make_unique<int>(42);
auto f = [p = std::move(ptr)] { return *p; };   // ptr 被移入 lambda

// 捕获时计算
int x = 10;
auto g = [y = x * 2] { return y; };   // y = 20
```

这是**唯一能把 move-only 类型（如 `unique_ptr`）放进 lambda** 的方式。

### 4. 捕获 this

在成员函数中，`[this]` 捕获的是指针，`[*this]`（C++17）捕获的是对象副本：

```c++
class Widget
{
    int value_ = 42;

public:
    auto byPointer()
    {
        return [this] { return value_; };   // 引用当前对象
    }

    auto byCopy()
    {
        return [*this] { return value_; };  // 拷贝整个对象（C++17）
    }
};
```

`[this]` 的 lambda 在对象销毁后调用会悬垂；`[*this]` 则安全。

## 三、泛型 Lambda

C++14 起，参数可以用 `auto`，此时 lambda 的 `operator()` 变成一个模板：

```c++
auto print = [](auto x) { std::cout << x << '\n'; };

print(42);        // int
print("hello");   // const char*
print(3.14);      // double
```

C++20 起可以显式写模板参数列表，从而使用 Concepts 约束：

```c++
auto add = []<typename T>(T a, T b) { return a + b; };

// 带约束
auto integralAdd = []<std::integral T>(T a, T b) { return a + b; };
```

显式模板参数还能解决一个常见问题——对 `vector` 的 `size()` 求值：

```c++
// C++20 前：需要额外处理
auto before = [](auto&& v) { return v.size(); };

// C++20：可以正确推导
auto after = []<typename T>(std::vector<T> const& v) { return v.size(); };
```

## 四、其它特性

### 1. constexpr Lambda（C++17）

如果 lambda 满足 `constexpr` 函数的要求，`operator()` 自动是 `constexpr`：

```c++
constexpr auto square = [](int x) { return x * x; };
static_assert(square(5) == 25);
```

### 2. 无捕获 Lambda 转换为函数指针

无捕获的 lambda 可以隐式转换为函数指针，因此能传给 C 风格 API：

```c++
void c_function(int (*callback)(int));

c_function([](int x) { return x * 2; });   // OK：无捕获
```

有捕获的 lambda **不能**这样转换，因为需要携带状态。

### 3. 立即调用

lambda 定义后可以立刻调用，常用于初始化复杂常量：

```c++
const int value = [] {
    // 复杂的计算过程
    return 42;
}();
```

### 4. 递归 Lambda

lambda 无法直接调用自己（因为名字还没定义完）。常见做法是把自己作为参数传入：

```c++
auto factorial = [](auto self, int n) -> int {
    return n <= 1 ? 1 : n * self(self, n - 1);
};

std::cout << factorial(factorial, 5);   // 120
```

C++23 起可以用 `this auto self` 显式对象参数，写法更自然：

```c++
auto factorial = [](this auto self, int n) -> int {
    return n <= 1 ? 1 : n * self(n - 1);
};
```

## 五、与函数对象的关系

lambda 在编译期被展开成一个匿名类：

```c++
int threshold = 10;
auto pred = [threshold](int x) { return x > threshold; };

// 大致等价于：
class __Lambda
{
    int threshold_;
public:
    __Lambda(int t) : threshold_(t) {}
    bool operator()(int x) const { return x > threshold_; }
};
```

这解释了 lambda 的几个特性：

- **每个 lambda 都有独一无二的类型**，因此不能用 `decltype` 之外的方式声明，通常用 `auto` 接收。
- 两个写法相同的 lambda 也是**不同类型**。
- 需要存储 lambda 时，如果类型必须统一，可以用 `std::function`（有类型擦除开销）。

## 六、常见用法

配合标准库算法是最典型的场景：

```c++
std::vector<int> v{5, 2, 8, 1, 9};

// 排序：按绝对值降序
std::sort(v.begin(), v.end(), [](int a, int b) {
    return std::abs(a) > std::abs(b);
});

// 查找：第一个大于 5 的元素
auto it = std::find_if(v.begin(), v.end(), [](int x) { return x > 5; });

// 统计：偶数个数
int evens = std::count_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; });

// 变换：每个元素平方
std::transform(v.begin(), v.end(), v.begin(), [](int x) { return x * x; });

// 移除：删除所有小于 3 的元素
v.erase(std::remove_if(v.begin(), v.end(), [](int x) { return x < 3; }), v.end());
```

## 七、注意事项

- **按引用捕获有生命周期风险**。lambda 不会延长被引用对象的生命周期，返回 lambda 时应按值捕获。
- **`[=]` 捕获 `this` 是陷阱**。在成员函数中，`[=]` 捕获的是 `this` 指针而非对象副本，对象销毁后调用会悬垂。C++20 起这种隐式捕获已被弃用，应显式写 `[this]` 或 `[*this]`。
- **`mutable` 只影响副本**。加了 `mutable` 后修改的是 lambda 内部的副本，原变量不变。
- **每个 lambda 类型唯一**。不要试图写出 lambda 的类型，用 `auto` 或 `std::function`。
- **`std::function` 有开销**。它涉及类型擦除和可能的堆分配，性能敏感处应直接用 `auto` 或模板参数。
- **捕获列表不能捕获类成员**。`[x]` 中的 `x` 必须是变量，类成员需要用 `[x = x]` 或 `[this]`。
- **无捕获 lambda 才能转函数指针**。有捕获的 lambda 携带状态，无法转换。

## 八、相关章节

- [函数](./Functions.md)：函数对象与函数指针
- [模板](./Template.md)：泛型 lambda 与模板的关系
- [Utility](../STL/Utility.md)：`std::function` 与 `std::bind`
- [Algorithms](../STL/Algorithms.md)：算法中 lambda 的典型用法
- [多线程与并发](../Advance/Concurrency.md)：lambda 作为线程函数
