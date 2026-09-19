# 动态内存分配

栈上的对象由编译器管理，出了作用域就自动销毁。但有些对象必须活得更久——比如跨函数传递的数据、大小在运行时才能确定的数组。这类对象需要放在**堆**上，由程序员（或封装好的工具）负责申请和释放。

这一章讲 `new` / `delete` 到底做了什么，以及它们有哪些容易踩的地方。

## 一、new 和 delete 做了什么

很多人以为 `new` 就是「分配内存」，其实它做两件事：

1. 调用 `operator new` 分配原始内存
2. 在这块内存上**构造对象**（调用构造函数）

`delete` 同样做两件事，顺序相反：

1. 调用对象的**析构函数**
2. 调用 `operator delete` 释放内存

```c++
MyClass* p = new MyClass(42);
// 等价于：
// void* mem = operator new(sizeof(MyClass));
// MyClass* p = new (mem) MyClass(42);   // placement new

delete p;
// 等价于：
// p->~MyClass();
// operator delete(p);
```

理解这个分解很重要，后面几个话题都建立在它之上。

## 二、operator new 与 operator delete

`operator new` 是**可以重载的全局函数**，默认实现就是调用 `malloc`：

```c++
// 全局 operator new 的签名
void* operator new(std::size_t size);
void* operator new[](std::size_t size);

void operator delete(void* ptr) noexcept;
void operator delete[](void* ptr) noexcept;
```

重载它们可以接管整个程序的堆分配：

```c++
void* operator new(std::size_t size)
{
    std::cout << "分配 " << size << " 字节\n";
    if (void* p = std::malloc(size))
        return p;
    throw std::bad_alloc();
}

void operator delete(void* ptr) noexcept
{
    std::free(ptr);
}
```

也可以只为某个类重载：

```c++
class Pooled
{
public:
    static void* operator new(std::size_t size);
    static void operator delete(void* ptr) noexcept;
};
```

这常用于对象池、内存追踪等场景。

> `new` 失败时默认抛 `std::bad_alloc`。如果希望像 `malloc` 那样返回 `nullptr`，可以用 `new (std::nothrow) T`。

## 三、placement new

placement new 允许在**已分配的内存**上构造对象，不分配新内存：

```c++
#include <new>

// 1. 先分配原始内存
void* buffer = std::malloc(sizeof(MyClass));

// 2. 在已有内存上构造对象
MyClass* p = new (buffer) MyClass(42);

// 3. 手动析构（不能 delete！）
p->~MyClass();

// 4. 释放原始内存
std::free(buffer);
```

这里**绝对不能调用 `delete p`**——因为内存不是 `operator new` 分配的，`delete` 会试图释放一块不属于它的内存。必须手动调用析构函数，再自己释放缓冲区。

标准库容器正是这样实现的：`std::vector` 先分配一块原始内存，再用 placement new 逐个构造元素。

## 四、new[] 与 delete[]

数组版本的 `new` 会在内存头部额外记录元素个数，`delete[]` 需要这个信息来逐个调用析构函数：

```c++
MyClass* arr = new MyClass[10];
delete[] arr;   // 正确：析构 10 个对象
// delete arr;  // 错误：只析构 1 个，且释放方式不匹配
```

**必须配对使用**。用 `new[]` 分配却用 `delete` 释放（或反之）是未定义行为。

对于没有析构函数的类型（如 `int`），编译器通常不会额外记录长度，但**混用仍然是未定义行为**，不要依赖这一点。

现代 C++ 中，几乎不需要手写 `new[]`——用 `std::vector` 或 `std::make_unique<T[]>` 更安全：

```c++
auto arr = std::make_unique<int[]>(10);   // 自动管理
```

## 五、内存对齐

有些类型要求内存地址必须是某个值的整数倍。比如 SIMD 类型通常要求 16 或 32 字节对齐。

C++17 起，`new` 会自动处理**过度对齐类型**：

```c++
struct alignas(64) CacheLine
{
    char data[64];
};

CacheLine* p = new CacheLine;   // C++17 保证 64 字节对齐
```

C++17 之前需要手动指定：

```c++
void* mem = aligned_alloc(64, sizeof(CacheLine));
```

如果需要手动管理对齐内存：

```c++
#include <memory>

// 分配 64 字节对齐、能放下 10 个 CacheLine 的内存
void* mem = std::aligned_alloc(64, sizeof(CacheLine) * 10);
std::free(mem);
```

## 六、常见错误

### 1. 忘记释放

```c++
void leak()
{
    int* p = new int(42);
    // 忘记 delete p
}
```

短命程序里泄漏可能无所谓，但长期运行的服务会慢慢吃光内存。

### 2. 重复释放

```c++
int* p = new int(42);
delete p;
delete p;   // 未定义行为，通常直接崩溃
```

释放后应该把指针置空，或者干脆用智能指针：

```c++
delete p;
p = nullptr;   // 至少让第二次 delete 变成安全的空操作
```

### 3. 释放后继续使用

```c++
int* p = new int(42);
delete p;
std::cout << *p;   // 悬垂指针，未定义行为
```

这块内存可能已被重新分配给别的对象，读到的值无法预料。

### 4. 释放非堆内存

```c++
int local = 42;
int* p = &local;
delete p;   // 灾难：试图释放栈内存
```

### 5. 混用 malloc/free 与 new/delete

```c++
int* p = (int*)std::malloc(sizeof(int));
delete p;   // 错误：malloc 配 free，new 配 delete

int* q = new int(42);
std::free(q);   // 错误
```

两套机制的内存管理方式不同，混用是未定义行为。

## 七、异常安全

`new` 可能抛出 `std::bad_alloc`，这让手动管理内存更容易出错：

```c++
void risky()
{
    MyClass* a = new MyClass();
    MyClass* b = new MyClass();   // 如果这里抛异常，a 就泄漏了

    // ... 使用

    delete b;
    delete a;
}
```

如果第二个 `new` 失败，`a` 指向的内存永远不会被释放。正确做法是用 RAII：

```c++
void safe()
{
    auto a = std::make_unique<MyClass>();
    auto b = std::make_unique<MyClass>();   // 抛异常时 a 会自动释放
}
```

这是「不要手写 `delete`」这条建议的核心理由——异常路径上的清理太容易遗漏了。

## 八、什么时候该用堆

堆分配有成本，不是所有对象都该放堆上：

| 场景 | 建议 |
| :--- | :--- |
| 局部临时对象 | 栈 |
| 大小在编译期确定的小对象 | 栈 |
| 需要活过作用域的对象 | 堆（用智能指针） |
| 大小运行时才能确定 | 堆（用 `std::vector`） |
| 体积很大（几 MB 以上） | 堆 |
| 多态对象的动态创建 | 堆（用智能指针） |

一个经验法则：**如果对象的生命周期不需要超出当前作用域，就别放堆上**。

## 九、相关章节

- [内存布局与模型](./Layout.md)：栈与堆的区别
- [RAII 与资源管理](./RAII.md)：用对象管理资源，避免手写 delete
- [智能指针](./Smart_Pointer.md)：`unique_ptr` 与 `shared_ptr`
- [自定义内存管理](./Custom_Allocator.md)：内存池与分配器
- [内存调试与诊断](./Debugging.md)：泄漏与越界的排查
