# std::uninitialized_default_construct / uninitialized_value_construct

C++17 引入的这两组算法在**未初始化内存**区域上构造对象，区别在于初始化方式：

- `uninitialized_default_construct`：**默认初始化**（`T t;`，内置类型值不确定）
- `uninitialized_value_construct`：**值初始化**（`T t{};`，内置类型置零）

各自都有 `_n` 版本，用于构造指定数量的对象。

## 1. 引入

```c++
#include <memory>
```

## 2. 原理

```c++
template<class NoThrowForwardIt>
void uninitialized_default_construct(NoThrowForwardIt first, NoThrowForwardIt last);

template<class NoThrowForwardIt, class Size>
NoThrowForwardIt uninitialized_default_construct_n(NoThrowForwardIt first, Size n);

template<class NoThrowForwardIt>
void uninitialized_value_construct(NoThrowForwardIt first, NoThrowForwardIt last);

template<class NoThrowForwardIt, class Size>
NoThrowForwardIt uninitialized_value_construct_n(NoThrowForwardIt first, Size n);
```

- **迭代器要求**：`ForwardIterator`（指向未初始化内存）。
- **复杂度**：\\( O(n) \\) 次构造。
- **返回值**：非 `_n` 版本返回 `void`；`_n` 版本返回末尾位置。

**关键区别**：

| 算法 | 等价写法 | `int` 的初值 |
| :--- | :--- | :--- |
| `uninitialized_default_construct` | `::new (p) T;` | **不确定** |
| `uninitialized_value_construct` | `::new (p) T();` | `0` |

对类类型，默认构造和值构造通常等价（都调用默认构造函数）；对内置类型则不同。

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <memory>

int main()
{
    std::allocator<int> alloc;
    int* raw = alloc.allocate(3);

    // 值初始化：置零
    std::uninitialized_value_construct_n(raw, 3);
    std::cout << raw[0] << raw[1] << raw[2] << '\n';   // 000

    std::destroy_n(raw, 3);

    // 默认初始化：值不确定
    std::uninitialized_default_construct_n(raw, 3);
    // raw[0..2] 的值是不确定的，读取是未定义行为

    std::destroy_n(raw, 3);
    alloc.deallocate(raw, 3);
}
```

### (2) 谓词与投影

没有谓词或投影参数。`ranges::` 版本支持投影。

### (3) 执行策略

**不支持**执行策略。

## 4. 注意事项

- **默认初始化 vs 值初始化**：这是最容易混淆的地方。内置类型用默认初始化会得到不确定的值。
- **`std::vector<T> v(n)` 用的是值初始化**：所以 `vector<int> v(5)` 中的元素都是 0。
- **必须手动销毁**。
- **目标必须是未初始化内存**。
- **`_n` 版本的 `n` 非负**。

## 5. 相关算法

- [uninitialized_copy](./Uninitialized_copy.md)：拷贝构造
- [uninitialized_fill](./Uninitialized_fill.md)：填充值
- [construct_at](./Construct_at.md)：在单个地址构造
- [destroy](./Destroy.md)：销毁对象
