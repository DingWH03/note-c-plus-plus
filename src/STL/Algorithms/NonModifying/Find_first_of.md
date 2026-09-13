# std::find_first_of

`std::find_first_of` 在范围 `[first, last)` 中查找**任意一个**出现在集合 `[s_first, s_last)` 中的元素，返回第一个这样的元素的位置。

注意它的语义：不是查找整个子序列，而是查找「集合中的任意一个元素」。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class ForwardIt>
constexpr InputIt find_first_of(InputIt first, InputIt last,
                                ForwardIt s_first, ForwardIt s_last);

template<class InputIt, class ForwardIt, class BinaryPred>
constexpr InputIt find_first_of(InputIt first, InputIt last,
                                ForwardIt s_first, ForwardIt s_last, BinaryPred p);
```

- **迭代器要求**：主范围为 `InputIterator`，集合为 `ForwardIterator`。
- **复杂度**：至多 \\( O(N \cdot S) \\) 次比较，其中 \\( N \\) 为主范围长度，\\( S \\) 为集合大小。
- **返回值**：指向第一个匹配元素；无匹配时返回 `last`。

`ranges::find_first_of` 支持投影。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};
    std::vector<int> targets{4, 6, 2};

    // 查找 targets 中任意一个元素在 v 中首次出现的位置
    auto it = std::find_first_of(v.begin(), v.end(), targets.begin(), targets.end());
    std::cout << *it << '\n';   // 2
}
```

### (2) 谓词与投影

二元谓词用于自定义相等判断：

```c++
std::string text = "Hello World";
std::string vowels = "aeiouAEIOU";

// 找第一个元音
auto it = std::find_first_of(text.begin(), text.end(),
                             vowels.begin(), vowels.end());
std::cout << *it;   // e
```

### (3) 执行策略

`find_first_of` **不支持**执行策略。

## 4. 注意事项

- **语义容易误解**：它查找的是「集合中任意一个元素」，而不是「集合作为子序列」。如果想找子序列，用 `search`。
- **复杂度是乘积**：\\( O(N \cdot S) \\)，集合很大时性能较差。集合较大时可考虑先排序再二分，或使用哈希集合。
- **集合不能为空**：如果 `[s_first, s_last)` 为空，返回 `last`。
- **有序范围可用二分**：如果两个范围都有序，可以用 `std::set_intersection` 更高效地求交集。

## 5. 相关算法

- [search / search_n](./Search.md)：查找完整的子序列
- [find_end](./Find_end.md)：查找子序列的最后一次出现
- [find / find_if](./Find.md)：查找单个元素
- [集合操作](../Sorting/Set_operations.md)：在有序范围上求交集
