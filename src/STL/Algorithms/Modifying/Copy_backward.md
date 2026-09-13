# std::copy_backward

`std::copy_backward` 以**从后往前**的顺序把范围 `[first, last)` 的元素复制到以 `d_last` 结尾的目标位置，返回指向目标范围首元素的迭代器。

它主要用于**目标范围与源范围重叠且目标在右侧**的场景。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class BidirIt1, class BidirIt2>
constexpr BidirIt2 copy_backward(BidirIt1 first, BidirIt1 last, BidirIt2 d_last);
```

- **迭代器要求**：`BidirectionalIterator`（因为要从后往前）。
- **复杂度**：恰好 \\( n \\) 次赋值。
- **返回值**：指向目标范围中第一个被复制元素的迭代器。

复制顺序是从 `*(last-1)` 到 `*(d_last-1)`，然后依次向前。这样即使目标范围与源范围有重叠，只要目标在右侧，也不会覆盖尚未复制的元素。

**重叠范围的选择规则**：

| 目标位置 | 应使用的算法 |
| :--- | :--- |
| 目标在**左侧**（目标末尾在源范围之前） | `copy` |
| 目标在**右侧**（目标开头在源范围之后） | `copy_backward` |

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    // 把前 3 个元素复制到末尾（目标在右侧，重叠）
    std::copy_backward(v.begin(), v.begin() + 3, v.end());
    // v 变为 {1, 2, 3, 1, 2, 3}? 不对——原 v 只有 5 个元素
    // 实际结果：{1, 2, 1, 2, 3}
    for (int x : v) std::cout << x << ' ';   // 1 2 1 2 3
}
```

更典型的例子是「在容器内部右移一段元素」：

```c++
std::vector<int> v{1, 2, 3, 4, 5, 0, 0};   // 后两个是预留空间
// 把 {1,2,3,4,5} 右移 2 位，得到 {0,0,1,2,3,4,5}
std::copy_backward(v.begin(), v.begin() + 5, v.end());
// v 变为 {1, 2, 1, 2, 3, 4, 5}
```

### (2) 谓词与投影

`copy_backward` 没有谓词或投影参数，它总是复制整个范围。`ranges::copy_backward` 也不支持投影。

### (3) 执行策略

`copy_backward` 支持 C++17 执行策略：

```c++
#include <execution>

std::copy_backward(std::execution::par, src.begin(), src.end(), dst.end());
```

## 4. 注意事项

- **`d_last` 是目标范围的末尾**：不是起始位置，这是最容易出错的地方。目标范围是 `[d_last - (last - first), d_last)`。
- **需要双向迭代器**：`forward_list` 的迭代器不可用。
- **重叠判断**：只有目标在右侧时才用 `copy_backward`；目标在左侧应该用 `copy`。
- **返回值的含义**：返回目标范围的首迭代器，而非末尾。

## 5. 相关算法

- [copy / copy_if / copy_n](./Copy.md)：从前往后复制
- [move / move_backward](./Move.md)：移动版本
- [reverse_copy](./Reverse.md)：反转后复制
- [rotate](./Rotate.md)：原地旋转
