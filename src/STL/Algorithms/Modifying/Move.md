# std::move / std::move_backward

这两个算法把范围内的元素**移动**到目标位置（通过移动赋值/构造），源范围的元素处于「有效但未指定」的状态。

- `move`：从前往后移动
- `move_backward`：从后往前移动（用于目标在右侧的重叠场景）

注意：这里的 `std::move` 是**算法**（定义在 `<algorithm>`），与 `<utility>` 中的 `std::move`（类型转换）同名但完全不同。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class OutputIt>
constexpr OutputIt move(InputIt first, InputIt last, OutputIt d_first);

template<class BidirIt1, class BidirIt2>
constexpr BidirIt2 move_backward(BidirIt1 first, BidirIt1 last, BidirIt2 d_last);
```

- **迭代器要求**：`move` 需要 `InputIterator`；`move_backward` 需要 `BidirectionalIterator`。
- **复杂度**：恰好 \\( n \\) 次移动赋值。
- **返回值**：指向目标范围中最后一个被移动元素之后的位置（`move_backward` 返回首元素位置）。

移动语义避免了拷贝开销，对持有资源的类型（如 `std::string`、`std::vector`）尤其高效。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <string>
#include <vector>

int main()
{
    std::vector<std::string> src{"hello", "world", "foo"};

    // 移动到新容器
    std::vector<std::string> dst(src.size());
    std::move(src.begin(), src.end(), dst.begin());

    // dst = {"hello", "world", "foo"}
    // src 中的字符串变为空（有效但未指定状态）

    for (const auto& s : src)
        std::cout << '[' << s << ']';   // [][][]
}
```

### (2) 谓词与投影

`move` 没有谓词或投影参数。如果需要「有条件地移动」，可以先用 `copy_if` 配合 `std::make_move_iterator`：

```c++
std::vector<std::string> src{"a", "", "b", ""};
std::vector<std::string> dst;

std::copy_if(std::make_move_iterator(src.begin()),
             std::make_move_iterator(src.end()),
             std::back_inserter(dst),
             [](const std::string& s) { return !s.empty(); });
// dst = {"a", "b"}
```

### (3) 执行策略

`move` 和 `move_backward` 支持 C++17 执行策略：

```c++
#include <execution>

std::move(std::execution::par, src.begin(), src.end(), dst.begin());
```

## 4. 注意事项

- **与 `std::move` 同名**：`<utility>` 的 `std::move` 是类型转换，这里是算法。同时 `using namespace std;` 时要注意区分。
- **源元素状态**：移动后源元素处于「有效但未指定」状态，不要假设它们为空，只能重新赋值或销毁。
- **目标空间必须足够**：不检查目标范围大小。
- **重叠范围**：目标在右侧时用 `move_backward`。
- **`vector<string>` 扩容**：`std::vector` 内部扩容正是用 `uninitialized_move` 把元素搬到新内存。

## 5. 相关算法

- [copy / copy_if / copy_n](./Copy.md)：复制版本
- [copy_backward](./Copy_backward.md)：从后往前复制
- [uninitialized_move](../Memory/Uninitialized_move.md)：移动到未初始化内存
- [remove](./Remove.md)：配合移动语义实现高效删除
