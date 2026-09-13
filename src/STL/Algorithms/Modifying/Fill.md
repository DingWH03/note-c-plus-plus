# std::fill / std::fill_n

这两个算法把给定值赋给范围内的元素：

- `fill`：给整个范围赋值
- `fill_n`：给前 \\( N \\) 个元素赋值

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt, class T>
constexpr void fill(ForwardIt first, ForwardIt last, const T& value);

template<class OutputIt, class Size, class T>
constexpr OutputIt fill_n(OutputIt first, Size count, const T& value);
```

- **迭代器要求**：`fill` 需要 `ForwardIterator`；`fill_n` 需要 `OutputIterator`。
- **复杂度**：恰好 \\( n \\) 次赋值。
- **返回值**：`fill` 返回 `void`；`fill_n` 返回最后一个被赋值元素之后的位置。

`fill` 的经典实现会对可平凡赋值的类型调用 `std::memset`，因此对 `int`、`char` 等非常高效。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v(5);
    std::fill(v.begin(), v.end(), 7);   // v = {7,7,7,7,7}

    // 只填充前 3 个
    std::vector<int> v2(5);
    std::fill_n(v2.begin(), 3, 1);      // v2 = {1,1,1,0,0}

    // 配合 back_inserter 填充空容器
    std::vector<int> v3;
    std::fill_n(std::back_inserter(v3), 4, 9);   // v3 = {9,9,9,9}
}
```

### (2) 谓词与投影

`fill` 和 `fill_n` 都**没有谓词参数**，它们无条件赋值。需要按条件赋值时用 `replace_if` 或 `transform`。

### (3) 执行策略

`fill` 和 `fill_n` 支持 C++17 执行策略：

```c++
#include <execution>

std::fill(std::execution::par, v.begin(), v.end(), 7);
```

## 4. 注意事项

- **`fill_n` 的 `count` 非负**：`count < 0` 时行为未定义。
- **目标空间必须足够**：`fill_n` 不检查范围大小。
- **`fill` vs `std::array::fill`**：`std::array` 和 `std::vector` 等容器有成员函数 `fill`，用法类似。
- **`memset` 只适用于 0 和 -1**：不要用 `memset` 给 `int` 数组赋非 0/-1 的值，那会得到错误结果（按字节填充）。
- **初始化列表 vs `fill`**：构造时直接用 `std::vector<int> v(5, 7)` 更简洁。

## 5. 相关算法

- [generate / generate_n](./Generate.md)：用函数返回值填充
- [replace / replace_if](./Replace.md)：按条件替换
- [iota](../Numeric/Iota.md)：用递增序列填充
- [uninitialized_fill](../Memory/Uninitialized_fill.md)：填充未初始化内存
