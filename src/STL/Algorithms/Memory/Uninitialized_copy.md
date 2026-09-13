# std::uninitialized_copy / uninitialized_copy_n

这两个算法把对象**拷贝构造**到一块**未初始化内存**区域。与 `std::copy` 不同，它们不要求目标区域已有对象，而是直接在原始内存上构造。

- `uninitialized_copy`：复制整个范围
- `uninitialized_copy_n`：复制指定数量的元素（C++11）

## 1. 引入

```c++
#include <memory>
```

## 2. 原理

```c++
template<class InputIt, class NoThrowForwardIt>
ForwardIt uninitialized_copy(InputIt first, InputIt last, NoThrowForwardIt d_first);

template<class InputIt, class Size, class NoThrowForwardIt>
ForwardIt uninitialized_copy_n(InputIt first, Size count, NoThrowForwardIt d_first);
```

- **迭代器要求**：输入 `InputIterator`，输出 `ForwardIterator`（指向未初始化内存）。
- **复杂度**：\\( O(n) \\) 次拷贝构造。
- **返回值**：指向最后一个被构造元素之后的位置。

实现等价于：

```c++
template<class InputIt, class NoThrowForwardIt>
NoThrowForwardIt uninitialized_copy(InputIt first, InputIt last, NoThrowForwardIt d_first)
{
    using T = typename std::iterator_traits<NoThrowForwardIt>::value_type;
    NoThrowForwardIt current = d_first;
    try {
        for (; first != last; ++first, (void)++current)
            ::new (static_cast<void*>(std::addressof(*current))) T(*first);
        return current;
    } catch (...) {
        // 销毁已构造的对象
        for (; d_first != current; ++d_first)
            d_first->~T();
        throw;
    }
}
```

**异常安全**：如果构造过程中抛出异常，已构造的对象会被销毁，不会泄漏。

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

    // 分配原始内存（未构造对象）
    std::allocator<std::string> alloc;
    std::string* raw = alloc.allocate(src.size());

    // 在原始内存上拷贝构造
    std::uninitialized_copy(src.begin(), src.end(), raw);

    // 此时 raw[0]、raw[1] 是有效对象
    std::cout << raw[0] << ' ' << raw[1] << '\n';   // hello world

    // 手动销毁并释放
    std::destroy_n(raw, src.size());
    alloc.deallocate(raw, src.size());
}
```

### (2) 谓词与投影

`uninitialized_copy` 没有谓词或投影参数。`ranges::uninitialized_copy` 支持投影。

### (3) 执行策略

**不支持**执行策略。内存构造过程需要精确控制异常安全，无法简单并行化。

## 4. 注意事项

- **目标必须是真正的未初始化内存**：如果目标已有对象，应改用 `std::copy`（否则会重复构造，导致泄漏或未定义行为）。
- **必须手动销毁**：构造出的对象需要用 `std::destroy` 或显式调用析构函数来销毁。
- **异常安全**：实现会保证异常时不泄漏已构造的对象。
- **典型用途**：`std::vector` 扩容时把旧元素搬到新内存。
- **`ranges::uninitialized_copy`（C++20）**：返回 `ranges::uninitialized_copy_result`，支持投影。

## 5. 相关算法

- [uninitialized_move](./Uninitialized_move.md)：移动到未初始化内存
- [uninitialized_fill](./Uninitialized_fill.md)：填充未初始化内存
- [destroy](./Destroy.md)：销毁对象
- [copy / copy_if / copy_n](../Modifying/Copy.md)：复制到已初始化内存
