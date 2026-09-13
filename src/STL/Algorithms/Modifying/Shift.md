# std::shift_left / std::shift_right

C++20 引入的这两个算法把范围内的元素**左移**或**右移**若干位置。被移出范围的元素会被覆盖，空出的位置处于有效但未指定的状态。

- `shift_left`：把元素向左移动，丢弃前面的元素
- `shift_right`：把元素向右移动，丢弃后面的元素

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt>
constexpr ForwardIt shift_left(ForwardIt first, ForwardIt last,
                               typename std::iterator_traits<ForwardIt>::difference_type n);

template<class ForwardIt>
constexpr ForwardIt shift_right(ForwardIt first, ForwardIt last,
                                typename std::iterator_traits<ForwardIt>::difference_type n);
```

- **迭代器要求**：`shift_left` 需要 `ForwardIterator`；`shift_right` 需要 `BidirectionalIterator`。
- **复杂度**：至多 \\( n \\) 次移动赋值。
- **返回值**：
  - `shift_left` 返回新的末尾位置（`last - n`，若 `n >= size` 则返回 `first`）
  - `shift_right` 返回新的起始位置（`first + n`，若 `n >= size` 则返回 `last`）

如果 `n <= 0`，不执行任何操作。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    // 左移 2 位
    std::vector<int> v{1, 2, 3, 4, 5};
    auto newEnd = std::shift_left(v.begin(), v.end(), 2);
    // v 的前 3 个元素变为 {3,4,5}，后面是残留元素
    v.erase(newEnd, v.end());   // v = {3,4,5}

    // 右移 2 位
    std::vector<int> v2{1, 2, 3, 4, 5};
    auto newBegin = std::shift_right(v2.begin(), v2.end(), 2);
    // v2 变为 {?,?,1,2,3}，前两个是残留元素
    // 需要把前面的位置赋值为想要的值，或者用 erase 删除
}
```

### (2) 谓词与投影

`shift_left` 和 `shift_right` 都**没有谓词或投影参数**。

### (3) 执行策略

`shift_left` 和 `shift_right` 支持 C++17 执行策略。

## 4. 注意事项

- **不改变容器大小**：和 `remove` 一样，需要配合 `erase` 才能真正删除。
- **残留元素**：被移出范围的元素仍在容器中，处于有效但未指定的状态，不要依赖它们的值。
- **`n` 超过范围大小**：`n >= size` 时，`shift_left` 返回 `first`（所有元素都被移出），`shift_right` 返回 `last`。
- **`shift_right` 需要双向迭代器**。
- **与 `rotate` 的区别**：`rotate` 是循环移位（不丢元素），`shift_*` 是丢弃元素的线性移位。
- **`std::copy` / `std::move` 也能实现**：但 `shift_*` 语义更明确，且对重叠范围处理正确。

## 5. 相关算法

- [rotate / rotate_copy](./Rotate.md)：循环旋转
- [move / move_backward](./Move.md)：移动元素
- [copy / copy_if / copy_n](./Copy.md)：复制元素
- [remove / remove_if](./Remove.md)：逻辑移除
