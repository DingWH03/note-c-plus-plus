# std::next_permutation / prev_permutation / is_permutation

这三个算法处理**排列**：

- `next_permutation`：把范围变换为下一个字典序更大的排列
- `prev_permutation`：把范围变换为下一个字典序更小的排列
- `is_permutation`：判断一个序列是否为另一个序列的排列（C++11）

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class BidirIt>
constexpr bool next_permutation(BidirIt first, BidirIt last);

template<class BidirIt>
constexpr bool prev_permutation(BidirIt first, BidirIt last);

template<class ForwardIt1, class ForwardIt2>
constexpr bool is_permutation(ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2);
```

- **迭代器要求**：`next_permutation` / `prev_permutation` 需要 `BidirectionalIterator`；`is_permutation` 需要 `ForwardIterator`。
- **复杂度**：排列操作至多 \\( n/2 \\) 次交换；`is_permutation` 最坏 \\( O(n^2) \\) 次比较（若可用则用 \\( O(n \log n) \\) 的排序方法）。
- **返回值**：`next_permutation` 返回 `true` 表示还有下一个排列；如果已经是最后一个排列，则把范围重置为**第一个排列**并返回 `false`。

`next_permutation` 的算法步骤：

1. 从右往左找第一个满足 `*i < *(i+1)` 的位置 `i`
2. 从右往左找第一个满足 `*i < *j` 的位置 `j`
3. 交换 `*i` 和 `*j`
4. 反转 `[i+1, last)` 区间

要生成**所有**排列，范围必须先按升序排序。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    // 遍历所有排列（必须先排序）
    std::vector<int> v{1, 2, 3};
    do {
        for (int x : v) std::cout << x;
        std::cout << ' ';
    } while (std::next_permutation(v.begin(), v.end()));
    // 输出: 123 132 213 231 312 321

    // prev_permutation：需要先降序排序
    std::vector<int> v2{3, 2, 1};
    do {
        for (int x : v2) std::cout << x;
        std::cout << ' ';
    } while (std::prev_permutation(v2.begin(), v2.end()));
    // 输出: 321 312 231 213 132 123

    // is_permutation
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{3, 1, 2};
    std::cout << std::boolalpha
              << std::is_permutation(a.begin(), a.end(), b.begin(), b.end()) << '\n';  // true
}
```

### (2) 谓词与投影

可以传入自定义比较器：

```c++
std::vector<int> v{1, 2, 3};
// 按降序生成排列
std::sort(v.begin(), v.end(), std::greater<>{});
do {
    // ...
} while (std::next_permutation(v.begin(), v.end(), std::greater<>{}));
```

### (3) 执行策略

这三个算法**不支持**执行策略。

## 4. 注意事项

- **必须先排序**：要生成所有排列，初始范围必须有序（`next_permutation` 需升序，`prev_permutation` 需降序）。
- **重复元素**：有重复元素时，`next_permutation` 会自动跳过重复的排列，只生成**不同**的排列。
- **返回 `false` 时范围被重置**：`next_permutation` 在最后一个排列时返回 `false`，但会把范围改为第一个排列。用 `do-while` 循环是正确的写法。
- **`is_permutation` 的复杂度**：最坏 \\( O(n^2) \\)，如果范围很大且可排序，先排序再比较可能更快。
- **排列数量爆炸**：\\( n! \\) 增长极快，只适用于小规模数据。
- **`is_permutation` 的 C++14 双范围版本**：可以处理长度不同的范围。

## 5. 相关算法

- [sort / stable_sort](./Sort.md)：排序（排列的前提）
- [is_sorted](./Is_sorted.md)：检查是否已排序
- [shuffle](../Modifying/Shuffle.md)：随机打乱
- [equal](../NonModifying/Equal.md)：判断范围相同
