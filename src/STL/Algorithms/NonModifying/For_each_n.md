# std::for_each_n

`std::for_each_n` 对范围起始处的前 \\( N \\) 个元素依次调用一元函数对象 `f`，并返回指向第 \\( N \\) 个元素之后位置的迭代器。

它是 `for_each` 的「限定数量」版本，适用于只处理序列开头一部分元素的场景。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class Size, class UnaryFunction>
constexpr InputIt for_each_n(InputIt first, Size n, UnaryFunction f);
// C++11~C++17 返回 void，C++17 起返回 InputIt
```

- **迭代器要求**：`InputIterator`。
- **复杂度**：恰好 \\( n \\) 次函数调用。
- **返回值**：C++17 起返回 `first + n`，即最后一个被处理元素的下一个位置。C++11/14 中返回 `void`。

`ranges::for_each_n` 返回 `ranges::in_fun_result`，同时携带末尾迭代器和函数对象。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    // 只处理前 3 个元素
    auto it = std::for_each_n(v.begin(), 3, [](int x) {
        std::cout << x << ' ';   // 输出: 1 2 3
    });
    std::cout << '\n';

    std::cout << *it;   // 输出: 4（第 3 个元素之后的位置）
}
```

### (2) 谓词与投影

和 `for_each` 一样，`for_each_n` 接受一元可调用对象，没有谓词概念。`ranges::for_each_n` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}, {"Carol", 35}};

std::ranges::for_each_n(people.begin(), 2,
                        [](int age) { std::cout << age << ' '; },
                        &Person::age);   // 输出: 30 25
```

### (3) 执行策略

与 `for_each` 一样，`for_each_n` 也不要求元素类型可平凡复制，可以安全地并行使用：

```c++
#include <execution>

std::for_each_n(std::execution::par, v.begin(), 3, [](int& x) { x *= 2; });
```

## 4. 注意事项

- **`n` 必须非负**：如果 `n < 0`，行为未定义。
- **范围必须足够长**：调用者需自行保证 `[first, first + n)` 是有效范围，否则越界。
- **C++17 前无返回值**：如果需要在 C++11/14 中获取结束位置，需自行计算 `std::next(first, n)`。
- **函数对象按值传递**，需要修改外部变量时用引用捕获或 `std::ref`。

## 5. 相关算法

- [for_each](./For_each.md)：处理整个范围
- [copy_n](../Modifying/Copy.md)：复制前 \\( N \\) 个元素
- [fill_n](../Modifying/Fill.md)：给前 \\( N \\) 个元素赋值
- [generate_n](../Modifying/Generate.md)：用函数返回值填充前 \\( N \\) 个元素
