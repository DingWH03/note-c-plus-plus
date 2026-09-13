# std::uninitialized_fill / uninitialized_fill_n

这两个算法在**未初始化内存**区域上**拷贝构造**给定值的副本：

- `uninitialized_fill`：填充整个范围
- `uninitialized_fill_n`：填充前 \\( N \\) 个元素

## 1. 引入

```c++
#include <memory>
```

## 2. 原理

```c++
template<class NoThrowForwardIt, class T>
void uninitialized_fill(NoThrowForwardIt first, NoThrowForwardIt last, const T& value);

template<class NoThrowForwardIt, class Size, class T>
NoThrowForwardIt uninitialized_fill_n(NoThrowForwardIt first, Size count, const T& value);
```

- **迭代器要求**：`ForwardIterator`（指向未初始化内存）。
- **复杂度**：\\( O(n) \\) 次拷贝构造。
- **返回值**：`uninitialized_fill` 返回 `void`；`uninitialized_fill_n` 返回最后一个被构造元素之后的位置。

与 `std::fill` 的区别：`fill` 对**已存在**的对象执行**赋值**，`uninitialized_fill` 在原始内存上执行**拷贝构造**。

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <memory>
#include <string>

int main()
{
    std::allocator<std::string> alloc;
    std::string* raw = alloc.allocate(3);

    // 在原始内存上构造 3 个 "hi"
    std::uninitialized_fill_n(raw, 3, std::string("hi"));

    std::cout << raw[0] << raw[1] << raw[2] << '\n';   // hihihi

    // 销毁并释放
    std::destroy_n(raw, 3);
    alloc.deallocate(raw, 3);
}
```

### (2) 谓词与投影

没有谓词或投影参数。`ranges::uninitialized_fill` 支持投影。

### (3) 执行策略

**不支持**执行策略。

## 4. 注意事项

- **目标必须是未初始化内存**：否则会重复构造导致泄漏。
- **必须手动销毁**。
- **`uninitialized_fill_n` 的 `count` 非负**。
- **异常安全**：构造失败时会销毁已构造的对象。
- **`std::vector<T> v(n, value)` 内部就用它**：先分配内存，再逐个构造。

## 5. 相关算法

- [uninitialized_copy](./Uninitialized_copy.md)：复制到未初始化内存
- [uninitialized_move](./Uninitialized_move.md)：移动到未初始化内存
- [uninitialized_construct](./Uninitialized_construct.md)：默认/值构造
- [fill / fill_n](../Modifying/Fill.md)：填充已初始化内存
