# Algorithms

C++ 标准库中的算法 (Algorithms) 是一组强大的函数模板，用于对容器或其他范围（`[first, last)`）内的元素进行搜索、排序、计数、修改等操作。自 C++20 以来，**Ranges (约束算法)** 的引入极大地简化了算法的使用，并增强了其通用性。C++17 则引入了**执行策略**，允许算法进行并行或乱序执行以提升性能。

## C++20 约束算法 (Constrained Algorithms / Ranges)

**Ranges** 是 C++20 引入的一项革命性特性，它极大地简化了 STL 算法的使用，并增强了它们的组合性。

### 核心概念与优势

| 特性 | 优势和示例 |
| :--- | :--- |
| **单一 Range 参数** | 您不再需要手动传递 `begin()` 和 `end()` 迭代器。只需将整个容器（或 Range 视图）作为**一个参数**传递给算法即可。 |
| **Projections (投影)** | **简化复杂对象操作。** 算法可以直接操作容器内元素的某个成员变量或计算结果，而无需使用复杂的 Lambda 表达式。例如，对一个 `vector<Person>` 按照 `Person::age` 成员进行排序。 |
| **返回类型增强** | **提供所有有用的信息。** 大多数 `std::ranges::` 算法不再只返回一个迭代器，而是返回一个包含所有相关结果的**结构体**（如 `std::ranges::sort_result`）。例如，`ranges::copy` 会返回输入范围和输出范围的结束迭代器，方便后续操作。 |
| **更强的约束** | 它们是“约束算法”，这意味着它们使用 C++20 的 **Concepts** 确保传入的迭代器和 Range 满足算法所需的最低要求，从而提供**更清晰的编译错误**。 |

**示例:**

```cpp
std::vector<int> v{7, 1, 4, 0, -1};

// 经典算法：需要显式传入begin()和end()
std::sort(v.begin(), v.end());

// C++20 Ranges 算法：更简洁
std::ranges::sort(v);
```

-----

## C++17/C++20 执行策略 (Execution Policies)

**执行策略** 是 C++17 引入的功能，它允许程序员通过向算法传递一个策略对象，来指示算法如何执行——是串行、并行还是乱序执行，从而在多核 CPU 上实现**性能优化**。

### 核心策略类型

| 策略类型 | 全局对象 | C++ 版本 | 描述 |
| :--- | :--- | :--- | :--- |
| **`sequenced_policy`** | `seq` | C++17 | **序列化执行**：算法按顺序执行，不会使用并行或乱序操作。这是所有算法的默认行为。 |
| **`parallel_policy`** | `par` | C++17 | **并行执行**：算法可以在不同的线程中并发执行。它保证了并发性，但不保证指令的顺序。 |
| **`parallel_unsequenced_policy`** | `par_unseq` | C++17 | **并行且乱序执行**：结合了并行和乱序执行，允许并发执行，同时允许编译器进行**矢量化 (vectorization)** 优化。 |
| **`unsequenced_policy`** | `unseq` | C++20 | **乱序执行**：**不保证并行**，但允许编译器对单个线程中的操作进行乱序处理（矢量化），以提高性能。 |

**头文件：** 所有执行策略都定义在 `<execution>` 头文件中，并位于 `std::execution` 命名空间下（尽管全局对象如 `std::par` 在 `std` 命名空间）。

**示例:**

```cpp
#include <algorithm>
#include <execution>
#include <vector>

std::vector<int> data = ...;

// 使用并行策略对数据进行排序，以利用多核CPU
std::sort(std::execution::par, data.begin(), data.end());

// C++20 Ranges 版本也支持策略（如果算法有重载）
std::ranges::sort(std::execution::par, data);
```

### 并行算法的技术限制

在使用执行策略时，有一个关键的**安全限制**需要注意：

对于大多数并行算法（除了 `std::for_each` 和 `std::for_each_n`）：

- 如果 Range 中的元素类型 \\( T \\) 满足 **Trivial Copy Construction** (`std::is_trivially_copy_constructible_v<T> == true`) 和 **Trivial Destruction** (`std::is_trivially_destructible_v<T> == true`)，**库可以对元素进行任意复制**，以便在不同的并行线程之间分发数据。

**这意味着：** 如果你的自定义对象具有非平凡（复杂）的构造函数、析构函数或资源管理，使用并行策略时需要额外小心，以确保你的操作是线程安全的。对于基本类型（如 `int`, `float`）和简单的 C 结构体，这通常不是问题。

## 一、非修改序列操作 (Non-modifying sequence operations)

**主要头文件：**

- **`<algorithm>`**：包含绝大多数经典的非修改序列操作。
- **`<ranges>`**：包含所有 `ranges::` 版本的算法。

### 批处理操作 (Batch operations)

| 批处理操作 (Batch operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **for\_each** / **ranges::for\_each** | (C++20) | `<algorithm>` / `<ranges>` | 对范围内的元素应用一个一元函数对象 |
| **for\_each\_n** / **ranges::for\_each\_n** | (C++17) / (C++20) | `<algorithm>` / `<ranges>` | 对序列的前 \\( N \\) 个元素应用一个函数对象 |

### 搜索操作 (Search operations)

| 搜索操作 (Search operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **all\_of** / **ranges::all\_of** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 检查一个谓词对范围内的**所有**元素是否为真 |
| **any\_of** / **ranges::any\_of** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 检查一个谓词对范围内的**任一**元素是否为真 |
| **none\_of** / **ranges::none\_of** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 检查一个谓词对范围内的**所有**元素是否为假 |
| **ranges::contains** | (C++23) | `<ranges>` | 检查范围是否包含给定元素 |
| **ranges::contains\_subrange** | (C++23) | `<ranges>` | 检查范围是否包含给定子范围 |
| **find** / **ranges::find** | (C++20) | `<algorithm>` / `<ranges>` | 查找满足特定条件的**第一个**元素 |
| **find\_if** / **ranges::find\_if** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 查找满足**谓词**的**第一个**元素 |
| **find\_if\_not** / **ranges::find\_if\_not** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 查找**不满足谓词**的**第一个**元素 |
| **ranges::find\_last** | (C++23) | `<ranges>` | 查找满足特定条件的**最后一个**元素 |
| **ranges::find\_last\_if** | (C++23) | `<ranges>` | 查找满足**谓词**的**最后一个**元素 |
| **ranges::find\_last\_if\_not** | (C++23) | `<ranges>` | 查找**不满足谓词**的**最后一个**元素 |
| **find\_end** / **ranges::find\_end** | (C++20) | `<algorithm>` / `<ranges>` | 查找某一范围内的**最后一个**序列 |
| **find\_first\_of** / **ranges::find\_first\_of** | (C++20) | `<algorithm>` / `<ranges>` | 搜索一组元素中的**任意一个** |
| **adjacent\_find** / **ranges::adjacent\_find** | (C++20) | `<algorithm>` / `<ranges>` | 查找**第一个**两个相邻且相等的项（或满足给定谓词） |
| **count** / **ranges::count** | (C++20) | `<algorithm>` / `<ranges>` | 返回满足特定条件的元素的**数量** |
| **count\_if** / **ranges::count\_if** | (C++20) | `<algorithm>` / `<ranges>` | 返回满足**谓词**的元素的**数量** |
| **mismatch** / **ranges::mismatch** | (C++20) | `<algorithm>` / `<ranges>` | 查找两个范围**第一次不同**的位置 |
| **equal** / **ranges::equal** | (C++20) | `<algorithm>` / `<ranges>` | 确定两组元素是否**相同** |
| **search** / **ranges::search** | (C++20) | `<algorithm>` / `<ranges>` | 搜索一个元素范围的**第一次出现** |
| **search\_n** / **ranges::search\_n** | (C++20) | `<algorithm>` / `<ranges>` | 搜索一个元素在范围中**连续出现 \\( N \\) 次**的第一次出现 |
| **ranges::starts\_with** | (C++23) | `<ranges>` | 检查一个范围是否**以另一个范围开始** |
| **ranges::ends\_with** | (C++23) | `<ranges>` | 检查一个范围是否**以另一个范围结束** |

### 折叠操作 (Fold operations)

| 折叠操作 (Fold operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **ranges::fold\_left** | (C++23) | `<ranges>` | 对一系列元素进行**左折叠** |
| **ranges::fold\_left\_first** | (C++23) | `<ranges>` | 使用**第一个元素**作为初始值对一系列元素进行**左折叠** |
| **ranges::fold\_right** | (C++23) | `<ranges>` | 对一系列元素进行**右折叠** |
| **ranges::fold\_right\_last** | (C++23) | `<ranges>` | 使用**最后一个元素**作为初始值对一系列元素进行**右折叠** |
| **ranges::fold\_left\_with\_iter** | (C++23) | `<ranges>` | 对一系列元素进行**左折叠**，并返回 (迭代器, 值) 对 |
| **ranges::fold\_left\_first\_with\_iter** | (C++23) | `<ranges>` | 使用**第一个元素**作为初始值对一系列元素进行**左折叠**，并返回 (迭代器, 可选值) 对 |

## 二、修改序列操作 (Modifying sequence operations)

**主要头文件：**

- **`<algorithm>`**：包含绝大多数经典的修改序列操作。
- **`<ranges>`**：包含所有 `ranges::` 版本的算法。
- **`<utility>`**：包含 `std::swap` 和 `std::iter_swap`。

### 复制操作 (Copy operations)

| 复制操作 (Copy operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **copy** / **ranges::copy** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 将一个元素范围**复制**到一个新位置 |
| **copy\_if** / **ranges::copy\_if** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | **有条件地复制**一个元素范围到新位置 |
| **copy\_n** / **ranges::copy\_n** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 将**指定数量**的元素复制到一个新位置 |
| **copy\_backward** / **ranges::copy\_backward** | (C++20) | `<algorithm>` / `<ranges>` | 以**向后顺序复制**一个元素范围 |
| **move** / **ranges::move** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 将一个元素范围**移动**到一个新位置 |
| **move\_backward** / **ranges::move\_backward** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 以**向后顺序移动**一个元素范围 |

### 交换操作 (Swap operations)

| 交换操作 (Swap operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **swap** | (C++11) | `<utility>` | **交换**两个对象的值 |
| **swap\_ranges** / **ranges::swap\_ranges** | (C++20) | `<algorithm>` / `<ranges>` | **交换**两个元素范围 |
| **iter\_swap** | | `<utility>` | **交换**两个迭代器所指向的元素 |

### 变换操作 (Transformation operations)

| 变换操作 (Transformation operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **transform** / **ranges::transform** | (C++20) | `<algorithm>` / `<ranges>` | 对一个元素范围**应用一个函数**，并将结果存储在目标范围中 |
| **replace** / **ranges::replace** | (C++20) | `<algorithm>` / `<ranges>` | 将所有满足特定条件的值**替换**为另一个值 |
| **replace\_if** / **ranges::replace\_if** | (C++20) | `<algorithm>` / `<ranges>` | 将所有满足**谓词**的值**替换**为另一个值 |
| **replace\_copy** / **ranges::replace\_copy** | (C++20) | `<algorithm>` / `<ranges>` | **复制**一个范围，并将满足特定条件的元素**替换**为另一个值 |
| **replace\_copy\_if** / **ranges::replace\_copy\_if** | (C++20) | `<algorithm>` / `<ranges>` | **复制**一个范围，并将满足**谓词**的元素**替换**为另一个值 |

### 生成操作 (Generation operations)

| 生成操作 (Generation operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **fill** / **ranges::fill** | (C++20) | `<algorithm>` / `<ranges>` | 将给定值**赋值**给范围内的每个元素 |
| **fill\_n** / **ranges::fill\_n** | (C++20) | `<algorithm>` / `<ranges>` | 将给定值**赋值**给范围内的 \\( N \\) 个元素 |
| **generate** / **ranges::generate** | (C++20) | `<algorithm>` / `<ranges>` | 将连续函数调用的结果**赋值**给范围内的每个元素 |
| **generate\_n** / **ranges::generate\_n** | (C++20) | `<algorithm>` / `<ranges>` | 将连续 \\( N \\) 次函数调用的结果**赋值**给范围内的 \\( N \\) 个元素 |

### 移除操作 (Removing operations)

这里的“移除”通常是**逻辑移除**，通过将未被移除的元素移动到范围的前部来实现，并返回新的逻辑尾部。

| 移除操作 (Removing operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **remove** / **ranges::remove** | (C++20) | `<algorithm>` / `<ranges>` | **移除**满足特定条件的元素（逻辑移除） |
| **remove\_if** / **ranges::remove\_if** | (C++20) | `<algorithm>` / `<ranges>` | **移除**满足**谓词**的元素（逻辑移除） |
| **remove\_copy** / **ranges::remove\_copy** | (C++20) | `<algorithm>` / `<ranges>` | **复制**一个范围，**省略**满足特定条件的元素 |
| **remove\_copy\_if** / **ranges::remove\_copy\_if** | (C++20) | `<algorithm>` / `<ranges>` | **复制**一个范围，**省略**满足**谓词**的元素 |
| **unique** / **ranges::unique** | (C++20) | `<algorithm>` / `<ranges>` | **移除**范围内的**连续重复**元素（逻辑移除） |
| **unique\_copy** / **ranges::unique\_copy** | (C++20) | `<algorithm>` / `<ranges>` | **创建**一个不含**连续重复**元素的范围**副本** |

### 顺序改变操作 (Order-changing operations)

| 顺序改变操作 (Order-changing operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **reverse** / **ranges::reverse** | (C++20) | `<algorithm>` / `<ranges>` | **反转**范围内的元素顺序 |
| **reverse\_copy** / **ranges::reverse\_copy** | (C++20) | `<algorithm>` / `<ranges>` | **创建**一个**反转**后的范围**副本** |
| **rotate** / **ranges::rotate** | (C++20) | `<algorithm>` / `<ranges>` | **旋转**范围内的元素顺序 |
| **rotate\_copy** / **ranges::rotate\_copy** | (C++20) | `<algorithm>` / `<ranges>` | **复制**并**旋转**一个元素范围 |
| **shift\_left** / **ranges::shift\_left** | (C++20) / (C++23) | `<algorithm>` / `<ranges>` | **左移**范围内的元素 |
| **shift\_right** / **ranges::shift\_right** | (C++20) / (C++23) | `<algorithm>` / `<ranges>` | **右移**范围内的元素 |
| **shuffle** / **ranges::shuffle** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | **随机重新排序**范围内的元素 |

### 采样操作 (Sampling operations)

| 采样操作 (Sampling operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **sample** / **ranges::sample** | (C++17) / (C++20) | `<algorithm>` / `<ranges>` | 从序列中**选择 \\( N \\) 个随机元素** |

## 三、排序及相关操作 (Sorting and related operations)

**主要头文件：**

- **`<algorithm>`**：包含绝大多数经典的排序及相关操作。
- **`<ranges>`**：包含所有 `ranges::` 版本的算法。

### 分区操作 (Partitioning operations)

| 分区操作 (Partitioning operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **is\_partitioned** / **ranges::is\_partitioned** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 确定范围是否被给定谓词**分区** |
| **partition** / **ranges::partition** | (C++20) | `<algorithm>` / `<ranges>` | 将一个元素范围**划分**为两组 |
| **partition\_copy** / **ranges::partition\_copy** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | **复制**一个范围，将元素**划分**为两组 |
| **stable\_partition** / **ranges::stable\_partition** | (C++20) | `<algorithm>` / `<ranges>` | 将元素**划分**为两组，同时**保持**它们的**相对顺序** |
| **partition\_point** / **ranges::partition\_point** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | **定位**一个已分区范围的**分区点** |

### 排序操作 (Sorting operations)

| 排序操作 (Sorting operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **sort** / **ranges::sort** | (C++20) | `<algorithm>` / `<ranges>` | 将一个范围**排序**为升序 |
| **stable\_sort** / **ranges::stable\_sort** | (C++20) | `<algorithm>` / `<ranges>` | **排序**一个元素范围，同时**保持**相等元素间的**相对顺序** |
| **partial\_sort** / **ranges::partial\_sort** | (C++20) | `<algorithm>` / `<ranges>` | **排序**一个范围的**前 \\( N \\) 个**元素 |
| **partial\_sort\_copy** / **ranges::partial\_sort\_copy** | (C++20) | `<algorithm>` / `<ranges>` | **复制**并**部分排序**一个元素范围 |
| **is\_sorted** / **ranges::is\_sorted** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 检查一个范围是否已**排序**为升序 |
| **is\_sorted\_until** / **ranges::is\_sorted\_until** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 查找**最大**的**已排序子范围** |
| **nth\_element** / **ranges::nth\_element** | (C++20) | `<algorithm>` / `<ranges>` | **部分排序**给定范围，确保它被给定元素**分区** |

### 二分搜索操作 (Binary search operations)

这些操作要求输入范围**必须是已排序**的。

| 二分搜索操作 (Binary search operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **lower\_bound** / **ranges::lower\_bound** | (C++20) | `<algorithm>` / `<ranges>` | 返回**第一个不小于**给定值的元素的迭代器 |
| **upper\_bound** / **ranges::upper\_bound** | (C++20) | `<algorithm>` / `<ranges>` | 返回**第一个大于**给定值的元素的迭代器 |
| **equal\_range** / **ranges::equal\_range** | (C++20) | `<algorithm>` / `<ranges>` | 返回**匹配**特定键的元素的**范围** |
| **binary\_search** / **ranges::binary\_search** | (C++20) | `<algorithm>` / `<ranges>` | 确定一个元素是否存在于一个**部分有序**的范围中 |

### 集合操作 (Set operations)

这些操作要求输入范围**必须是已排序**的。

| 集合操作 (Set operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **includes** / **ranges::includes** | (C++20) | `<algorithm>` / `<ranges>` | 如果一个序列是另一个序列的**子序列**则返回 true |
| **set\_union** / **ranges::set\_union** | (C++20) | `<algorithm>` / `<ranges>` | 计算两个集合的**并集** |
| **set\_intersection** / **ranges::set\_intersection** | (C++20) | `<algorithm>` / `<ranges>` | 计算两个集合的**交集** |
| **set\_difference** / **ranges::set\_difference** | (C++20) | `<algorithm>` / `<ranges>` | 计算两个集合的**差集** |
| **set\_symmetric\_difference** / **ranges::set\_symmetric\_difference** | (C++20) | `<algorithm>` / `<ranges>` | 计算两个集合的**对称差集** |

### 合并操作 (Merge operations)

| 合并操作 (Merge operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **merge** / **ranges::merge** | (C++20) | `<algorithm>` / `<ranges>` | **合并**两个已排序的范围 |
| **inplace\_merge** / **ranges::inplace\_merge** | (C++20) | `<algorithm>` / `<ranges>` | 在**原地**合并两个有序范围 |

### 堆操作 (Heap operations)

这些操作用于维护最大堆 (max heap) 的属性。

| 堆操作 (Heap operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **push\_heap** / **ranges::push\_heap** | (C++20) | `<algorithm>` / `<ranges>` | 向**最大堆**中添加一个元素 |
| **pop\_heap** / **ranges::pop\_heap** | (C++20) | `<algorithm>` / `<ranges>` | 从**最大堆**中移除最大的元素 |
| **make\_heap** / **ranges::make\_heap** | (C++20) | `<algorithm>` / `<ranges>` | 将一个元素范围**创建**为**最大堆** |
| **sort\_heap** / **ranges::sort\_heap** | (C++20) | `<algorithm>` / `<ranges>` | 将一个**最大堆**转换为一个**升序排序**的元素范围 |
| **is\_heap** / **ranges::is\_heap** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 检查给定范围是否为**最大堆** |
| **is\_heap\_until** / **ranges::is\_heap\_until** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 查找作为**最大堆**的**最大子范围** |

### 最小/最大操作 (Minimum/maximum operations)

| 最小/最大操作 (Minimum/maximum operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **max** / **ranges::max** | (C++20) | `<algorithm>` / `<ranges>` | 返回给定值中的**较大者** |
| **max\_element** / **ranges::max\_element** | (C++20) | `<algorithm>` / `<ranges>` | 返回范围中的**最大元素** |
| **min** / **ranges::min** | (C++20) | `<algorithm>` / `<ranges>` | 返回给定值中的**较小者** |
| **min\_element** / **ranges::min\_element** | (C++20) | `<algorithm>` / `<ranges>` | 返回范围中的**最小元素** |
| **minmax** / **ranges::minmax** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 返回两个元素中的**较小者**和**较大者** |
| **minmax\_element** / **ranges::minmax\_element** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 返回范围中的**最小**和**最大元素** |
| **clamp** / **ranges::clamp** | (C++17) / (C++20) | `<algorithm>` / `<ranges>` | 将一个值**钳制**在一对边界值之间 |

### 字典序比较操作 (Lexicographical comparison operations)

| 字典序比较操作 (Lexicographical comparison operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **lexicographical\_compare** / **ranges::lexicographical\_compare** | (C++20) | `<algorithm>` / `<ranges>` | 如果一个范围**字典序上小于**另一个，则返回 true |
| **lexicographical\_compare\_three\_way** | (C++20) | `<algorithm>` | 使用**三路比较**比较两个范围 |

### 排列操作 (Permutation operations)

| 排列操作 (Permutation operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **next\_permutation** / **ranges::next\_permutation** | (C++20) | `<algorithm>` / `<ranges>` | 生成范围元素的**下一个更大**的字典序排列 |
| **prev\_permutation** / **ranges::prev\_permutation** | (C++20) | `<algorithm>` / `<ranges>` | 生成范围元素的**下一个更小**的字典序排列 |
| **is\_permutation** / **ranges::is\_permutation** | (C++11) / (C++20) | `<algorithm>` / `<ranges>` | 确定一个序列是否是另一个序列的**排列** |

## 四、数值操作 (Numeric operations)

**主要头文件：**

- **`<numeric>`**：包含所有传统的数值算法和 C++17 的并行数值算法。
- **`<ranges>`**：包含 `ranges::iota`。

| 数值操作 (Numeric operations) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **iota** / **ranges::iota** | (C++11) / (C++23) | `<numeric>` / `<ranges>` | 用**递增**的起始值填充一个范围 |
| **accumulate** | | `<numeric>` | 对一个元素范围**求和**或**折叠** |
| **inner\_product** | | `<numeric>` | 计算两个元素范围的**内积** |
| **adjacent\_difference** | | `<numeric>` | 计算一个范围中**相邻元素**之间的**差值** |
| **partial\_sum** | | `<numeric>` | 计算一个元素范围的**部分和** |
| **reduce** | (C++17) | `<numeric>` | 类似于 `std::accumulate`，但**无序** |
| **exclusive\_scan** | (C++17) | `<numeric>` | 类似于 `std::partial_sum`，但**不包括**第 \\( N \\) 个输入元素在第 \\( N \\) 个和中 |
| **inclusive\_scan** | (C++17) | `<numeric>` | 类似于 `std::partial_sum`，**包括**第 \\( N \\) 个输入元素在第 \\( N \\) 个和中 |
| **transform\_reduce** | (C++17) | `<numeric>` | 应用一个可调用对象，然后**无序归约** |
| **transform\_exclusive\_scan** | (C++17) | `<numeric>` | 应用一个可调用对象，然后计算**独占扫描** |
| **transform\_inclusive\_scan** | (C++17) | `<numeric>` | 应用一个可调用对象，然后计算**包含扫描** |

## 五、未初始化内存操作 (Operations on uninitialized memory)

**主要头文件：**

- **`<memory>`**：包含所有未初始化内存操作。

| 未初始化内存操作 (Operations on uninitialized memory) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **uninitialized\_copy** / **ranges::uninitialized\_copy** | (C++20) | `<memory>` | 将一系列对象**复制**到一块**未初始化内存**区域 |
| **uninitialized\_copy\_n** / **ranges::uninitialized\_copy\_n** | (C++11) / (C++20) | `<memory>` | 将**指定数量**的对象**复制**到一块**未初始化内存**区域 |
| **uninitialized\_fill** / **ranges::uninitialized\_fill** | (C++20) | `<memory>` | 将一个对象**复制赋值**给一块**未初始化内存**区域 |
| **uninitialized\_fill\_n** / **ranges::uninitialized\_fill\_n** | (C++20) | `<memory>` | 将一个对象**复制赋值**给**指定数量**的**未初始化内存**区域 |
| **uninitialized\_move** / **ranges::uninitialized\_move** | (C++17) / (C++20) | `<memory>` | 将一系列对象**移动**到一块**未初始化内存**区域 |
| **uninitialized\_move\_n** / **ranges::uninitialized\_move\_n** | (C++17) / (C++20) | `<memory>` | 将**指定数量**的对象**移动**到一块**未初始化内存**区域 |
| **uninitialized\_default\_construct** / **ranges::uninitialized\_default\_construct** | (C++17) / (C++20) | `<memory>` | 在一块**未初始化内存**区域中**默认构造**对象 |
| **uninitialized\_default\_construct\_n** / **ranges::uninitialized\_default\_construct\_n** | (C++17) / (C++20) | `<memory>` | 在一块**未初始化内存**区域中**默认构造** \\( N \\) 个对象 |
| **uninitialized\_value\_construct** / **ranges::uninitialized\_value\_construct** | (C++17) / (C++20) | `<memory>` | 在一块**未初始化内存**区域中**值构造**对象 |
| **uninitialized\_value\_construct\_n** / **ranges::uninitialized\_value\_construct\_n** | (C++17) / (C++20) | `<memory>` | 在一块**未初始化内存**区域中**值构造** \\( N \\) 个对象 |
| **destroy** / **ranges::destroy** | (C++17) / (C++20) | `<memory>` | **销毁**一系列对象 |
| **destroy\_n** / **ranges::destroy\_n** | (C++17) / (C++20) | `<memory>` | **销毁**范围内的 \\( N \\) 个对象 |
| **destroy\_at** / **ranges::destroy\_at** | (C++17) / (C++20) | `<memory>` | **销毁**给定地址处的对象 |
| **construct\_at** / **ranges::construct\_at** | (C++20) | `<memory>` | 在给定地址处**创建**一个对象 |

## 六、其他算法和工具

### 随机数生成 (Random number generation)

**主要头文件：**

- **`<ranges>`**

| 随机数生成 (Random number generation) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **ranges::generate\_random** | (C++26) | `<ranges>` | 用**均匀随机位生成器**填充一个随机数范围 |

### C 库函数 (C library functions)

**主要头文件：**

- **`<cstdlib>`** (或 `<stdlib.h>`)

| C 库函数 (C library functions) | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- |
| **qsort** | | `<cstdlib>` | **排序**一个**类型未指定**的元素范围 |
| **bsearch** | | `<cstdlib>` | **搜索**一个**类型未指定**的数组中的元素 |

### 附注：执行策略 (Execution policies)

**主要头文件：**

- **`<execution>`**

| 执行策略类型 (Execution policy types) | 宏/类/对象 | C++ 版本 | 头文件 | 描述 (功能) |
| :--- | :--- | :--- | :--- | :--- |
| **sequenced\_policy** (`seq`) | 类 / 全局对象 | (C++17) | `<execution>` | **序列化**执行（无并行） |
| **parallel\_policy** (`par`) | 类 / 全局对象 | (C++17) | `<execution>` | **并行**执行 |
| **parallel\_unsequenced\_policy** (`par\_unseq`) | 类 / 全局对象 | (C++17) | `<execution>` | **并行**且**乱序**执行 |
| **unsequenced\_policy** (`unseq`) | 类 / 全局对象 | (C++20) | `<execution>` | **乱序**执行（允许编译器向量化） |
| **is\_execution\_policy** | 类模板 | (C++17) | `<execution>` | 测试一个类是否表示一个执行策略 |
