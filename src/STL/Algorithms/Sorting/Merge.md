# std::merge / std::inplace_merge

这两个算法合并两个**已排序**范围：

- `merge`：把两个有序范围合并到第三个输出范围
- `inplace_merge`：把同一个范围内两个**连续**的有序段原地合并

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt1, class InputIt2, class OutputIt>
constexpr OutputIt merge(InputIt1 first1, InputIt1 last1,
                         InputIt2 first2, InputIt2 last2, OutputIt d_first);

template<class BidirIt>
void inplace_merge(BidirIt first, BidirIt middle, BidirIt last);
```

- **迭代器要求**：`merge` 输入 `InputIterator`、输出 `OutputIterator`；`inplace_merge` 需要 `BidirectionalIterator`。
- **复杂度**：
  - `merge`：至多 \\( N_1 + N_2 - 1 \\) 次比较
  - `inplace_merge`：有足够额外内存时 \\( O(n) \\)，否则 \\( O(n \log n) \\)
- **返回值**：`merge` 返回输出范围末尾；`inplace_merge` 返回 `void`。

`merge` 是**稳定**的：相等元素中，第一个范围的元素排在前面。

`inplace_merge` 常用于**归并排序**的实现：先递归排序两半，再原地合并。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> a{1, 3, 5, 7};
    std::vector<int> b{2, 4, 6, 8};

    // 合并到新容器
    std::vector<int> merged;
    std::merge(a.begin(), a.end(), b.begin(), b.end(),
               std::back_inserter(merged));
    // merged = {1,2,3,4,5,6,7,8}

    // 原地合并：同一容器内两个有序段
    std::vector<int> v{1, 3, 5, 2, 4, 6};
    //              [1,3,5] [2,4,6] 两段各自有序
    std::inplace_merge(v.begin(), v.begin() + 3, v.end());
    // v = {1,2,3,4,5,6}
}
```

### (2) 谓词与投影

可以传入自定义比较器：

```c++
std::vector<int> a{7, 5, 3, 1};   // 降序
std::vector<int> b{8, 6, 4, 2};   // 降序

std::vector<int> m;
std::merge(a.begin(), a.end(), b.begin(), b.end(),
           std::back_inserter(m), std::greater<>{});
// m = {8,7,6,5,4,3,2,1}
```

`ranges::merge` 支持投影。

### (3) 执行策略

`merge` 和 `inplace_merge` 支持 C++17 执行策略。

## 4. 注意事项

- **必须先排序**：两个输入范围都必须已按同一比较器排序。
- **`merge` 是稳定的**：相等元素保持第一个范围在前的顺序。
- **`inplace_merge` 的两段必须连续**：范围是 `[first, middle)` 和 `[middle, last)`。
- **输出范围必须足够大**：`merge` 不检查大小。
- **`list::merge` 是成员函数**：`std::list` 有成员 `merge`，会把另一个链表**节点**搬过来，不拷贝元素，效率更高。
- **`inplace_merge` 的内存开销**：实现可能需要临时缓冲区，内存不足时退化为 \\( O(n \log n) \\)。

## 5. 相关算法

- [sort / stable_sort](./Sort.md)：排序
- [集合操作](./Set_operations.md)：并集、交集等
- [unique](../Modifying/Unique.md)：去除重复
- [binary_search](./Binary_search.md)：在有序范围上查找
