# 修改序列操作 (Modifying sequence operations)

修改序列操作会改变范围内元素的值或位置，但**通常不改变容器的大小**。需要特别注意「移除类」算法只是**逻辑移除**——它们把保留的元素移到范围前部，并返回新的逻辑末尾，真正的删除仍需配合容器的 `erase`。

**主要头文件：**

- **`<algorithm>`**：包含绝大多数经典的修改序列操作。
- **`<ranges>`**：包含所有 `ranges::` 版本的算法。
- **`<utility>`**：包含 `std::swap` 和 `std::iter_swap`（C++11 起）。

## 算法总览

| 算法 | C++ 版本 | 头文件 | 描述 |
| :--- | :--- | :--- | :--- |
| [copy / copy_if / copy_n](./Modifying/Copy.md) | (C++11) | `<algorithm>` | 将元素范围复制到新位置 |
| [copy_backward](./Modifying/Copy_backward.md) | | `<algorithm>` | 以向后顺序复制一个元素范围 |
| [move / move_backward](./Modifying/Move.md) | (C++11) | `<algorithm>` | 将元素范围移动（或向后移动）到新位置 |
| [swap / swap_ranges / iter_swap](./Modifying/Swap.md) | (C++11) | `<utility>` | 交换两个对象、范围或迭代器所指元素 |
| [transform](./Modifying/Transform.md) | | `<algorithm>` | 对元素范围应用函数并写入目标范围 |
| [replace / replace_if](./Modifying/Replace.md) | | `<algorithm>` | 将满足条件的值替换为另一个值 |
| [replace_copy / replace_copy_if](./Modifying/Replace_copy.md) | | `<algorithm>` | 复制范围并替换其中满足条件的元素 |
| [fill / fill_n](./Modifying/Fill.md) | | `<algorithm>` | 将给定值赋值给范围内的元素 |
| [generate / generate_n](./Modifying/Generate.md) | | `<algorithm>` | 将连续函数调用的结果赋值给元素 |
| [remove / remove_if](./Modifying/Remove.md) | | `<algorithm>` | 逻辑移除满足条件的元素 |
| [remove_copy / remove_copy_if](./Modifying/Remove_copy.md) | | `<algorithm>` | 复制范围并省略满足条件的元素 |
| [unique / unique_copy](./Modifying/Unique.md) | | `<algorithm>` | 移除（或复制时省略）连续重复元素 |
| [reverse / reverse_copy](./Modifying/Reverse.md) | | `<algorithm>` | 反转元素顺序或创建反转副本 |
| [rotate / rotate_copy](./Modifying/Rotate.md) | | `<algorithm>` | 旋转元素顺序或复制并旋转 |
| [shift_left / shift_right](./Modifying/Shift.md) | (C++20) | `<algorithm>` | 左移或右移范围内的元素 |
| [shuffle](./Modifying/Shuffle.md) | (C++11) | `<algorithm>` | 随机重新排序范围内的元素 |
| [sample](./Modifying/Sample.md) | (C++17) | `<algorithm>` | 从序列中随机选择 \\( N \\) 个元素 |

> **说明：** 表中「C++ 版本」列标注的是该算法（或 `ranges::` 版本）首次引入的标准版本，留空表示自 C++98 起即存在。

## 共性说明

- **移除类算法**：`remove`、`remove_if`、`unique` 属于**逻辑移除**，必须配合容器的 `erase` 才能真删除，这就是所谓的 **erase-remove 惯用法**（C++20 起可用 `std::erase` / `std::erase_if` 替代）。
- **重叠范围**：`copy` 要求目标范围不与源范围重叠，向右复制时应改用 `copy_backward`；`move` 与 `move_backward` 同理。
- **执行策略**：`copy`、`move`、`fill`、`transform`、`replace`、`remove`、`reverse`、`rotate`、`shuffle`、`generate` 等支持 C++17 执行策略。
