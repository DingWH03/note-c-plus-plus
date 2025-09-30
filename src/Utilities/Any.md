# std::any

`std::any` 是 C++17 引入的一个工具类，用来表示 **类型安全的任意类型容器**。它可以存储任何 **可拷贝构造** 的类型，并且在运行时动态管理其类型和生命周期。`std::any` 常用于需要存储不同类型对象，但在编译时无法确定类型的场景。

- **有值 (engaged)**：存储一个特定类型的对象。
- **无值 (disengaged)**：表示空状态，没有存储任何对象。

相比于 `void*` 或不安全的类型转换，`std::any` 提供了类型安全的访问和异常检测机制。

其模板定义如下：

```cpp
class any;
```

## 1. 引入

被包括在 `<any>` 头文件中。

```cpp
#include <any>
```

存储的类型必须满足 **CopyConstructible**（可拷贝构造）。

## 2. `std::any` 的工作原理

- 内部通过 **指针 + 类型信息（typeid）** 或小对象优化实现：
  - 当对象有值时，存储实际类型对象。
  - 当对象为空时，存储状态标记为空。
- 通过 `any_cast` 提供类型安全访问，如果类型不匹配会抛出 `std::bad_any_cast` 异常。

### (1) 为什么需要 `std::any`？

- 当需要存储 **不同类型的对象**，但类型在编译时未知时。
- 用于实现 **动态类型容器**、事件系统或通用缓存。
- 避免使用不安全的 `void*` 或 `union`。

### (2) 基本使用

```cpp
std::any a = 1;             // 存储 int
a = 3.14;                   // 改为存储 double
a = std::string("hello");   // 改为存储 std::string

if (a.has_value()) {
    std::cout << std::any_cast<std::string>(a) << "\n";
}
```

## 3. 使用 `std::any` 的场景

### (1) 动态类型存储

```cpp
std::vector<std::any> values;
values.push_back(10);
values.push_back(3.14);
values.push_back(std::string("text"));

for (auto& v : values) {
    if (v.type() == typeid(int))
        std::cout << std::any_cast<int>(v) << "\n";
}
```

### (2) 类型安全访问

通过 `any_cast` 访问内部对象，类型不匹配会抛异常：

```cpp
std::any a = 42;
try {
    double d = std::any_cast<double>(a);  // 类型错误，会抛出 std::bad_any_cast
} catch (const std::bad_any_cast& e) {
    std::cout << e.what() << "\n";
}
```

### (3) 指针访问

`any_cast` 也可返回指针，类型错误时返回 nullptr：

```cpp
std::any a = 3;
if (int* p = std::any_cast<int>(&a)) {
    std::cout << *p << "\n";  // 输出 3
}
```

## 4. 常用操作

### (0) 构造与析构

```cpp
std::any a;                 // 默认构造：无值
std::any b = 42;            // 从值构造
std::any c = std::string("hi"); // 从其他类型构造

// 拷贝构造 / 拷贝赋值
std::any d = b;
d = c;

// 移动构造 / 移动赋值
std::any e = std::move(c);
e = std::move(b);

// 析构：生命周期结束时自动析构存储的对象
{
    std::any f = std::string("RAII");
} // f 的析构自动调用 std::string 析构函数
```

### (1) `has_value()` 与 `type()`

```cpp
std::any a = 42;
if (a.has_value()) {
    std::cout << "类型: " << a.type().name() << "\n";
}
```

### (2) `emplace()` 与 `reset()`

```cpp
std::any a;
a.emplace<std::string>("Hello");  // 原地构造新值
std::cout << std::any_cast<std::string>(a) << "\n";
a.reset();                         // 清空
```

### (3) 赋值操作 `operator=`

```cpp
std::any a, b;
a = 10;                // 赋值 int
b = std::string("hi"); // 赋值 string
a = b;                 // 拷贝 b 的值到 a
```

### (4) `swap()`

```cpp
std::any a = 1, b = 2;
a.swap(b);
std::cout << std::any_cast<int>(a) << " " << std::any_cast<int>(b) << "\n"; // 输出 2 1
```

### (5) 工厂函数 `std::make_any`

```cpp
auto a = std::make_any<int>(42);           // 创建 any 并存储 int
auto b = std::make_any<std::string>("hi"); // 创建 any 并存储 string
```

### (6) 非成员函数 `any_cast`

```cpp
std::any a = 123;
int* p = std::any_cast<int>(&a);           // 指针访问
int v = std::any_cast<int>(a);             // 值访问，类型错误抛异常
```

## 5. `std::any` 的常见误用

### (1) 滥用

- 不应将 `std::any` 用作所有类型的通用容器，它不能替代容器或指针管理。
- 仅在 **存储单个动态类型对象** 且类型不确定时使用。

### (2) 忘记类型检查

```cpp
std::any a = 42;
std::cout << std::any_cast<double>(a); // 未检查类型，抛异常
```
