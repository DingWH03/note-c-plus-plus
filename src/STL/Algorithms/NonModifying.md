# 非修改序列操作 (Non-modifying sequence operations)

非修改序列操作只读取范围内的元素，不会改变元素的值，也不会改变容器的大小。它们主要用于**查找、计数、比较和遍历**。

**主要头文件：**

- **`<algorithm>`**：包含绝大多数经典的非修改序列操作。
- **`<ranges>`**：包含所有 `ranges::` 版本的算法。

## 算法总览

| 算法 | C++ 版本 | 头文件 | 描述 |
| :--- | :--- | :--- | :--- |
| [for_each](./NonModifying/For_each.md) | | `<algorithm>` | 对范围内的元素应用一个一元函数对象 |
| [for_each_n](./NonModifying/For_each_n.md) | (C++17) | `<algorithm>` | 对序列的前 \\( N \\) 个元素应用一个函数对象 |
| [all_of / any_of / none_of](./NonModifying/All_any_none_of.md) | (C++11) | `<algorithm>` | 检查谓词对范围内所有/任一/没有元素是否为真 |
| [find / find_if / find_if_not](./NonModifying/Find.md) | (C++98) / (C++11) | `<algorithm>` | 查找第一个满足条件的元素 |
| [find_last](./NonModifying/Find_last.md) | (C++23) | `<algorithm>` | 查找最后一个满足条件的元素 |
| [find_end](./NonModifying/Find_end.md) | | `<algorithm>` | 查找某一范围内的最后一个子序列 |
| [find_first_of](./NonModifying/Find_first_of.md) | | `<algorithm>` | 搜索一组元素中的任意一个 |
| [adjacent_find](./NonModifying/Adjacent_find.md) | | `<algorithm>` | 查找第一对相邻且相等（或满足谓词）的元素 |
| [count / count_if](./NonModifying/Count.md) | | `<algorithm>` | 返回满足条件的元素数量 |
| [mismatch](./NonModifying/Mismatch.md) | (C++14) | `<algorithm>` | 查找两个范围第一次不同的位置 |
| [equal](./NonModifying/Equal.md) | (C++14) | `<algorithm>` | 判断两组元素是否相同 |
| [search / search_n](./NonModifying/Search.md) | | `<algorithm>` | 搜索子范围或连续 N 个元素的首次出现 |
| [contains](./NonModifying/Contains.md) | (C++23) | `<algorithm>` | 检查范围是否包含给定元素或子范围 |
| [starts_with / ends_with](./NonModifying/Starts_ends_with.md) | (C++23) | `<algorithm>` | 检查范围是否以另一个范围开始或结束 |
| [fold](./NonModifying/Fold.md) | (C++23) | `<algorithm>` | 对范围进行左折叠或右折叠 |

> **说明：** 表中「C++ 版本」列标注的是该算法（或 `ranges::` 版本）首次引入的标准版本，留空表示自 C++98 起即存在。

## 共性说明

- **复杂度**：除 `find_end`、`search`、`search_n` 等子序列搜索算法外，绝大多数非修改序列操作都是线性复杂度 \\(O(n)\\)。
- **谓词与投影**：经典版本接受一元谓词（`Pred`），`ranges::` 版本额外支持**投影**（`Proj`），可以只对元素的某个成员进行比较。
- **返回类型**：`ranges::` 版本通常返回结构体（如 `ranges::in_in_result`），一次性给出多个有用信息。
