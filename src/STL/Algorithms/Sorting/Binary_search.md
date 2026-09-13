# std::lower_bound / upper_bound / equal_range / binary_search

这四个算法在**已排序**范围上进行二分查找，复杂度均为 \\( O(\log n) \\)。

| 算法 | 含义 |
| :--- | :--- |
| `lower_bound` | 返回第一个**不小于**给定值的元素位置 |
| `upper_bound` | 返回第一个**大于**给定值的元素位置 |
| `equal_range` | 返回 `{lower_bound, upper_bound}` 构成的区间 |
| `binary_search` | 判断元素是否存在，返回 `bool` |

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt, class T>
constexpr ForwardIt lower_bound(ForwardIt first, ForwardIt last, const T& value);

template<class ForwardIt, class T>
constexpr ForwardIt upper_bound(ForwardIt first, ForwardIt last, const T& value);

template<class ForwardIt, class T>
constexpr std::pair<ForwardIt, ForwardIt>
    equal_range(ForwardIt first, ForwardIt last, const T& value);

template<class ForwardIt, class T>
constexpr bool binary_search(ForwardIt first, ForwardIt last, const T& value);
```

- **迭代器要求**：`ForwardIterator`（实际用随机访问迭代器才能达到 \\( O(\log n) \\) 次**迭代器移动**，用 forward 迭代器比较次数仍是 \\( O(\log n) \\) 但移动是 \\( O(n) \\)）。
- **复杂度**：\\( O(\log n) \\) 次比较。
- **前提**：范围必须**已按同一比较器排序**，否则行为未定义。

四个算法的关系：

```
已排序范围: 1 2 2 2 3 4 5
                ^     ^
        lower_bound   upper_bound   （查找值 2）
        equal_range = [lower_bound, upper_bound)
```

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 2, 2, 3, 4, 5};

    // lower_bound：第一个 >= 2 的位置
    auto lb = std::lower_bound(v.begin(), v.end(), 2);
    std::cout << "lower_bound 索引: " << (lb - v.begin()) << '\n';   // 1

    // upper_bound：第一个 > 2 的位置
    auto ub = std::upper_bound(v.begin(), v.end(), 2);
    std::cout << "upper_bound 索引: " << (ub - v.begin()) << '\n';   // 4

    // equal_range：等于 2 的区间
    auto [first, last] = std::equal_range(v.begin(), v.end(), 2);
    std::cout << "2 的个数: " << (last - first) << '\n';   // 3

    // binary_search：判断存在性
    std::cout << std::boolalpha
              << std::binary_search(v.begin(), v.end(), 3) << '\n'     // true
              << std::binary_search(v.begin(), v.end(), 9) << '\n';    // false

    // 降序范围：用 greater
    std::vector<int> d{5, 4, 3, 2, 1};
    auto it = std::lower_bound(d.begin(), d.end(), 3, std::greater<>{});
    std::cout << "降序 lower_bound 索引: " << (it - d.begin()) << '\n';   // 2
}
```

### (2) 谓词与投影

`ranges::` 版本支持投影，可以直接按成员查找：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Bob", 25}, {"Alice", 30}, {"Carol", 35}};

// 按年龄查找
auto it = std::ranges::lower_bound(people, 30, {}, &Person::age);
std::cout << it->name;   // Alice
```

### (3) 执行策略

这四个算法**不支持**执行策略。二分查找是顺序依赖的过程，无法并行化。

## 4. 注意事项

- **必须先排序**：这是硬性前提，违反会导致未定义行为。建议用 `is_sorted` 做断言。
- **比较器必须一致**：排序和查找必须使用同一个比较器（如都用 `std::greater<>`）。
- **`lower_bound` 与 `upper_bound` 的区别**：`lower_bound` 找 `>=`，`upper_bound` 找 `>`。查找单个存在的值时两者只差 1。
- **`binary_search` 只返回 `bool`**：需要位置时用 `lower_bound`。
- **关联容器有成员版本**：`std::set`、`std::map` 等有成员函数 `lower_bound`、`upper_bound`、`equal_range`，性能更好。
- **C++20 的 `contains`**：`std::set::contains` 等成员函数比 `binary_search` 更直观。

## 5. 相关算法

- [sort / stable_sort](./Sort.md)：排序（二分查找的前提）
- [is_sorted](./Is_sorted.md)：检查是否已排序
- [find / find_if](../NonModifying/Find.md)：线性查找
- [contains](../NonModifying/Contains.md)：判断元素是否存在
- [集合操作](./Set_operations.md)：有序范围上的集合运算
