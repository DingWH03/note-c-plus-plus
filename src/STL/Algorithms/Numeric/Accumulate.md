# std::accumulate

`std::accumulate` 对范围内的元素进行**左折叠**：从初始值 `init` 开始，依次对每个元素应用二元操作，返回最终结果。默认操作是加法（求和）。

## 1. 引入

```c++
#include <numeric>
```

## 2. 原理

```c++
template<class InputIt, class T>
constexpr T accumulate(InputIt first, InputIt last, T init);

template<class InputIt, class T, class BinaryOp>
constexpr T accumulate(InputIt first, InputIt last, T init, BinaryOp op);
```

- **迭代器要求**：`InputIterator`。
- **复杂度**：恰好 \\( n \\) 次 `op` 调用。
- **返回值**：折叠结果，类型为 `T`。

实现等价于：

```c++
template<class InputIt, class T, class BinaryOp>
T accumulate(InputIt first, InputIt last, T init, BinaryOp op)
{
    for (; first != last; ++first)
        init = op(std::move(init), *first);
    return init;
}
```

**严格按顺序求值**，因此对非交换、非结合的操作也能给出确定结果。

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <numeric>
#include <string>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    // 求和
    int sum = std::accumulate(v.begin(), v.end(), 0);
    std::cout << sum << '\n';   // 15

    // 求积
    int product = std::accumulate(v.begin(), v.end(), 1, std::multiplies<>{});
    std::cout << product << '\n';   // 120

    // 拼接字符串
    std::vector<std::string> words{"a", "b", "c"};
    std::string joined = std::accumulate(words.begin(), words.end(), std::string{});
    std::cout << joined << '\n';   // abc

    // 求最大值
    int mx = std::accumulate(v.begin(), v.end(), v.front(),
                             [](int a, int b) { return std::max(a, b); });
    std::cout << mx << '\n';   // 5
}
```

### (2) 谓词与投影

`accumulate` 接受二元操作，没有投影参数。可以自己处理成员：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}};

int totalAge = std::accumulate(people.begin(), people.end(), 0,
                               [](int acc, const Person& p) { return acc + p.age; });
// 55
```

### (3) 执行策略

`accumulate` **不支持**执行策略，因为它要求严格顺序求值。需要并行归约时用 `std::reduce`（要求操作满足交换律和结合律）。

## 4. 注意事项

- **初始值类型决定返回类型**：`std::accumulate(v.begin(), v.end(), 0)` 返回 `int`，即使元素是 `double` 也会被截断。应写成 `0.0`。
- **初始值类型决定累加类型**：常见的坑是 `accumulate(v.begin(), v.end(), 0)` 对 `vector<double>` 会丢失小数部分。
- **顺序求值**：`accumulate` 严格按顺序，因此对浮点数求和的结果是可复现的，但也无法并行加速。
- **空范围返回 `init`**。
- **与 `reduce` 的区别**：`reduce` 允许任意顺序，要求操作满足交换律和结合律，但支持并行。

## 5. 相关算法

- [reduce](./Reduce.md)：允许无序的并行归约
- [transform_reduce](./Transform_reduce.md)：先变换再归约
- [partial_sum](./Partial_sum.md)：计算部分和
- [fold](../NonModifying/Fold.md)：C++23 的折叠算法
- [inner_product](./Inner_product.md)：内积
