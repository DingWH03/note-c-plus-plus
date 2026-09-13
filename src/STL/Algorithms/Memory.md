# 未初始化内存操作 (Operations on uninitialized memory)

这一组算法定义在 `<memory>` 中，用于在**未初始化的原始内存**上构造、复制、移动和销毁对象。它们不负责分配内存，只负责在已分配的内存上管理对象的生命周期，是手写容器（如自定义 `vector`）的基础设施。

**主要头文件：**

- **`<memory>`**：包含所有未初始化内存操作。

## 算法总览

### 构造与复制

| 算法 | C++ 版本 | 描述 |
| :--- | :--- | :--- |
| [uninitialized_copy / uninitialized_copy_n](./Memory/Uninitialized_copy.md) | (C++11) | 将对象复制到未初始化内存区域 |
| [uninitialized_fill / uninitialized_fill_n](./Memory/Uninitialized_fill.md) | | 将对象复制赋值到未初始化内存区域 |
| [uninitialized_move / uninitialized_move_n](./Memory/Uninitialized_move.md) | (C++17) | 将对象移动到未初始化内存区域 |
| [uninitialized_default_construct / uninitialized_value_construct](./Memory/Uninitialized_construct.md) | (C++17) | 在未初始化内存中默认构造或值构造对象 |
| [construct_at](./Memory/Construct_at.md) | (C++20) | 在给定地址处构造一个对象 |

### 销毁

| 算法 | C++ 版本 | 描述 |
| :--- | :--- | :--- |
| [destroy / destroy_n / destroy_at](./Memory/Destroy.md) | (C++17) | 销毁一系列对象或给定地址处的对象 |

> **说明：** 表中「C++ 版本」列标注的是该算法（或 `ranges::` 版本）首次引入的标准版本，留空表示自 C++98 起即存在。

## 共性说明

- **异常安全**：如果构造过程中抛出异常，这些算法会负责销毁已成功构造的对象，不会造成泄漏。
- **与 `std::copy` 的区别**：`std::copy` 要求目标区域**已存在对象**并执行赋值；`uninitialized_copy` 则在原始内存上执行**拷贝构造**。
- **`ranges::` 版本**：C++20 起提供了对应的 `ranges::uninitialized_*`、`ranges::destroy*` 版本，支持范围参数和投影。
- **典型用途**：`std::vector` 扩容时正是用 `uninitialized_move` 把旧元素搬到新内存，再销毁旧对象。
