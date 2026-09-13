# std::transform_exclusive_scan / std::transform_inclusive_scan

C++17 引入的这两个算法先对元素应用变换，再计算扫描（前缀和），相当于 `transform` + `scan` 的组合。

- `transform_inclusive_scan`：结果**包含**当前元素
- `transform_exclusive_scan`：结果**不包含**当前元素

## 1. 引入

```c++
#include <numeric>
```

## 2. 原理

```c++
template<class InputIt, class OutputIt, class BinaryOp, class UnaryOp>
constexpr OutputIt transform_inclusive_scan(InputIt first, InputIt last, OutputIt d_first,
                                            BinaryOp binary_op, UnaryOp unary_op);

template<class InputIt, class OutputIt, class T, class BinaryOp, class UnaryOp>
constexpr OutputIt transform_exclusive_scan(InputIt first, InputIt last, OutputIt d_first,
                                            T init, BinaryOp binary_op, UnaryOp unary_op);
```

- **迭代器要求**：输入 `InputIterator`，输出 `OutputIterator`（并行版本需要 `ForwardIterator`）。
- **复杂度**：\\( O(n) \\) 次变换和归约调用。
- **返回值**：指向输出范围末尾的迭代器。

注意参数顺序：**先归约操作 `binary_op`，再变换操作 `unary_op`**，这与直觉相反。

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <numeric>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4};

    // 先平方，再包含扫描求和
    std::vector<int> inc(v.size());
    std::transform_inclusive_scan(v.begin(), v.end(), inc.begin(),
                                  std::plus<>{},
                                  [](int x) { return x * x; });
    // 平方后为 {1,4,9,16}，扫描得 inc = {1, 5, 14, 30}

    // 独占扫描
    std::vector<int> exc(v.size());
    std::transform_exclusive_scan(v.begin(), v.end(), exc.begin(), 0,
                                  std::plus<>{},
                                  [](int x) { return x * x; });
    // exc = {0, 1, 5, 14}
}
```

### (2) 谓词与投影

接受归约和变换两个可调用对象，没有投影参数。

### (3) 执行策略

两者都支持 C++17 执行策略：

```c++
#include <execution>

std::transform_inclusive_scan(std::execution::par, v.begin(), v.end(), out.begin(),
                              std::plus<>{}, [](int x) { return x * x; });
```

## 4. 注意事项

- **参数顺序反直觉**：`binary_op` 在前，`unary_op` 在后，容易传错。
- **`transform_exclusive_scan` 必须提供初始值**。
- **操作必须满足结合律**（并行时还需交换律）。
- **原地操作**：`transform_inclusive_scan` 允许 `d_first == first`；`transform_exclusive_scan` 不允许。
- **`transform_inclusive_scan` 的初始值**：可以额外传入 `init`，此时结果会加上该初始值。

## 5. 相关算法

- [inclusive_scan / exclusive_scan](./Scan.md)：不带变换的扫描
- [transform_reduce](./Transform_reduce.md)：变换后归约
- [partial_sum](./Partial_sum.md)：顺序版本的扫描
- [transform](../Modifying/Transform.md)：只做变换
