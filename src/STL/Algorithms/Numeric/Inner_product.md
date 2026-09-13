# std::inner_product

`std::inner_product` 计算两个范围的**内积**：把对应元素相乘后累加。默认行为是「乘积累加」，但可以通过两个自定义操作实现更一般的「广义内积」。

## 1. 引入

```c++
#include <numeric>
```

## 2. 原理

```c++
template<class InputIt1, class InputIt2, class T>
constexpr T inner_product(InputIt1 first1, InputIt1 last1, InputIt2 first2, T init);

template<class InputIt1, class InputIt2, class T, class BinaryOp1, class BinaryOp2>
constexpr T inner_product(InputIt1 first1, InputIt1 last1, InputIt2 first2, T init,
                          BinaryOp1 op1, BinaryOp2 op2);
```

- **迭代器要求**：`InputIterator`。
- **复杂度**：恰好 \\( n \\) 次 `op1` 和 \\( n \\) 次 `op2` 调用。
- **返回值**：内积结果，类型为 `T`。

实现等价于：

```c++
template<class InputIt1, class InputIt2, class T, class BinaryOp1, class BinaryOp2>
T inner_product(InputIt1 first1, InputIt1 last1, InputIt2 first2, T init,
                BinaryOp1 op1, BinaryOp2 op2)
{
    for (; first1 != last1; ++first1, ++first2)
        init = op1(std::move(init), op2(*first1, *first2));
    return init;
}
```

默认 `op1` 是 `+`，`op2` 是 `*`。**严格按顺序求值**。

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <numeric>
#include <vector>

int main()
{
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{4, 5, 6};

    // 内积：1*4 + 2*5 + 3*6 = 32
    int dot = std::inner_product(a.begin(), a.end(), b.begin(), 0);
    std::cout << dot << '\n';   // 32

    // 广义内积：先相加再取最大值（相当于求 max(a[i]+b[i])）
    int mx = std::inner_product(a.begin(), a.end(), b.begin(), INT_MIN,
                                [](int x, int y) { return std::max(x, y); },
                                std::plus<>{});
    std::cout << mx << '\n';   // 9
}
```

### (2) 谓词与投影

`inner_product` 接受两个二元操作，没有投影参数。

### (3) 执行策略

`inner_product` **不支持**执行策略，因为要求严格顺序求值。需要并行时用 `transform_reduce`：

```c++
// 并行计算内积
int dot = std::transform_reduce(std::execution::par,
                                a.begin(), a.end(), b.begin(), 0);
```

## 4. 注意事项

- **第二个范围必须足够长**：只检查第一个范围的长度，第二个范围较短会越界。
- **初始值类型决定返回类型**：同样有 `0` 与 `0.0` 的坑。
- **顺序求值**：无法并行加速。
- **`transform_reduce` 是并行替代品**：语义相同但允许乱序，要求操作满足交换律和结合律。
- **数学上的内积**：对向量来说就是点积，是很多数值算法的基础。

## 5. 相关算法

- [accumulate](./Accumulate.md)：单范围折叠
- [transform_reduce](./Transform_reduce.md)：先变换再归约（可并行）
- [adjacent_difference](./Adjacent_difference.md)：相邻差值
- [partial_sum](./Partial_sum.md)：部分和
