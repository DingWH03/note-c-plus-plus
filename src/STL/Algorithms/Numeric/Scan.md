# std::exclusive_scan / std::inclusive_scan

C++17 引入的这两个算法计算范围的**前缀和（扫描）**，区别在于是否包含当前元素：

- `inclusive_scan`：第 \\( i \\) 个结果**包含**第 \\( i \\) 个输入元素
- `exclusive_scan`：第 \\( i \\) 个结果**不包含**第 \\( i \\) 个输入元素

它们都允许以任意顺序计算，因此支持并行。

## 1. 引入

```c++
#include <numeric>
```

## 2. 原理

```c++
template<class InputIt, class OutputIt>
constexpr OutputIt inclusive_scan(InputIt first, InputIt last, OutputIt d_first);

template<class InputIt, class OutputIt, class T>
constexpr OutputIt exclusive_scan(InputIt first, InputIt last, OutputIt d_first, T init);

template<class ExecutionPolicy, class ForwardIt, class OutputIt>
OutputIt inclusive_scan(ExecutionPolicy&& policy, ForwardIt first, ForwardIt last,
                        OutputIt d_first);
```

- **迭代器要求**：输入 `InputIterator`，输出 `OutputIterator`（并行版本需要 `ForwardIterator`）。
- **复杂度**：\\( O(n) \\) 次 `op` 调用。
- **返回值**：指向输出范围末尾的迭代器。

两者的差异（以 `[1, 2, 3, 4]` 为例，操作为加法）：

| 算法 | 结果 |
| :--- | :--- |
| `inclusive_scan`（初始值 0） | `{1, 3, 6, 10}` |
| `exclusive_scan`（初始值 0） | `{0, 1, 3, 6}` |

`exclusive_scan` **必须提供初始值**，`inclusive_scan` 可以不提供（用第一个元素作为初值）。

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <numeric>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    // inclusive_scan：包含当前元素
    std::vector<int> inc(v.size());
    std::inclusive_scan(v.begin(), v.end(), inc.begin());
    // inc = {1, 3, 6, 10, 15}

    // exclusive_scan：不包含当前元素
    std::vector<int> exc(v.size());
    std::exclusive_scan(v.begin(), v.end(), exc.begin(), 0);
    // exc = {0, 1, 3, 6, 10}

    // 指定初始值
    std::vector<int> inc2(v.size());
    std::inclusive_scan(v.begin(), v.end(), inc2.begin(), std::plus<>{}, 100);
    // inc2 = {101, 103, 106, 110, 115}
}
```

### (2) 谓词与投影

接受二元操作，没有投影参数。需要先变换时用 `transform_inclusive_scan` / `transform_exclusive_scan`。

### (3) 执行策略

两者都支持 C++17 执行策略：

```c++
#include <execution>

std::inclusive_scan(std::execution::par, v.begin(), v.end(), out.begin());
```

同样要求操作满足交换律和结合律。

## 4. 注意事项

- **`exclusive_scan` 必须提供初始值**：这是它与 `inclusive_scan` 的重要区别。
- **输出范围必须足够大**。
- **操作必须满足结合律**（并行时还需交换律）。
- **与 `partial_sum` 的关系**：`inclusive_scan` 语义等同 `partial_sum`，但支持并行。
- **原地操作**：`inclusive_scan` 允许 `d_first == first`；`exclusive_scan` **不允许**（因为输出与输入错位）。

## 5. 相关算法

- [partial_sum](./Partial_sum.md)：顺序版本的包含扫描
- [transform_scan](./Transform_scan.md)：先变换再扫描
- [reduce](./Reduce.md)：归约（只要最终结果）
- [adjacent_difference](./Adjacent_difference.md)：相邻差值
