# std::is_partitioned / std::partition_point

这两个算法处理「已分区」的范围：

- `is_partitioned`：判断范围是否已按谓词分区
- `partition_point`：返回已分区范围的**分区点**（第一个不满足谓词的元素）

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class UnaryPred>
constexpr bool is_partitioned(InputIt first, InputIt last, UnaryPred p);

template<class ForwardIt, class UnaryPred>
constexpr ForwardIt partition_point(ForwardIt first, ForwardIt last, UnaryPred p);
```

- **迭代器要求**：`is_partitioned` 需要 `InputIterator`；`partition_point` 需要 `ForwardIterator`。
- **复杂度**：`is_partitioned` 至多 \\( O(n) \\)；`partition_point` 为 \\( O(\log n) \\) 次谓词调用（二分查找）。
- **返回值**：`is_partitioned` 返回 `bool`；`partition_point` 返回分区点迭代器。

**重要**：`partition_point` 要求范围**已经分区**，否则行为未定义。它通过二分查找定位分区点，因此比线性扫描快。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{2, 4, 6, 1, 3, 5};

    // 判断是否已分区（偶数在前）
    bool ok = std::is_partitioned(v.begin(), v.end(),
                                  [](int x) { return x % 2 == 0; });
    std::cout << std::boolalpha << ok << '\n';   // true

    // 定位分区点
    auto it = std::partition_point(v.begin(), v.end(),
                                   [](int x) { return x % 2 == 0; });
    std::cout << "分区点索引: " << (it - v.begin()) << ", 值: " << *it << '\n';
    // 分区点索引: 3, 值: 1
}
```

### (2) 谓词与投影

`ranges::is_partitioned` 和 `ranges::partition_point` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Carol", 25}, {"Bob", 17}};

auto it = std::ranges::partition_point(people,
                                       [](int age) { return age >= 18; },
                                       &Person::age);
```

### (3) 执行策略

`is_partitioned` 支持 C++17 执行策略；`partition_point` 不支持（二分查找需要顺序访问）。

## 4. 注意事项

- **`partition_point` 要求已分区**：这是硬性前提，违反会导致未定义行为。
- **分区点可能等于 `last`**：如果所有元素都满足谓词，返回 `last`。
- **`is_partitioned` 对空范围返回 `true`**。
- **与 `lower_bound` 的关系**：`partition_point` 是更通用的二分查找，`lower_bound` 相当于谓词为 `*it < value` 的特例。

## 5. 相关算法

- [partition / stable_partition](./Partition.md)：执行分区
- [partition_copy](./Partition_copy.md)：复制时分区
- [二分查找](./Binary_search.md)：在有序范围上查找
- [is_sorted](./Is_sorted.md)：判断是否已排序
