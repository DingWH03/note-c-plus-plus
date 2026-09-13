# std::rotate / std::rotate_copy

这两个算法**旋转**范围内的元素，即把中间位置的元素移到开头：

- `rotate`：原地旋转
- `rotate_copy`：旋转后复制到目标范围

`rotate(first, middle, last)` 的效果是把 `[first, middle)` 移到 `[middle, last)` 之后，等价于把序列左移 `middle - first` 位。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt>
constexpr ForwardIt rotate(ForwardIt first, ForwardIt middle, ForwardIt last);

template<class ForwardIt, class OutputIt>
constexpr OutputIt rotate_copy(ForwardIt first, ForwardIt middle, ForwardIt last,
                               OutputIt d_first);
```

- **迭代器要求**：`ForwardIterator`。
- **复杂度**：恰好 \\( n \\) 次交换或赋值。
- **返回值**：`rotate` 返回原 `first` 所指向元素的新位置（即 `first + (last - middle)`）。

C++11 起 `rotate` 的实现优化为 \\( O(n) \\) 次交换（此前是 \\( O(n^2) \\) 的朴素实现）。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    // 把中间位置（索引 2）的元素移到开头
    std::rotate(v.begin(), v.begin() + 2, v.end());
    // v = {3,4,5,1,2}

    // 左移 1 位
    std::vector<int> v2{1, 2, 3, 4, 5};
    std::rotate(v2.begin(), v2.begin() + 1, v2.end());
    // v2 = {2,3,4,5,1}

    // 右移 1 位（等价于左移 n-1 位）
    std::vector<int> v3{1, 2, 3, 4, 5};
    std::rotate(v3.begin(), v3.end() - 1, v3.end());
    // v3 = {5,1,2,3,4}

    // rotate_copy
    std::vector<int> src{1, 2, 3, 4, 5};
    std::vector<int> dst;
    std::rotate_copy(src.begin(), src.begin() + 2, src.end(), std::back_inserter(dst));
    // dst = {3,4,5,1,2}
}
```

### (2) 谓词与投影

`rotate` 和 `rotate_copy` 都**没有谓词或投影参数**。

### (3) 执行策略

`rotate` 和 `rotate_copy` 支持 C++17 执行策略：

```c++
#include <execution>

std::rotate(std::execution::par, v.begin(), v.begin() + 2, v.end());
```

## 4. 注意事项

- **`middle` 必须在范围内**：`middle` 可以等于 `first` 或 `last`，此时无操作。
- **返回值的含义**：返回原 `first` 元素的新位置，容易误解为「新开头」。
- **C++20 有 `shift_left` / `shift_right`**：如果只是简单地左移或右移，用 `std::shift_left` / `std::shift_right` 语义更清晰。
- **`std::rotate` 常用于实现「移动元素到末尾」**：例如把某个元素移到容器末尾。

## 5. 相关算法

- [shift_left / shift_right](./Shift.md)：左移或右移元素
- [reverse / reverse_copy](./Reverse.md)：反转顺序
- [next_permutation / prev_permutation](../Sorting/Permutation.md)：排列
- [partition](../Sorting/Partition.md)：按谓词分组
