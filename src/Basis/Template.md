# 模板 (Template)

**模板**是C++中实现**泛型编程**的基础工具，泛型编程即以一种独立于任何特定类型的方式编写代码。模板是创建泛型类或函数的蓝图或公式。库容器，比如迭代器和算法，都是泛型编程的例子，它们都使用了模板的概念。它允许我们编写与类型无关的代码（如类或函数），从而提高代码的重用性。编译器在编译时会根据用户提供的类型参数，从模板生成具体的类型或函数，这个过程称为**模板实例化（Template Instantiation）**。

## 一、模板的基本类型

C++主要有两种模板：

### 1. 函数模板 (Function Template)

用于创建可处理不同数据类型的函数。

**语法:**

```cpp
template <typename T>
T minimum(const T& lhs, const T& rhs) {
    return lhs < rhs ? lhs : rhs;
}

// 或使用 class 关键字，效果等同于 typename
template <class T>
T maximum(const T& lhs, const T& rhs) {
    return lhs > rhs ? lhs : rhs;
}
```

**使用:**

```cpp
int a = 10, b = 20;
int min_val = minimum(a, b);         // 编译器推导 T 为 int
double x = 3.14, y = 1.618;
double max_val = maximum<double>(x, y); // 显式指定 T 为 double
```

### 2. 类模板 (Class Template)

用于创建可处理不同数据类型的类，常用于实现容器（如 `std::vector`、`std::pair`）。

**语法:**

```cpp
template <typename T1, typename T2>
class Pair {
public:
    T1 first;
    T2 second;
    Pair(T1 f, T2 s) : first(f), second(s) {}
    // 成员函数也可以使用模板参数
    void print() const;
};

template <typename T1, typename T2>
void Pair<T1, T2>::print() const {
    // ... 实现细节
}
```

**使用:**

```cpp
Pair<int, double> p(10, 3.14); // 实例化 Pair<int, double> 类
```

## 二、模板参数 (Template Parameters)

模板参数可以是以下几种类型：

| 参数类型 | 描述 | 示例 |
| :--- | :--- | :--- |
| **类型参数** | 作为类型的占位符。使用 `typename` 或 `class` 关键字声明。 | `template <typename T>` |
| **非类型参数** | 模板参数可以是整型、枚举、指针、引用、`std::nullptr_t` 等编译期常量。 | `template <typename T, size_t N>` (如 `std::array`) |
| **模板的模板参数** | 参数本身是一个模板（较高级）。 | `template <typename T, template<typename> class Container>` |

**默认模板参数:**

模板参数可以拥有默认值（从C++11开始，函数模板也可以拥有默认模板参数）。

```cpp
template <typename T, size_t N = 10> // N 的默认值为 10
struct Array { T arr[N]; };
Array<int> a; // N 默认为 10
Array<double, 5> b; // N 显式指定为 5
```

## 三、模板特化 (Template Specialization)

模板特化允许为特定的类型参数提供一个**不同的**实现，以优化性能或处理通用模板无法处理的情况。

### 1. 完全特化 (Full Specialization)

为模板的所有参数提供具体的类型或值。

**语法:**

```cpp
// 原始函数模板
template <typename T>
T process(T value) { /* 通用实现 */ }

// int 类型的完全特化
template <> // 尖括号内为空
int process<int>(int value) {
    // 针对 int 类型的优化或特殊实现
    return value * 2;
}
```

### 2. 部分特化 (Partial Specialization)

只对模板的部分参数进行特化，或对参数的形式进行限制（例如，特化为指针类型）。**注意：只有类模板可以进行部分特化，函数模板不能。**

**语法 (类模板):**

```cpp
// 原始类模板
template <typename T1, typename T2>
class MyPair { /* 通用实现 */ };

// 部分特化：当第二个参数为 int 时
template <typename T1>
class MyPair<T1, int> { /* T2 固定为 int 的实现 */ };

// 部分特化：当两个参数都是指针时
template <typename T1, typename T2>
class MyPair<T1*, T2*> { /* 两个参数都是指针的实现 */ };
```

## 四、关键字 `typename`

`typename` 关键字有两个主要用途：

1. **在模板参数列表中**，指示一个模板参数是一个类型参数（等同于 `class`）。

    ```cpp
    template <typename T> // 或 template <class T>
    ```

2. **在模板定义内部**，用于显式告诉编译器一个**依赖于模板参数的嵌套名称**是一个类型。
    这是因为在编译时，编译器无法确定依赖名称是否是一个类型或静态成员。

    ```cpp
    template <typename T>
    void func(T value) {
        // 假设 T::InnerType 是 T 内部定义的类型
        typename T::InnerType *ptr; // 必须加 typename
    }
    ```

## 五、模板参数推导 (Argument Deduction)

### 1. 函数模板参数推导

调用函数模板时，编译器通常可以根据传递的实参类型自动推导出模板参数的类型。

```cpp
template <typename T> void print(T val) { /* ... */ }
print(42);   // 编译器推导 T 为 int
print("abc"); // 编译器推导 T 为 const char*
```

### 2. 类模板参数推导 (CTAD, C++17)

从 C++17 开始，类模板的实例化也可以进行类型推导，无需显式指定模板参数。

```cpp
// 假设 Pair 是一个类模板
// C++17 之前:
Pair<int, double> p1(1, 2.3);

// C++17 之后:
Pair p2(1, 2.3); // 编译器推导为 Pair<int, double>
```
