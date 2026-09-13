# 数值操作 (Numeric operations)

数值算法定义在 `<numeric>` 中，用于对范围进行求和、折叠、扫描等数值计算。C++17 引入了支持执行策略的并行版本（`reduce`、`scan` 等），它们允许以**任意顺序**组合元素，从而获得更好的并行性能。

**主要头文件：**

- **`<numeric>`**：包含所有传统的数值算法和 C++17 的并行数值算法。
- **`<ranges>`**：包含 `ranges::iota`。

## 算法总览

| 算法 | C++ 版本 | 头文件 | 描述 |
| :--- | :--- | :--- | :--- |
| [iota](./Numeric/Iota.md) | (C++11) | `<numeric>` | 用递增的起始值填充一个范围 |
| [accumulate](./Numeric/Accumulate.md) | | `<numeric>` | 对元素范围求和或折叠 |
| [inner_product](./Numeric/Inner_product.md) | | `<numeric>` | 计算两个元素范围的内积 |
| [adjacent_difference](./Numeric/Adjacent_difference.md) | | `<numeric>` | 计算相邻元素之间的差值 |
| [partial_sum](./Numeric/Partial_sum.md) | | `<numeric>` | 计算元素范围的部分和 |
| [reduce](./Numeric/Reduce.md) | (C++17) | `<numeric>` | 类似于 `accumulate`，但允许无序归约 |
| [exclusive_scan / inclusive_scan](./Numeric/Scan.md) | (C++17) | `<numeric>` | 计算独占扫描或包含扫描 |
| [transform_reduce](./Numeric/Transform_reduce.md) | (C++17) | `<numeric>` | 先应用可调用对象，再无序归约 |
| [transform_exclusive_scan / transform_inclusive_scan](./Numeric/Transform_scan.md) | (C++17) | `<numeric>` | 先应用可调用对象，再计算扫描 |

> **说明：** 表中「C++ 版本」列标注的是该算法（或 `ranges::` 版本）首次引入的标准版本，留空表示自 C++98 起即存在。

## 共性说明

- **`accumulate` vs `reduce`**：`accumulate` 严格按顺序计算，因此要求运算满足结合律即可；`reduce` 允许任意顺序组合，因此运算必须**满足交换律和结合律**，否则结果不确定。
- **扫描的差异**：`partial_sum` 属于包含扫描；`exclusive_scan` 的第 \\( N \\) 个结果不包含第 \\( N \\) 个输入元素；`inclusive_scan` 则包含。
- **并行支持**：`reduce`、`exclusive_scan`、`inclusive_scan`、`transform_reduce`、`transform_*_scan` 支持执行策略；`accumulate`、`inner_product`、`partial_sum`、`adjacent_difference` 不支持。
- **`ranges::` 版本**：目前只有 `ranges::iota`（C++23），其余数值算法暂无 Ranges 版本。
