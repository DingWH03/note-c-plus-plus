# std::uninitialized_move / uninitialized_move_n

C++17 引入的这两个算法把对象**移动构造**到**未初始化内存**区域：

- `uninitialized_move`：移动整个范围
- `uninitialized_move_n`：移动指定数量的元素

它们与 `uninitialized_copy` 的区别在于使用**移动构造**而非拷贝构造，对持有资源的类型（如 `std::string`、`std::vector`）更高效。

## 1. 引入

```c++
#include <memory>
```

## 2. 原理

```c++
template<class InputIt, class NoThrowForwardIt>
NoThrowForwardIt uninitialized_move(InputIt first, InputIt last, NoThrowForwardIt d_first);

template<class InputIt, class Size, class NoThrowForwardIt>
std::pair<InputIt, NoThrowForwardIt>
    uninitialized_move_n(InputIt first, Size count, NoThrowForwardIt d_first);
```

- **迭代器要求**：输入 `InputIterator`，输出 `ForwardIterator`（指向未初始化内存）。
- **复杂度**：\\( O(n) \\) 次移动构造。
- **返回值**：
  - `uninitialized_move` 返回目标范围末尾
  - `uninitialized_move_n` 返回 `std::pair`，包含源范围和目标范围各自的末尾

移动后源对象处于「有效但未指定」状态。

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <memory>
#include <string>
#include <vector>

int main()
{
    std::vector<std::string> src{"hello", "world"};

    std::allocator<std::string> alloc;
    std::string* raw = alloc.allocate(src.size());

    // 移动构造到原始内存
    std::uninitialized_move(src.begin(), src.end(), raw);

    std::cout << raw[0] << ' ' << raw[1] << '\n';   // hello world
    // src 中的字符串现在为空（有效但未指定）

    std::destroy_n(raw, src.size());
    alloc.deallocate(raw, src.size());
}
```

### (2) 谓词与投影

没有谓词或投影参数。`ranges::uninitialized_move` 支持投影。

### (3) 执行策略

**不支持**执行策略。

## 4. 注意事项

- **源对象状态**：移动后源对象处于有效但未指定状态，不要假设它们为空。
- **目标必须是未初始化内存**。
- **必须手动销毁**目标对象。
- **典型用途**：`std::vector` 扩容时用 `uninitialized_move_if_noexcept` 决定是移动还是拷贝（如果移动构造可能抛异常且类型可拷贝，则选择拷贝以保证强异常安全）。
- **`uninitialized_move_n` 返回 `pair`**：同时给出源和目标的末尾位置。

## 5. 相关算法

- [uninitialized_copy](./Uninitialized_copy.md)：拷贝构造到未初始化内存
- [uninitialized_fill](./Uninitialized_fill.md)：填充未初始化内存
- [destroy](./Destroy.md)：销毁对象
- [move / move_backward](../Modifying/Move.md)：移动到已初始化内存
