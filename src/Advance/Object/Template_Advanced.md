# 类模板与函数模板高级应用

基础语法在 [模板](../../Basis/Template.md) 里已经讲过：怎么写函数模板、类模板，怎么做特化。这一章讲的是模板真正发挥威力的部分——**编译期计算、约束与泛型设计**。

这些技术让 C++ 的泛型能力远超「把类型当参数」的层面。

## 一、可变参数模板

C++11 引入的**参数包**让模板能接受任意数量的类型或值：

```c++
template<class... Args>
void print(Args... args)
{
    (std::cout << ... << args) << '\n';   // C++17 折叠表达式
}

print(1, "hello", 3.14, 'c');
```

`Args...` 是类型参数包，`args...` 是函数参数包。展开它们有几种方式。

### 1. 折叠表达式（C++17）

最简洁的展开方式：

```c++
template<class... Args>
auto sum(Args... args)
{
    return (args + ...);           // 一元右折叠：a + (b + (c + ...))
    // return (... + args);        // 一元左折叠：((a + b) + c) + ...
    // return (0 + ... + args);    // 二元折叠，带初始值
}
```

四种形式：

| 形式 | 展开结果 |
| :--- | :--- |
| `(... op pack)` | 左折叠，`((a op b) op c)` |
| `(pack op ...)` | 右折叠，`(a op (b op c))` |
| `(init op ... op pack)` | 带初值的左折叠 |
| `(pack op ... op init)` | 带初值的右折叠 |

### 2. 递归展开（C++11）

C++17 之前只能用递归：

```c++
// 递归终止
void print() {}

template<class T, class... Rest>
void print(const T& first, const Rest&... rest)
{
    std::cout << first << ' ';
    print(rest...);   // 递归展开
}
```

### 3. 逗号表达式展开

利用初始化列表保证求值顺序：

```c++
template<class... Args>
void print(Args... args)
{
    int dummy[] = { (std::cout << args << ' ', 0)... };
    (void)dummy;
}
```

### 4. sizeof... 获取数量

```c++
template<class... Args>
constexpr std::size_t count(Args...)
{
    return sizeof...(Args);
}

count(1, 2, 3);   // 3
```

## 二、SFINAE

**SFINAE**（Substitution Failure Is Not An Error，替换失败并非错误）是 C++ 模板的核心机制之一。

它的含义是：当编译器用具体类型替换模板参数时，如果产生的代码无效，**不会立即报错**，而是把这个候选从重载集中移除，继续尝试其他候选。

```c++
// 只有当 T 有 size() 成员时，这个重载才有效
template<class T>
auto getSize(const T& container) -> decltype(container.size())
{
    return container.size();
}

// 备选：用于没有 size() 的类型
template<class T>
std::size_t getSize(...)
{
    return 0;
}

std::vector<int> v{1, 2, 3};
getSize(v);      // 调用第一个
getSize(42);     // 第一个替换失败（int 没有 size()），调用第二个
```

### enable_if

`std::enable_if` 是最经典的 SFINAE 工具，用于按条件启用重载：

```c++
#include <type_traits>

// 只对整数类型启用
template<class T>
std::enable_if_t<std::is_integral_v<T>, T>
half(T value)
{
    return value / 2;
}

// 只对浮点类型启用
template<class T>
std::enable_if_t<std::is_floating_point_v<T>, T>
half(T value)
{
    return value / 2.0;
}

half(10);     // 调用整数版本
half(3.14);   // 调用浮点版本
```

`enable_if_t<cond, T>` 在 `cond` 为真时是 `T`，为假时替换失败，从而移除该候选。

### void_t 检测成员

`std::void_t` 可以把任意类型映射为 `void`，常用于检测成员是否存在：

```c++
template<class T, class = void>
struct has_serialize : std::false_type {};

template<class T>
struct has_serialize<T, std::void_t<decltype(std::declval<T>().serialize())>>
    : std::true_type {};

static_assert(has_serialize<MyClass>::value);
```

如果 `T().serialize()` 无效，特化版本替换失败，回退到主模板（`false_type`）。

## 三、Concepts（C++20）

SFINAE 能工作，但错误信息极其难读。C++20 的 **Concepts** 提供了更清晰的替代方案。

### 1. 定义与使用

```c++
#include <concepts>

// 定义概念
template<class T>
concept Numeric = std::is_arithmetic_v<T>;

// 约束模板参数
template<Numeric T>
T square(T x) { return x * x; }

// 也可以用在 requires 子句中
template<class T>
requires Numeric<T>
T cube(T x) { return x * x * x; }
```

违反约束时，编译器会直接告诉你「`T` 不满足 `Numeric`」，而不是一堆模板实例化错误。

### 2. requires 表达式

用于描述类型需要支持哪些操作：

```c++
template<class T>
concept Container = requires(T c, const T& cc)
{
    { c.begin() } -> std::same_as<typename T::iterator>;
    { c.end() }   -> std::same_as<typename T::iterator>;
    { cc.size() } -> std::convertible_to<std::size_t>;
    typename T::value_type;
};

template<Container C>
void process(const C& c) { /* ... */ }
```

`requires` 块里可以写四类要求：

| 形式 | 含义 |
| :--- | :--- |
| `expr;` | 表达式必须合法 |
| `{ expr } -> Concept;` | 表达式合法且返回值满足概念 |
| `typename T::type;` | 类型必须存在 |
| `requires cond;` | 嵌套的进一步约束 |

### 3. 标准库概念

标准库提供了一批常用概念：

| 概念 | 含义 |
| :--- | :--- |
| `std::integral` | 整数类型 |
| `std::floating_point` | 浮点类型 |
| `std::same_as<T, U>` | 类型相同 |
| `std::convertible_to<From, To>` | 可转换 |
| `std::derived_from<D, B>` | 继承关系 |
| `std::invocable<F, Args...>` | 可调用 |
| `std::copyable` / `std::movable` | 可拷贝 / 可移动 |
| `std::ranges::range` | 是范围 |

### 4. 简写形式

概念可以直接用在参数位置：

```c++
void process(std::integral auto x);        // 等价于 template<std::integral T>
void handle(std::ranges::range auto&& r);
```

## 四、模板与继承的结合

### 1. CRTP：静态多态

**CRTP**（Curiously Recurring Template Pattern，奇异递归模板模式）让基类知道派生类的类型：

```c++
template<class Derived>
class Shape
{
public:
    void draw() const
    {
        // 转换为派生类，调用其实现
        static_cast<const Derived*>(this)->drawImpl();
    }
};

class Circle : public Shape<Circle>
{
public:
    void drawImpl() const { std::cout << "画圆\n"; }
};

class Square : public Shape<Square>
{
public:
    void drawImpl() const { std::cout << "画方\n"; }
};

template<class T>
void render(const Shape<T>& s) { s.draw(); }
```

**好处**：`draw()` 在编译期绑定，可以内联，没有虚表开销。

**代价**：不能通过基类指针统一存储不同派生类对象——`Shape<Circle>` 和 `Shape<Square>` 是完全不同的类型。

CRTP 的典型应用：

- 静态多态（替代虚函数）
- 给类批量添加操作符（如 `operator!=` 由 `operator==` 推导）
- 计数器（统计每个派生类实例化次数）

### 2. 策略模式

用模板参数注入行为：

```c++
template<class ComparePolicy>
class Sorter
{
public:
    template<class Container>
    void sort(Container& c)
    {
        ComparePolicy cmp;
        std::sort(c.begin(), c.end(), cmp);
    }
};

struct Ascending
{
    template<class T>
    bool operator()(const T& a, const T& b) const { return a < b; }
};

struct Descending
{
    template<class T>
    bool operator()(const T& a, const T& b) const { return a > b; }
};

Sorter<Ascending> asc;
Sorter<Descending> desc;
```

比运行时多态更快（可内联），但同样牺牲了运行期切换的能力。

### 3. 混入（Mixin）

用可变参数模板叠加多个功能：

```c++
template<class... Mixins>
class Combined : public Mixins...
{
public:
    Combined(const Mixins&... mixins) : Mixins(mixins)... {}
};

struct Loggable
{
    void log() const { std::cout << "logging\n"; }
};

struct Serializable
{
    void serialize() const { std::cout << "serializing\n"; }
};

Combined<Loggable, Serializable> obj{Loggable{}, Serializable{}};
obj.log();
obj.serialize();
```

## 五、模板元编程

模板在编译期就能做计算。经典的例子是编译期阶乘：

```c++
template<unsigned N>
struct Factorial
{
    static constexpr unsigned value = N * Factorial<N - 1>::value;
};

template<>
struct Factorial<0>
{
    static constexpr unsigned value = 1;
};

static_assert(Factorial<5>::value == 120);
```

C++11 之后，`constexpr` 函数更易读，多数场景不再需要这种写法：

```c++
constexpr unsigned factorial(unsigned n)
{
    return n <= 1 ? 1 : n * factorial(n - 1);
}

static_assert(factorial(5) == 120);
```

### 类型萃取

`<type_traits>` 提供了大量编译期类型操作：

```c++
template<class T>
void process(T value)
{
    if constexpr (std::is_pointer_v<T>)
    {
        // T 是指针
        if (value) use(*value);
    }
    else
    {
        use(value);
    }
}
```

`if constexpr`（C++17）让编译期分支写起来像普通 `if`，比 SFINAE 清晰得多。

### 类型列表操作

```c++
template<class... Ts>
struct TypeList {};

// 获取第一个类型
template<class List>
struct Front;

template<class Head, class... Tail>
struct Front<TypeList<Head, Tail...>>
{
    using type = Head;
};

using First = Front<TypeList<int, double, char>>::type;   // int
```

这类技术是很多库（如 Boost.MPL、range-v3）的基础，日常开发中很少需要手写。

## 六、编译期成本

模板的代价是**编译时间**。几个注意点：

**减少实例化次数**。同一个模板用 10 种类型实例化，就生成 10 份代码。如果这些类型的行为相同，考虑用类型擦除（如 `std::function`）合并。

**避免头文件膨胀**。模板定义必须在头文件里，大量模板会让每个包含它的 `.cpp` 都变慢。可以用显式实例化减少重复：

```c++
// 在 .cpp 中显式实例化
template class MyTemplate<int>;
template class MyTemplate<double>;
```

**注意错误信息长度**。深层嵌套的模板错误可能几百行。Concepts 能显著改善这一点。

**外部模板**（C++11）可以抑制重复实例化：

```c++
extern template class MyTemplate<int>;   // 告诉编译器别在这里实例化
```

## 七、常见误区

**「模板只能做泛型容器」** —— 模板能做编译期计算、约束检查、代码生成，远不止容器。

**「SFINAE 已经过时了」** —— 在 C++20 之前它是唯一的手段，大量现有代码仍在用。理解它有助于读懂老代码和标准库实现。

**「Concepts 只是更好看的 enable_if」** —— 除了语法糖，Concepts 还支持子概念、约束的合取与析取、以及重载决议中的偏序规则。

**「CRTP 比虚函数总是更快」** —— 少了虚表间接调用，但代码膨胀更严重。如果类型种类很多，指令缓存反而可能成为瓶颈。

**「模板元编程应该尽量用」** —— 编译期计算有编译期成本。如果运行期开销可以接受，普通代码更易维护。

**「`if constexpr` 和普通 `if` 一样」** —— 不同。`if constexpr` 在编译期选择分支，未选中的分支**不会被实例化**，因此可以包含对当前类型无效的代码。

## 八、相关章节

- [模板](../../Basis/Template.md)：模板基础语法
- [继承](./Inheritance.md)：运行时多态
- [虚函数与多态](./Virtual_Function.md)：虚函数与 CRTP 的对比
- [C++ 新特性](./CPP_Modern_Features.md)：`constexpr` 与编译期计算
- [现代 C++ 演进](../Modern_Cpp.md)：Concepts 在标准中的位置
