# std::ranges::fold_left / fold_right 系列

C++23 引入的折叠算法对范围内的元素进行**左折叠**或**右折叠**，相当于 C++17 折叠表达式（`(... op pack)`）的运行时版本。它们**只有 Ranges 版本**。

| 算法 | 说明 |
| :--- | :--- |
| `fold_left` | 左折叠，需要提供初始值 |
| `fold_left_first` | 左折叠，用第一个元素作为初始值 |
| `fold_right` | 右折叠，需要提供初始值 |
| `fold_right_last` | 右折叠，用最后一个元素作为初始值 |
| `fold_left_with_iter` | 左折叠，返回 `in_value_result`（迭代器 + 值） |
| `fold_left_first_with_iter` | 左折叠（首元素为初值），返回 `in_value_result` |

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

折叠的本质是把二元操作依次应用到元素上。以左折叠为例，`fold_left(r, init, f)` 计算：

$$
f(f(f(init, x_1), x_2), \dots, x_n)
$$

而右折叠 `fold_right(r, init, f)` 计算：

$$
f(x_1, f(x_2, f(\dots, f(x_n, init))))
$$

- **迭代器要求**：`InputIterator`（`fold_right` 系列需要 `BidirectionalIterator`）。
- **复杂度**：恰好 \\( n \\) 次函数调用。
- **返回值**：折叠结果；`*_with_iter` 版本返回 `ranges::in_value_result`。
- **空范围**：`fold_left` / `fold_right` 返回初始值；`fold_left_first` / `fold_right_last` 对空范围返回 `std::optional` 的空值。

函数签名（以 `fold_left` 为例）：

```c++
template<std::input_iterator I, std::sentinel_for<I> S, class T,
         class F = std::plus<>>
requires std::indirectly_binary_left_foldable<F, T, I>
constexpr auto fold_left(I first, S last, T init, F f = {});
```

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    // 左折叠求和：(0+1)+2)+3)+4)+5
    int sum = std::ranges::fold_left(v, 0, std::plus<>{});
    std::cout << sum << '\n';   // 15

    // 用第一个元素作为初始值
    int sum2 = std::ranges::fold_left_first(v, std::plus<>{}).value();
    std::cout << sum2 << '\n';   // 15

    // 左折叠求最大值
    int mx = std::ranges::fold_left(v, v.front(),
                                    [](int a, int b) { return std::max(a, b); });
    std::cout << mx << '\n';   // 5
}
```

### (2) 谓词与投影

折叠算法接受二元操作 `f`，本身没有投影参数，但可以在 `f` 内部处理：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}, {"Carol", 35}};

// 求年龄总和
int total = std::ranges::fold_left(people, 0,
                                   [](int acc, const Person& p) { return acc + p.age; });
std::cout << total;   // 90
```

左折叠与右折叠对**非结合运算**结果不同，例如字符串拼接：

```c++
std::vector<std::string> words{"a", "b", "c"};

auto l = std::ranges::fold_left(words, std::string{}, std::plus<>{});   // "abc"
auto r = std::ranges::fold_right(words, std::string{}, std::plus<>{});  // "abc"
// 但对减法这类运算，两者结果不同
```

### (3) 执行策略

折叠算法**不支持**执行策略。它们有严格的求值顺序要求，无法并行化。需要并行归约时应使用 `std::reduce` 或 `std::transform_reduce`。

## 4. 注意事项

- **只有 Ranges 版本**：C++23 才引入，且没有 `std::fold_left` 经典版本。
- **`fold_left_first` 返回 `optional`**：对空范围返回空 `optional`，使用前需检查或调用 `.value()`（可能抛异常）。
- **`fold_right` 需要双向迭代器**：因为要从后往前遍历。
- **结合律很重要**：如果 `f` 不满足结合律，左折叠和右折叠结果不同，需根据语义选择。
- **与 `accumulate` 的区别**：`std::accumulate` 是左折叠的经典版本，但只支持迭代器对且没有 `optional` 返回；`fold_left_first` 等是更现代的补充。
- **与 `reduce` 的区别**：`reduce` 允许任意顺序，要求运算满足交换律；折叠算法严格按顺序求值。

## 5. 相关算法

- [accumulate](../Numeric/Accumulate.md)：经典左折叠
- [reduce](../Numeric/Reduce.md)：允许无序的并行归约
- [transform_reduce](../Numeric/Transform_reduce.md)：先变换再归约
- [for_each](./For_each.md)：只遍历不求值
