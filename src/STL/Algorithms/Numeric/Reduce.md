# std::reduce

C++17 引入的 `std::reduce` 与 `std::accumulate` 类似，都是对范围进行归约，但 `reduce` **允许以任意顺序**组合元素，因此可以并行化。

**代价**：操作必须满足**交换律和结合律**，否则结果不确定。

## 1. 引入

```c++
#include <numeric>
```

## 2. 原理

```c++
template<class InputIt>
constexpr typename iterator_traits<InputIt>::value_type reduce(InputIt first, InputIt last);

template<class InputIt, class T>
constexpr T reduce(InputIt first, InputIt last, T init);

template<class InputIt, class T, class BinaryOp>
constexpr T reduce(InputIt first, InputIt last, T init, BinaryOp op);

template<class ExecutionPolicy, class ForwardIt, class T, class BinaryOp>
T reduce(ExecutionPolicy&& policy, ForwardIt first, ForwardIt last, T init, BinaryOp op);
```

- **迭代器要求**：`InputIterator`（并行版本需要 `ForwardIterator`）。
- **复杂度**：\\( O(n) \\) 次 `op` 调用。
- **返回值**：归约结果。

与 `accumulate` 的关键区别：

| 特性 | `accumulate` | `reduce` |
| :--- | :--- | :--- |
| 求值顺序 | 严格从左到右 | 任意顺序 |
| 并行支持 | 否 | 是 |
| 操作要求 | 无 | 必须满足交换律和结合律 |
| 结果确定性 | 确定 | 浮点运算可能有细微差异 |

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <numeric>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    // 求和
    int sum = std::reduce(v.begin(), v.end());
    std::cout << sum << '\n';   // 15

    // 指定初始值
    int sum2 = std::reduce(v.begin(), v.end(), 100);
    std::cout << sum2 << '\n';   // 115

    // 自定义操作：求积
    int product = std::reduce(v.begin(), v.end(), 1, std::multiplies<>{});
    std::cout << product << '\n';   // 120
}
```

### (2) 谓词与投影

`reduce` 接受二元操作，没有投影参数。需要先变换时用 `transform_reduce`。

### (3) 执行策略

`reduce` 支持 C++17 执行策略：

```c++
#include <execution>

int sum = std::reduce(std::execution::par, v.begin(), v.end(), 0);
```

**重要**：并行版本要求操作满足交换律和结合律。对于浮点加法，由于精度问题，并行结果可能与串行结果有细微差异。

## 4. 注意事项

- **操作必须满足交换律和结合律**：这是使用 `reduce` 的硬性要求。减法、除法不满足，不能用。
- **浮点数的非确定性**：并行归约的分组方式取决于实现，浮点加法不满足结合律，结果可能有微小差异。
- **无初始值版本**：`reduce(first, last)` 要求范围非空，且返回 `value_type`。
- **与 `accumulate` 的选择**：需要严格顺序或操作不满足交换律时用 `accumulate`；需要并行加速时用 `reduce`。
- **`reduce` 的初始值类型**：不指定时用 `value_type{}`，可能不是期望的类型。

## 5. 相关算法

- [accumulate](./Accumulate.md)：严格顺序的归约
- [transform_reduce](./Transform_reduce.md)：先变换再归约
- [inclusive_scan / exclusive_scan](./Scan.md)：扫描
- [fold](../NonModifying/Fold.md)：C++23 的折叠算法
