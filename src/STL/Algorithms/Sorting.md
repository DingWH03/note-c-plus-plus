# 排序及相关操作 (Sorting and related operations)

这一组算法围绕**排序、分区、查找与堆**展开。其中二分查找、集合操作、合并操作都要求输入范围**已经有序**，否则行为未定义。

**主要头文件：**

- **`<algorithm>`**：包含绝大多数经典的排序及相关操作。
- **`<ranges>`**：包含所有 `ranges::` 版本的算法。

## 算法总览

### 分区操作

| 算法 | C++ 版本 | 描述 |
| :--- | :--- | :--- |
| [is_partitioned / partition_point](./Sorting/Is_partitioned.md) | (C++11) | 判断范围是否已分区，或定位分区点 |
| [partition / stable_partition](./Sorting/Partition.md) | | 将范围按谓词划分为两组 |
| [partition_copy](./Sorting/Partition_copy.md) | (C++11) | 复制范围并划分为两组 |

### 排序操作

| 算法 | C++ 版本 | 描述 |
| :--- | :--- | :--- |
| [sort / stable_sort](./Sorting/Sort.md) | | 对范围排序，可选择是否稳定 |
| [partial_sort](./Sorting/Partial_sort.md) | | 排序范围的前 \\( N \\) 个元素 |
| [is_sorted](./Sorting/Is_sorted.md) | (C++11) | 检查范围是否已排序 |
| [nth_element](./Sorting/Nth_element.md) | | 部分排序并定位第 \\( N \\) 个元素 |

### 二分查找操作

| 算法 | C++ 版本 | 描述 |
| :--- | :--- | :--- |
| [lower_bound / upper_bound](./Sorting/Binary_search.md) | | 返回第一个不小于 / 大于给定值的元素 |
| [equal_range / binary_search](./Sorting/Binary_search.md) | | 返回匹配范围或判断元素是否存在 |

### 集合操作

| 算法 | C++ 版本 | 描述 |
| :--- | :--- | :--- |
| [includes](./Sorting/Set_operations.md) | | 判断一个序列是否为另一个的子序列 |
| [set_union / set_intersection](./Sorting/Set_operations.md) | | 计算并集 / 交集 |
| [set_difference / set_symmetric_difference](./Sorting/Set_operations.md) | | 计算差集 / 对称差集 |

### 合并与堆操作

| 算法 | C++ 版本 | 描述 |
| :--- | :--- | :--- |
| [merge / inplace_merge](./Sorting/Merge.md) | | 合并两个已排序范围 |
| [make_heap / push_heap / pop_heap / sort_heap / is_heap](./Sorting/Heap.md) | (C++11) | 维护最大堆及堆与有序序列的转换 |

### 最值与比较操作

| 算法 | C++ 版本 | 描述 |
| :--- | :--- | :--- |
| [min / max / minmax](./Sorting/Min_max.md) | (C++11) | 返回给定值中的较小者 / 较大者 / 两者 |
| [min_element / max_element / minmax_element](./Sorting/Min_max.md) | (C++11) | 返回范围中的最小 / 最大 / 两者 |
| [clamp](./Sorting/Min_max.md) | (C++17) | 将值钳制在一对边界之间 |
| [lexicographical_compare](./Sorting/Lexicographical_compare.md) | (C++20) | 按字典序（或三路）比较两个范围 |
| [next_permutation / prev_permutation / is_permutation](./Sorting/Permutation.md) | (C++11) | 生成排列或判断排列关系 |

> **说明：** 表中「C++ 版本」列标注的是该算法（或 `ranges::` 版本）首次引入的标准版本，留空表示自 C++98 起即存在。

## 共性说明

- **有序前提**：二分查找、集合操作、`merge`、`includes` 都要求输入范围已按同一比较器排序，否则结果未定义。
- **稳定性**：`stable_sort`、`stable_partition` 保持相等元素的相对顺序，代价是额外的内存或时间开销。
- **堆的本质**：堆操作默认维护**最大堆**，配合 `std::greater<>` 可得到最小堆；`priority_queue` 正是基于这组算法实现的。
