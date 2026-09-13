# std::construct_at

C++20 引入的 `std::construct_at` 在**给定地址**处构造一个对象，是 placement new 的标准库封装版本。

## 1. 引入

```c++
#include <memory>
```

## 2. 原理

```c++
template<class T, class... Args>
constexpr T* construct_at(T* location, Args&&... args);
```

- **参数**：`location` 是目标地址，`args` 转发给 `T` 的构造函数。
- **复杂度**：常数。
- **返回值**：指向新构造对象的指针（即 `location`）。

实现等价于：

```c++
template<class T, class... Args>
constexpr T* construct_at(T* location, Args&&... args)
{
    return ::new (static_cast<void*>(location)) T(std::forward<Args>(args)...);
}
```

与 placement new 的区别：

- `construct_at` 是 `constexpr` 的（C++20 起），可用于常量求值
- 它会在构造前检查 `location` 是否指向有效的存储
- 可以在 `constexpr` 上下文中使用，而 placement new 不能

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <memory>
#include <string>

int main()
{
    std::allocator<std::string> alloc;
    std::string* raw = alloc.allocate(1);

    // 在原始内存上构造对象
    std::construct_at(raw, "hello");
    std::cout << *raw << '\n';   // hello

    // 销毁并释放
    std::destroy_at(raw);
    alloc.deallocate(raw, 1);
}
```

`constexpr` 上下文中的使用：

```c++
constexpr int f()
{
    std::aligned_storage_t<sizeof(int), alignof(int)> storage;
    int* p = reinterpret_cast<int*>(&storage);
    std::construct_at(p, 42);
    int result = *p;
    std::destroy_at(p);
    return result;
}
static_assert(f() == 42);
```

### (2) 谓词与投影

没有谓词或投影参数。`ranges::construct_at` 是 C++20 的 Ranges 版本，语义相同。

### (3) 执行策略

**不支持**执行策略。

## 4. 注意事项

- **地址必须指向足够的存储**：否则是未定义行为。
- **必须手动销毁**：构造出的对象需要用 `destroy_at` 销毁。
- **不要对已有对象调用**：会重复构造，导致资源泄漏。
- **优先用 `construct_at` 而非 placement new**：它更安全，且支持 `constexpr`。
- **C++20 起 `std::allocator::construct` 已被移除**：应改用 `std::construct_at` 或 `std::allocator_traits::construct`。

## 5. 相关算法

- [destroy](./Destroy.md)：销毁对象
- [uninitialized_copy](./Uninitialized_copy.md)：批量构造
- [uninitialized_construct](./Uninitialized_construct.md)：默认/值构造
- [uninitialized_fill](./Uninitialized_fill.md)：填充构造
