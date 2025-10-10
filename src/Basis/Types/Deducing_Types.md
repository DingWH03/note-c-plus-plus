# 自动类型推导

在 C++ 中，自动类型推导（type inference）允许编译器根据初始化表达式推断变量的类型，从而减少显式指定类型的需求。C++11 引入了 `auto` 关键字，它使得变量声明更加简洁和灵活，同时也有助于避免类型错误。

## `auto` 关键字的基本使用

`auto` 关键字让编译器根据初始化表达式推导出变量的类型。在声明变量时，使用 `auto` 代替类型名称，编译器将自动推断类型。

```cpp
#include <iostream>

int main() {
    auto x = 42;  // 自动推导类型为 int
    auto y = 3.14;  // 自动推导类型为 double

    std::cout << "x: " << x << ", y: " << y << std::endl;

    return 0;
}
```

在这个例子中，`auto` 自动推导出 `x` 的类型为 `int`，`y` 的类型为 `double`，根据其初始化值。无需显式地指定类型。

输出：

```text
x: 42, y: 3.14
```

## `auto` 与容器

`auto` 在处理容器时非常有用，尤其是当容器类型复杂且冗长时，使用 `auto` 可以简化代码并提高可读性。C++11 引入了范围-based `for` 循环，配合 `auto` 使用可以轻松地遍历容器。

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 使用 auto 遍历容器
    for (auto& v : vec) {
        std::cout << v << " ";
    }

    std::cout << std::endl;

    return 0;
}
```

在此示例中，`auto` 用来自动推导容器元素的类型。由于我们希望修改容器中的元素，所以使用了 `auto&` 来获取元素的引用。

输出：

```text
1 2 3 4 5
```

## `auto` 和指针

在使用指针时，`auto` 也可以自动推导出指针的类型。注意，`auto` 仅推导出变量本身的类型，而不推导出指针指向的类型。

```cpp
#include <iostream>

int main() {
    int num = 100;
    int* ptr = &num;

    // 使用 auto 推导指针类型
    auto p = ptr;  // p 将是 int* 类型

    std::cout << "p points to: " << *p << std::endl;

    return 0;
}
```

在这个例子中，`auto` 推导出变量 `p` 的类型为 `int*`，与原始指针 `ptr` 类型相同。

输出：

```text
p points to: 100
```

## `auto` 与函数返回类型

`auto` 不仅可以用于变量声明，还可以用于函数的返回类型。当返回类型较为复杂或需要根据返回的表达式推导时，`auto` 可以简化代码。

```cpp
#include <iostream>
#include <vector>

auto get_sum(const std::vector<int>& vec) {
    int sum = 0;
    for (auto v : vec) {
        sum += v;
    }
    return sum;  // 返回类型由 auto 自动推导
}

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    std::cout << "Sum: " << get_sum(vec) << std::endl;
    return 0;
}
```

在这个示例中，`auto` 用于函数 `get_sum` 的返回类型，编译器根据 `sum` 变量的类型推导出返回值的类型。

输出：

```text
Sum: 15
```

## `auto` 与模板

`auto` 也可以与模板结合使用，尤其是在模板函数中，允许推导类型从而使得函数更加灵活。使用 `auto` 可以避免显式指定模板参数，从而简化模板代码。

```cpp
#include <iostream>

template <typename T>
auto add(T a, T b) {
    return a + b;  // 返回类型自动推导
}

int main() {
    std::cout << add(1, 2) << std::endl;    // 输出 3
    std::cout << add(1.5, 2.5) << std::endl; // 输出 4.0

    return 0;
}
```

在这个示例中，`auto` 自动推导 `add` 函数的返回类型，无论是整型还是浮点型，都能正确处理。

输出：

```text
3
4
```

## 注意

1. `auto` 不能用于函数参数类型
   `auto` 只能用于变量声明和返回类型，不能用于函数的参数类型。例如，以下代码是错误的：

   ```cpp
   auto add(auto a, auto b) { // 错误，auto 不能用于函数参数类型
       return a + b;
   }
   ```

2. `auto` 与引用、常量结合
   当使用 `auto` 时，变量的类型是根据初始化表达式的类型来推导的。如果需要引用或常量，应该明确使用 `auto&` 或 `const auto&` 来推导引用类型或常量类型。

   ```cpp
   const auto& ref = some_variable; // 推导为 const 引用
   ```

3. 类型推导的限制
   编译器根据初始化表达式推导类型，但不能从不完整的类型推导。因此，必须确保变量在声明时能够通过表达式推导出类型。
