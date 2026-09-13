# std::nth_element

`std::nth_element` **部分排序**范围，使得第 \\( N \\) 个位置的元素就位（即如果整个范围排序，该位置就是它），并保证它左边的元素都不大于它，右边的元素都不小于它。

它常用于求「第 K 小/大的元素」，平均复杂度为 \\( O(n) \\)，比完整排序的 \\( O(n \log n) \\) 更快。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class RandomIt>
constexpr void nth_element(RandomIt first, RandomIt nth, RandomIt last);

template<class RandomIt, class Compare>
constexpr void nth_element(RandomIt first, RandomIt nth, RandomIt last, Compare comp);
```

- **迭代器要求**：`RandomAccessIterator`。
- **复杂度**：平均 \\( O(n) \\) 次比较（典型实现为 Introselect：快选 + 堆选混合），最坏 \\( O(n \log n) \\)。
- **返回值**：`void`。

实现基于**快速选择（Quickselect）**：类似快排的分区步骤，但只递归处理包含目标位置的那一侧。

分区后满足：

- `[first, nth)` 中的所有元素都 `<= *nth`
- `[nth, last)` 中的所有元素都 `>= *nth`

但两侧内部的顺序是**未指定的**。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{9, 3, 7, 1, 5, 8, 2, 6, 4};

    // 找第 5 小（索引 4）
    std::nth_element(v.begin(), v.begin() + 4, v.end());
    std::cout << "第 5 小: " << v[4] << '\n';   // 5

    // 找中位数
    std::vector<int> v2{9, 3, 7, 1, 5};
    auto mid = v2.begin() + v2.size() / 2;
    std::nth_element(v2.begin(), mid, v2.end());
    std::cout << "中位数: " << *mid << '\n';   // 5

    // 找第 3 大：用 greater 比较器
    std::vector<int> v3{9, 3, 7, 1, 5};
    std::nth_element(v3.begin(), v3.begin() + 2, v3.end(), std::greater<>{});
    std::cout << "第 3 大: " << v3[2] << '\n';   // 5
}
```

### (2) 谓词与投影

`ranges::nth_element` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}, {"Carol", 35}};

// 找年龄中位数
std::ranges::nth_element(people, people.begin() + 1, {}, &Person::age);
```

### (3) 执行策略

`nth_element` 支持 C++17 执行策略。

## 4. 注意事项

- **两侧顺序未指定**：`nth_element` 只保证分区性质，不保证两侧有序。需要两侧有序时用 `partial_sort` 或 `sort`。
- **比较器必须严格弱序**。
- **比 `sort` 快得多**：只需要第 K 小时，`nth_element` 是最优选择。
- **可以求前 K 小（无序）**：`nth_element(v.begin(), v.begin() + k, v.end())` 后，前 K 个就是最小的 K 个（顺序不定）。
- **`nth` 必须在范围内**。

## 5. 相关算法

- [partial_sort](./Partial_sort.md)：前 \\( N \\) 个有序
- [sort / stable_sort](./Sort.md)：完整排序
- [partition / stable_partition](./Partition.md)：按谓词分区
- [min / max / clamp](./Min_max.md)：求最值
