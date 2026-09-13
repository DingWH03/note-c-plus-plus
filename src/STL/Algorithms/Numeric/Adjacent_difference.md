# std::adjacent_difference

`std::adjacent_difference` 计算范围内**相邻元素之间的差值**：第一个输出元素是第一个输入元素本身，之后的每个输出元素是当前元素与前一个元素的差。

## 1. 引入

```c++
#include <numeric>
```

## 2. 原理

```c++
template<class InputIt, class OutputIt>
constexpr OutputIt adjacent_difference(InputIt first, InputIt last, OutputIt d_first);

template<class InputIt, class OutputIt, class BinaryOp>
constexpr OutputIt adjacent_difference(InputIt first, InputIt last, OutputIt d_first,
                                       BinaryOp op);
```

- **迭代器要求**：输入 `InputIterator`，输出 `OutputIterator`。
- **复杂度**：恰好 \\( n-1 \\) 次 `op` 调用。
- **返回值**：指向输出范围末尾的迭代器。

计算规则：

- `*d_first = *first`（第一个元素原样输出）
- `*(d_first + i) = op(*(first + i), *(first + i - 1))`，默认 `op` 是减法

这是 `partial_sum` 的**逆运算**：对序列做 `partial_sum` 再做 `adjacent_difference` 会得到原序列。

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <numeric>
#include <vector>

int main()
{
    std::vector<int> v{2, 4, 6, 8, 10};

    std::vector<int> diff(v.size());
    std::adjacent_difference(v.begin(), v.end(), diff.begin());
    // diff = {2, 2, 2, 2, 2}

    // 原地计算
    std::vector<int> v2{1, 3, 6, 10, 15};
    std::adjacent_difference(v2.begin(), v2.end(), v2.begin());
    // v2 = {1, 2, 3, 4, 5}

    // 自定义操作：相邻元素之和
    std::vector<int> v3{1, 2, 3, 4};
    std::vector<int> sum(v3.size());
    std::adjacent_difference(v3.begin(), v3.end(), sum.begin(), std::plus<>{});
    // sum = {1, 3, 5, 7}
}
```

### (2) 谓词与投影

接受二元操作，没有投影参数。

### (3) 执行策略

`adjacent_difference` **不支持**执行策略，因为每个输出依赖相邻的两个输入。

## 4. 注意事项

- **输出第一个元素是输入的第一个元素**：不是差值，容易忽略。
- **可以原地计算**：`d_first == first` 是允许的（实现会从后往前处理）。
- **输出范围必须足够大**：至少与输入等长。
- **与 `partial_sum` 互逆**：`adjacent_difference(partial_sum(x)) == x`。
- **默认操作是减法**：`op(*i, *(i-1))` 的顺序是「当前减前一个」。

## 5. 相关算法

- [partial_sum](./Partial_sum.md)：部分和（逆运算）
- [accumulate](./Accumulate.md)：求和
- [inner_product](./Inner_product.md)：内积
- [iota](./Iota.md)：递增填充
