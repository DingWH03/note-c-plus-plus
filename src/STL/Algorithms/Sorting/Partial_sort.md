# std::partial_sort / std::partial_sort_copy

这两个算法只对范围内的**前 \\( N \\) 个元素**排序：

- `partial_sort`：原地把最小的 \\( N \\) 个元素排到前面
- `partial_sort_copy`：把最小的 \\( N \\) 个元素排序后复制到目标范围

当只需要「前 K 名」时，它们比完整排序 `sort` 更高效。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class RandomIt>
constexpr void partial_sort(RandomIt first, RandomIt middle, RandomIt last);

template<class InputIt, class RandomIt>
constexpr RandomIt partial_sort_copy(InputIt first, InputIt last,
                                     RandomIt d_first, RandomIt d_last);
```

- **迭代器要求**：`partial_sort` 需要 `RandomAccessIterator`；`partial_sort_copy` 输入 `InputIterator`、输出 `RandomAccessIterator`。
- **复杂度**：约 \\( O(N \log K) \\) 次比较，其中 \\( N \\) 为范围长度，\\( K \\) 为 `middle - first`。
- **返回值**：`partial_sort` 返回 `void`；`partial_sort_copy` 返回目标范围中最后一个被写入元素之后的位置。

实现基于**堆**：先用前 \\( K \\) 个元素建最大堆，然后遍历剩余元素，比堆顶小的就替换堆顶并调整堆，最后对堆排序。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{9, 3, 7, 1, 5, 8, 2, 6, 4};

    // 把最小的 3 个排到前面
    std::partial_sort(v.begin(), v.begin() + 3, v.end());
    // v 的前 3 个是 {1,2,3}，后面元素的顺序不确定

    // 降序取前 3 大
    std::vector<int> v2{9, 3, 7, 1, 5, 8, 2, 6, 4};
    std::partial_sort(v2.begin(), v2.begin() + 3, v2.end(), std::greater<>{});
    // v2 前 3 个是 {9,8,7}

    // partial_sort_copy：复制前 3 小到新容器
    std::vector<int> src{9, 3, 7, 1, 5};
    std::vector<int> dst(3);
    std::partial_sort_copy(src.begin(), src.end(), dst.begin(), dst.end());
    // dst = {1,3,5}，src 不变
}
```

### (2) 谓词与投影

`ranges::partial_sort` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}, {"Carol", 35}};

// 按年龄取前 2 年轻
std::ranges::partial_sort(people, people.begin() + 2, {}, &Person::age);
```

### (3) 执行策略

`partial_sort` 和 `partial_sort_copy` 支持 C++17 执行策略。

## 4. 注意事项

- **后面元素的顺序不确定**：`partial_sort` 只保证前 \\( K \\) 个有序，其余元素的顺序是未指定的。
- **`middle` 必须在范围内**：`middle` 可以等于 `first`（无操作）或 `last`（等价于完整排序）。
- **比较器必须严格弱序**。
- **与 `nth_element` 的选择**：只需要「第 K 小的元素」用 `nth_element`（\\( O(n) \\)）；需要「前 K 个有序」用 `partial_sort`。
- **`partial_sort_copy` 的目标大小决定 \\( K \\)**：目标范围多大就复制多少个。

## 5. 相关算法

- [sort / stable_sort](./Sort.md)：完整排序
- [nth_element](./Nth_element.md)：只定位第 \\( N \\) 个元素
- [堆操作](./Heap.md)：`partial_sort` 的底层实现
- [is_sorted](./Is_sorted.md)：检查是否已排序
