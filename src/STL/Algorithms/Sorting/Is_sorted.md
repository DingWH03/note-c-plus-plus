# std::is_sorted / std::is_sorted_until

这两个算法检查范围是否已排序：

- `is_sorted`：判断整个范围是否已排序
- `is_sorted_until`：返回最大的已排序前缀的末尾位置

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt>
constexpr bool is_sorted(ForwardIt first, ForwardIt last);

template<class ForwardIt, class Compare>
constexpr bool is_sorted(ForwardIt first, ForwardIt last, Compare comp);

template<class ForwardIt>
constexpr ForwardIt is_sorted_until(ForwardIt first, ForwardIt last);
```

- **迭代器要求**：`ForwardIterator`。
- **复杂度**：`is_sorted` 至多 \\( n-1 \\) 次比较；`is_sorted_until` 至多 \\( n-1 \\) 次比较。
- **返回值**：`is_sorted` 返回 `bool`；`is_sorted_until` 返回第一个破坏有序性的元素位置。
- **空范围或单元素范围**：返回 `true` / `last`。

`is_sorted` 等价于 `is_sorted_until(first, last) == last`。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v1{1, 2, 3, 4, 5};
    std::vector<int> v2{1, 3, 2, 4, 5};

    std::cout << std::boolalpha
              << std::is_sorted(v1.begin(), v1.end()) << '\n'    // true
              << std::is_sorted(v2.begin(), v2.end()) << '\n';   // false

    // 找出已排序前缀的末尾
    auto it = std::is_sorted_until(v2.begin(), v2.end());
    std::cout << "已排序前缀长度: " << (it - v2.begin()) << ", 破坏点值: " << *it << '\n';
    // 已排序前缀长度: 2, 破坏点值: 2

    // 检查降序
    std::vector<int> v3{5, 4, 3, 2, 1};
    std::cout << std::is_sorted(v3.begin(), v3.end(), std::greater<>{}) << '\n';  // true
}
```

### (2) 谓词与投影

`ranges::is_sorted` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 25}, {"Bob", 30}};

bool ok = std::ranges::is_sorted(people, {}, &Person::age);   // true
```

### (3) 执行策略

`is_sorted` 和 `is_sorted_until` 支持 C++17 执行策略。

## 4. 注意事项

- **比较器必须与排序时一致**：用 `std::greater<>` 排序后，检查时也要用 `std::greater<>`。
- **`is_sorted` 对相等元素返回 `true`**：它检查的是非降序（`!comp(next, prev)`）。
- **调试利器**：在调用 `binary_search` 等要求有序的算法前，可以用 `is_sorted` 做断言检查。
- **`is_sorted_until` 的返回值**：指向第一个「比前一个元素小」的元素，可用于定位数据异常位置。

## 5. 相关算法

- [sort / stable_sort](./Sort.md)：排序
- [二分查找](./Binary_search.md)：要求范围有序
- [is_partitioned](./Is_partitioned.md)：判断是否已分区
- [is_heap](./Heap.md)：判断是否为堆
