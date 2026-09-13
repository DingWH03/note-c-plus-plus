# std::partition / std::stable_partition

这两个算法按谓词把范围内的元素**划分为两组**：满足谓词的排在前，不满足的排在后。

- `partition`：不保证两组内部的相对顺序
- `stable_partition`：保持两组内部的相对顺序

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt, class UnaryPred>
constexpr ForwardIt partition(ForwardIt first, ForwardIt last, UnaryPred p);

template<class BidirIt, class UnaryPred>
constexpr BidirIt stable_partition(BidirIt first, BidirIt last, UnaryPred p);
```

- **迭代器要求**：`partition` 需要 `ForwardIterator`；`stable_partition` 需要 `BidirectionalIterator`。
- **复杂度**：`partition` 恰好 \\( n \\) 次谓词调用和至多 \\( n \\) 次交换；`stable_partition` 在有足够内存时为 \\( O(n) \\)，否则为 \\( O(n \log n) \\)。
- **返回值**：指向第二组（不满足谓词）的第一个元素，即**分区点**。

`partition` 的经典实现是双指针相向扫描并交换。`stable_partition` 则需要额外内存或递归分治来保持顺序。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5, 6};

    // 偶数在前，奇数在后
    auto it = std::partition(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
    // v 可能是 {6,2,4,3,5,1}
    std::cout << "分区点索引: " << (it - v.begin()) << '\n';   // 3

    // stable_partition 保持相对顺序
    std::vector<int> v2{1, 2, 3, 4, 5, 6};
    std::stable_partition(v2.begin(), v2.end(), [](int x) { return x % 2 == 0; });
    // v2 = {2,4,6,1,3,5}
}
```

### (2) 谓词与投影

`ranges::partition` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 17}, {"Carol", 25}};

std::ranges::partition(people, [](int age) { return age >= 18; }, &Person::age);
// 成年人排在前面
```

### (3) 执行策略

`partition` 支持 C++17 执行策略；`stable_partition` 不支持（需要保持顺序）。

## 4. 注意事项

- **分区点很重要**：返回值把范围分成 `[first, it)` 和 `[it, last)` 两部分，常用于后续处理。
- **`partition` 不保证顺序**：如果需要保持相对顺序，必须用 `stable_partition`，但代价更高。
- **谓词不应有副作用**。
- **配合 `partition_point` 使用**：对已分区范围可以用 `partition_point` 二分查找分区点。
- **`nth_element` 也是一种分区**：它按第 N 个元素分区，但额外保证该元素就位。

## 5. 相关算法

- [is_partitioned / partition_point](./Is_partitioned.md)：判断分区或定位分区点
- [partition_copy](./Partition_copy.md)：复制时分区
- [nth_element](./Nth_element.md)：部分排序
- [sort / stable_sort](./Sort.md)：完全排序
- [remove / remove_if](../Modifying/Remove.md)：逻辑移除
