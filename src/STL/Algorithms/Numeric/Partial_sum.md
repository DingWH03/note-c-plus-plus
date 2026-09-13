# std::partial_sum

`std::partial_sum` 计算范围的**部分和**：第 \\( i \\) 个输出元素是输入范围前 \\( i+1 \\) 个元素的累加结果。

## 1. 引入

```c++
#include <numeric>
```

## 2. 原理

```c++
template<class InputIt, class OutputIt>
constexpr OutputIt partial_sum(InputIt first, InputIt last, OutputIt d_first);

template<class InputIt, class OutputIt, class BinaryOp>
constexpr OutputIt partial_sum(InputIt first, InputIt last, OutputIt d_first, BinaryOp op);
```

- **迭代器要求**：输入 `InputIterator`，输出 `OutputIterator`。
- **复杂度**：恰好 \\( n-1 \\) 次 `op` 调用。
- **返回值**：指向输出范围末尾的迭代器。

计算规则：

- `*d_first = *first`
- `*(d_first + i) = op(*(d_first + i - 1), *(first + i))`

默认 `op` 是加法。这是**包含扫描**（inclusive scan），与 `inclusive_scan` 语义相同。

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <numeric>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    std::vector<int> sums(v.size());
    std::partial_sum(v.begin(), v.end(), sums.begin());
    // sums = {1, 3, 6, 10, 15}

    // 原地计算
    std::vector<int> v2{1, 2, 3, 4, 5};
    std::partial_sum(v2.begin(), v2.end(), v2.begin());
    // v2 = {1, 3, 6, 10, 15}

    // 自定义操作：部分积
    std::vector<int> v3{1, 2, 3, 4};
    std::vector<int> prods(v3.size());
    std::partial_sum(v3.begin(), v3.end(), prods.begin(), std::multiplies<>{});
    // prods = {1, 2, 6, 24}
}
```

### (2) 谓词与投影

接受二元操作，没有投影参数。

### (3) 执行策略

`partial_sum` **不支持**执行策略，因为每个输出依赖前一个输出。需要并行扫描时用 `inclusive_scan` 或 `exclusive_scan`。

## 4. 注意事项

- **第一个输出等于第一个输入**。
- **可以原地计算**：`d_first == first` 是允许的。
- **输出范围必须足够大**。
- **溢出风险**：累加值可能超出类型范围。
- **与 `inclusive_scan` 的关系**：语义相同，但 `inclusive_scan` 支持执行策略且允许指定初始值。
- **与 `accumulate` 的区别**：`accumulate` 只返回最终结果，`partial_sum` 返回每一步的中间结果。

## 5. 相关算法

- [adjacent_difference](./Adjacent_difference.md)：相邻差值（逆运算）
- [accumulate](./Accumulate.md)：只求总和
- [inclusive_scan / exclusive_scan](./Scan.md)：可并行的扫描
- [reduce](./Reduce.md)：可并行的归约
