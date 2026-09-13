# std::reverse / std::reverse_copy

这两个算法反转元素顺序：

- `reverse`：原地反转范围内的元素顺序
- `reverse_copy`：把反转后的序列复制到目标范围，原范围不变

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class BidirIt>
constexpr void reverse(BidirIt first, BidirIt last);

template<class BidirIt, class OutputIt>
constexpr OutputIt reverse_copy(BidirIt first, BidirIt last, OutputIt d_first);
```

- **迭代器要求**：`reverse` 需要 `BidirectionalIterator`；`reverse_copy` 输入需要 `BidirectionalIterator`，输出 `OutputIterator`。
- **复杂度**：恰好 \\( n/2 \\) 次交换（`reverse`）或 \\( n \\) 次赋值（`reverse_copy`）。
- **返回值**：`reverse` 返回 `void`；`reverse_copy` 返回目标范围的末尾。

`reverse` 的实现就是双指针向中间靠拢并交换：

```c++
template<class BidirIt>
void reverse(BidirIt first, BidirIt last)
{
    while ((first != last) && (first != --last))
        std::iter_swap(first++, last);
}
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

    // 原地反转
    std::reverse(v.begin(), v.end());   // v = {5,4,3,2,1}

    // 反转后复制
    std::vector<int> src{1, 2, 3};
    std::vector<int> dst;
    std::reverse_copy(src.begin(), src.end(), std::back_inserter(dst));
    // src = {1,2,3}（不变），dst = {3,2,1}
}
```

### (2) 谓词与投影

`reverse` 和 `reverse_copy` 都**没有谓词或投影参数**，它们无条件反转。

### (3) 执行策略

`reverse` 和 `reverse_copy` 支持 C++17 执行策略：

```c++
#include <execution>

std::reverse(std::execution::par, v.begin(), v.end());
```

## 4. 注意事项

- **需要双向迭代器**：`forward_list` 的迭代器不可用，但 `forward_list` 有成员函数 `reverse`。
- **`reverse` 返回 `void`**：不能链式调用。
- **反向迭代器更简洁**：只是遍历时用 `rbegin()` / `rend()` 即可，不需要真的反转。
- **`std::string` 也有 `reverse`**：通过 `std::reverse(s.begin(), s.end())` 使用。
- **性能**：`reverse` 是 \\( O(n) \\) 且原地操作，非常高效。

## 5. 相关算法

- [rotate / rotate_copy](./Rotate.md)：旋转而非反转
- [reverse_iterator](../NonModifying/For_each.md)：反向遍历
- [sort / stable_sort](../Sorting/Sort.md)：排序
- [next_permutation / prev_permutation](../Sorting/Permutation.md)：排列
