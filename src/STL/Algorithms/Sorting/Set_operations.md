# std::includes / set_union / set_intersection / set_difference / set_symmetric_difference

这一组算法在**已排序**范围上执行集合运算。它们都要求输入范围已排序，输出结果也是有序的。

| 算法 | 含义 |
| :--- | :--- |
| `includes` | 判断一个序列是否为另一个的子序列 |
| `set_union` | 并集 |
| `set_intersection` | 交集 |
| `set_difference` | 差集（第一个有而第二个没有） |
| `set_symmetric_difference` | 对称差集（并集减去交集） |

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt1, class InputIt2, class OutputIt>
constexpr OutputIt set_union(InputIt1 first1, InputIt1 last1,
                             InputIt2 first2, InputIt2 last2, OutputIt d_first);

template<class InputIt1, class InputIt2, class OutputIt>
constexpr OutputIt set_intersection(InputIt1 first1, InputIt1 last1,
                                    InputIt2 first2, InputIt2 last2, OutputIt d_first);
```

- **迭代器要求**：输入 `InputIterator`，输出 `OutputIterator`。
- **复杂度**：至多 \\( 2(N_1 + N_2) - 1 \\) 次比较。
- **返回值**：指向输出范围末尾的迭代器（`includes` 返回 `bool`）。

这些算法基于**归并**思路：同时遍历两个有序范围，按比较结果决定输出。重复元素按「多重集」语义处理——如果某元素在第一个范围出现 \\( m \\) 次、第二个出现 \\( n \\) 次，则：

- 并集中出现 `max(m, n)` 次
- 交集中出现 `min(m, n)` 次
- 差集中出现 `max(m - n, 0)` 次

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> a{1, 2, 3, 4, 5};
    std::vector<int> b{3, 4, 5, 6, 7};

    std::vector<int> u, i, d, sd;

    std::set_union(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(u));
    // u = {1,2,3,4,5,6,7}

    std::set_intersection(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(i));
    // i = {3,4,5}

    std::set_difference(a.begin(), a.end(), b.begin(), b.end(), std::back_inserter(d));
    // d = {1,2}

    std::set_symmetric_difference(a.begin(), a.end(), b.begin(), b.end(),
                                  std::back_inserter(sd));
    // sd = {1,2,6,7}

    // includes：判断 a 是否包含 {2,4}
    std::vector<int> sub{2, 4};
    std::cout << std::boolalpha
              << std::includes(a.begin(), a.end(), sub.begin(), sub.end()) << '\n';  // true
}
```

### (2) 谓词与投影

可以传入自定义比较器（必须与排序时一致）：

```c++
std::vector<int> a{5, 4, 3, 2, 1};   // 降序
std::vector<int> b{7, 6, 5, 4, 3};   // 降序

std::vector<int> u;
std::set_union(a.begin(), a.end(), b.begin(), b.end(),
               std::back_inserter(u), std::greater<>{});
// u = {7,6,5,4,3,2,1}
```

`ranges::set_union` 等支持投影。

### (3) 执行策略

这些算法**不支持**执行策略。集合运算需要顺序归并两个范围。

## 4. 注意事项

- **必须先排序**：这是硬性前提。未排序会导致错误结果或未定义行为。
- **比较器必须一致**：排序和集合运算必须用同一个比较器。
- **多重集语义**：重复元素按上述规则处理，不是简单的「去重」。
- **输出范围必须足够大**：用 `back_inserter` 最安全。
- **`includes` 判断的是子序列**：要求第二个范围的每个元素都在第一个范围中，且重复次数不超过。
- **容器版本**：`std::set` 等关联容器也支持集合运算，但通过迭代器插入更麻烦，通常直接用这些算法。

## 5. 相关算法

- [merge / inplace_merge](./Merge.md)：合并两个有序范围
- [sort / stable_sort](./Sort.md)：排序（集合运算的前提）
- [binary_search](./Binary_search.md)：在有序范围上查找
- [unique](../Modifying/Unique.md)：去除重复元素
