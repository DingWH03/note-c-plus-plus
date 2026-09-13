# std::adjacent_find

`std::adjacent_find` 在范围内查找**第一对相邻且相等**的元素（或第一对满足给定二元谓词的相邻元素），返回指向这对元素中**第一个**的迭代器。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt>
constexpr ForwardIt adjacent_find(ForwardIt first, ForwardIt last);

template<class ForwardIt, class BinaryPred>
constexpr ForwardIt adjacent_find(ForwardIt first, ForwardIt last, BinaryPred p);
```

- **迭代器要求**：`ForwardIterator`。
- **复杂度**：恰好 `min(N-1, 找到位置)` 次比较。
- **返回值**：指向第一对相邻匹配元素中的第一个；未找到时返回 `last`。
- **空范围或单元素范围**：返回 `last`。

`ranges::adjacent_find` 支持投影。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 3, 4, 5};

    auto it = std::adjacent_find(v.begin(), v.end());
    if (it != v.end())
        std::cout << "重复元素: " << *it << " 索引: " << (it - v.begin()) << '\n';
        // 重复元素: 3 索引: 2
}
```

### (2) 谓词与投影

用二元谓词自定义「相邻关系」，例如查找相邻的逆序对：

```c++
std::vector<int> v{1, 3, 2, 4, 5};

// 找第一对降序相邻元素
auto it = std::adjacent_find(v.begin(), v.end(),
                             [](int a, int b) { return a > b; });
std::cout << *it << ' ' << *(it + 1);   // 3 2
```

### (3) 执行策略

`adjacent_find` **不支持**执行策略，因为需要顺序比较相邻元素。

## 4. 注意事项

- **只找相邻的**：它不会找出所有重复元素，只找**第一对相邻**的重复。要找出全部重复，需要排序或用哈希表。
- **配合 `unique` 使用**：`adjacent_find` 常用来判断范围内是否有连续重复，而 `unique` 用来移除它们。
- **返回值是第一个元素**：返回的是这对元素中的第一个，不是第二个。
- **需要 forward 迭代器**。

## 5. 相关算法

- [unique](../Modifying/Unique.md)：移除连续重复元素
- [find / find_if](./Find.md)：查找单个元素
- [search_n](./Search.md)：查找连续出现 \\( N \\) 次的元素
- [count / count_if](./Count.md)：统计元素数量
