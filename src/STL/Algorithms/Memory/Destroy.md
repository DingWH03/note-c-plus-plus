# std::destroy / destroy_n / destroy_at

C++17 引入的这三个算法**销毁**对象（调用析构函数），但不释放内存：

- `destroy_at`：销毁给定地址处的单个对象
- `destroy`：销毁范围内的所有对象
- `destroy_n`：销毁范围内的前 \\( N \\) 个对象

## 1. 引入

```c++
#include <memory>
```

## 2. 原理

```c++
template<class T>
void destroy_at(T* location);

template<class ForwardIt>
void destroy(ForwardIt first, ForwardIt last);

template<class ForwardIt, class Size>
ForwardIt destroy_n(ForwardIt first, Size n);
```

- **迭代器要求**：`ForwardIterator`。
- **复杂度**：\\( O(n) \\) 次析构调用。
- **返回值**：`destroy_at` 和 `destroy` 返回 `void`；`destroy_n` 返回最后一个被销毁元素之后的位置。

实现等价于：

```c++
template<class T>
void destroy_at(T* location)
{
    location->~T();
}

template<class ForwardIt>
void destroy(ForwardIt first, ForwardIt last)
{
    for (; first != last; ++first)
        std::destroy_at(std::addressof(*first));
}
```

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

    std::uninitialized_fill_n(raw, 3, std::string("hi"));

    // 销毁所有对象（但不释放内存）
    std::destroy_n(raw, 3);

    // 内存仍需手动释放
    alloc.deallocate(raw, 3);
}
```

### (2) 谓词与投影

没有谓词或投影参数。`ranges::destroy` 等支持投影。

### (3) 执行策略

**不支持**执行策略。

## 4. 注意事项

- **只销毁不释放**：`destroy` 调用析构函数，但内存仍需 `deallocate` 释放。两者是分开的步骤。
- **不要重复销毁**：对已销毁的对象再次调用析构函数是未定义行为。
- **`destroy_n` 的 `n` 非负**。
- **`std::allocator_traits::destroy`**：分配器也有 `destroy` 成员，语义相同但会考虑分配器的自定义行为。
- **典型用途**：`std::vector::clear()` 内部就是调用 `destroy` 销毁元素，但保留容量。

## 5. 相关算法

- [construct_at](./Construct_at.md)：构造对象
- [uninitialized_copy](./Uninitialized_copy.md)：构造对象
- [uninitialized_construct](./Uninitialized_construct.md)：默认/值构造
- [vector 的 clear](../../Containers/Vector.md)：内部使用 destroy
