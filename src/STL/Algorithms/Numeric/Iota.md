# std::iota

`std::iota` 用**递增的起始值**依次填充范围：第一个元素赋 `value`，第二个赋 `value + 1`，依此类推（实际是对前一个值执行 `++`）。

## 1. 引入

```c++
#include <numeric>
```

## 2. 原理

```c++
template<class ForwardIt, class T>
constexpr void iota(ForwardIt first, ForwardIt last, T value);
```

- **迭代器要求**：`ForwardIterator`。
- **复杂度**：恰好 \\( n \\) 次自增和赋值。
- **返回值**：`void`。

实现等价于：

```c++
template<class ForwardIt, class T>
void iota(ForwardIt first, ForwardIt last, T value)
{
    while (first != last)
        *first++ = value++;
}
```

注意它用的是 `++` 而非 `+ 1`，因此对支持自增的自定义类型也适用。

`ranges::iota` 是 C++23 新增的 Ranges 版本，支持投影，且要求 `T` 与元素类型可赋值。

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <numeric>
#include <vector>

int main()
{
    std::vector<int> v(5);
    std::iota(v.begin(), v.end(), 1);
    // v = {1,2,3,4,5}

    // 从 0 开始
    std::vector<int> idx(5);
    std::iota(idx.begin(), idx.end(), 0);
    // idx = {0,1,2,3,4}

    // 填充字符
    std::string s(5, ' ');
    std::iota(s.begin(), s.end(), 'a');
    // s = "abcde"
}
```

### (2) 谓词与投影

`iota` 没有谓词参数。`ranges::iota` 支持投影，但投影通常用于只读算法，对 `iota` 意义不大。

### (3) 执行策略

`iota` **不支持**执行策略。它依赖严格的前后依赖关系（每个值基于前一个值），无法并行化。

## 4. 注意事项

- **递增是连续的**：`iota` 只能生成连续递增序列，步长固定为 1。需要其他步长时用 `generate` 配合 lambda。
- **溢出风险**：如果 `value + n` 超出类型范围，行为未定义。
- **`iota` 不是 `itoa`**：名字来自 APL 语言的 `ι`（iota），表示「连续整数序列」。
- **C++23 的 `ranges::iota`**：支持范围参数，用法更简洁。
- **与 `fill` 的区别**：`fill` 用固定值填充，`iota` 用递增序列填充。

## 5. 相关算法

- [fill / fill_n](../Modifying/Fill.md)：用固定值填充
- [generate / generate_n](../Modifying/Generate.md)：用函数返回值填充
- [partial_sum](./Partial_sum.md)：计算部分和
- [adjacent_difference](./Adjacent_difference.md)：计算相邻差值
