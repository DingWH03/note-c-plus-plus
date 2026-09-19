# 自定义内存管理

前面几章讲的都是「怎么正确使用内存」。这一章讲另一件事：**怎么让内存分配更快、更可控**。

标准库的 `new` 和 `malloc` 是通用分配器，要应付各种大小、各种模式，因此实现复杂、开销不小。如果你的程序有明确的分配模式（比如大量相同大小的小对象、频繁创建销毁），自己管理内存可以带来数量级的提升。

不过要先说清楚：**绝大多数程序不需要自定义分配器**。先用 profiler 确认分配真的是瓶颈，再考虑优化。

## 一、通用分配器的问题

`malloc` 的典型开销：

- **查找空闲块**：维护多个大小类别的空闲链表，需要遍历
- **加锁**：多线程下要保护堆结构，即使用了 thread-local cache 也有开销
- **元数据**：每块内存前面有头部，记录大小等信息
- **碎片化**：反复分配释放不同大小的块，会留下难以利用的空洞
- **缓存不友好**：对象分散在堆各处，遍历时 cache miss 频繁

一个具体例子：分配 100 万个 32 字节的小对象，`malloc` 实际可能占用 48 字节/个（含头部和对齐），而且每次分配都是一次函数调用加链表操作。

## 二、分配器接口

标准库容器允许指定分配器，接口定义在 `<memory>`：

```c++
template<class T>
class MyAllocator
{
public:
    using value_type = T;

    MyAllocator() = default;

    template<class U>
    MyAllocator(const MyAllocator<U>&) noexcept {}

    T* allocate(std::size_t n);
    void deallocate(T* p, std::size_t n);

    // C++11 起，相等比较决定分配器能否互换
    bool operator==(const MyAllocator&) const noexcept { return true; }
    bool operator!=(const MyAllocator&) const noexcept { return false; }
};
```

一个最小实现：

```c++
template<class T>
T* MyAllocator<T>::allocate(std::size_t n)
{
    if (n > std::numeric_limits<std::size_t>::max() / sizeof(T))
        throw std::bad_alloc();
    if (auto p = static_cast<T*>(std::malloc(n * sizeof(T))))
        return p;
    throw std::bad_alloc();
}

template<class T>
void MyAllocator<T>::deallocate(T* p, std::size_t) noexcept
{
    std::free(p);
}
```

用法：

```c++
std::vector<int, MyAllocator<int>> v;
```

**注意**：分配器的 `allocate` 只分配**原始内存**，不构造对象。构造由容器通过 `allocator_traits::construct`（内部是 placement new）完成。

## 三、内存池

内存池的思路很简单：**一次申请一大块，之后从里面切分**。

### 1. 固定大小内存池

适合「大量相同大小的对象」场景：

```c++
class FixedPool
{
public:
    explicit FixedPool(std::size_t blockSize, std::size_t blockCount)
        : blockSize_(std::max(blockSize, sizeof(void*)))
    {
        // 一次分配所有内存
        storage_ = std::malloc(blockSize_ * blockCount);

        // 把空闲块串成链表
        for (std::size_t i = 0; i < blockCount; ++i)
        {
            void* block = static_cast<char*>(storage_) + i * blockSize_;
            *static_cast<void**>(block) = freeList_;   // 用块本身存 next 指针
            freeList_ = block;
        }
    }

    ~FixedPool() { std::free(storage_); }

    void* allocate()
    {
        if (!freeList_)
            throw std::bad_alloc();

        void* block = freeList_;
        freeList_ = *static_cast<void**>(freeList_);   // 取出下一个
        return block;
    }

    void deallocate(void* p) noexcept
    {
        *static_cast<void**>(p) = freeList_;   // 头插回空闲链表
        freeList_ = p;
    }

private:
    void* storage_ = nullptr;
    void* freeList_ = nullptr;
    std::size_t blockSize_;
};
```

分配和释放都只是链表操作，是**常数时间**，而且没有锁、没有元数据开销。

这个技巧叫 **free list**，用一个空闲块的前几个字节存下一个空闲块的地址——反正空闲块里的数据已经没用了。

### 2. 内存池的代价

- **不能释放单块给系统**：整个池要一起释放
- **块大小固定**：需要不同大小就得建多个池
- **可能浪费**：如果块大小不匹配，会有内部碎片

所以内存池适合**生命周期一致、大小固定**的对象，不适合通用场景。

## 四、对象池

内存池管理原始内存，对象池则管理**已构造的对象**，常用于需要频繁创建销毁的场景：

```c++
template<class T>
class ObjectPool
{
public:
    template<class... Args>
    T* create(Args&&... args)
    {
        void* mem = pool_.allocate();
        return new (mem) T(std::forward<Args>(args)...);   // placement new
    }

    void destroy(T* obj)
    {
        obj->~T();                  // 显式析构
        pool_.deallocate(obj);
    }

private:
    FixedPool pool_{sizeof(T), 1024};
};
```

这在游戏开发中很常见——子弹、粒子这类对象每秒创建销毁上千次，用对象池可以避免反复的堆分配。

## 五、pmr：多态内存资源

C++17 引入了 `<memory_resource>`，提供了一套**运行时多态**的分配器机制。它解决了传统分配器的一个痛点：分配器类型是容器类型的一部分，改分配器就要改类型。

```c++
#include <memory_resource>

// 传统方式：分配器写进类型
std::vector<int, MyAllocator<int>> v1;

// pmr 方式：分配器是构造参数
std::pmr::vector<int> v2{&myResource};
```

`std::pmr::vector<T>` 就是 `std::vector<T, std::pmr::polymorphic_allocator<T>>` 的别名。

### 内置的内存资源

| 类型 | 说明 |
| :--- | :--- |
| `std::pmr::new_delete_resource()` | 默认，转发到 `new` / `delete` |
| `std::pmr::null_memory_resource()` | 任何分配都抛异常，用于测试 |
| `std::pmr::monotonic_buffer_resource` | 单调递增，只增不减，极快 |
| `std::pmr::unsynchronized_pool_resource` | 池化分配，单线程 |
| `std::pmr::synchronized_pool_resource` | 池化分配，线程安全 |

### monotonic_buffer_resource

这是最实用的一个。它在一块预分配的内存上单调递增地分配，**不支持单独释放**，直到整个资源销毁：

```c++
// 用栈上的缓冲区，避免任何堆分配
std::array<std::byte, 4096> buffer;
std::pmr::monotonic_buffer_resource pool{buffer.data(), buffer.size()};

std::pmr::vector<int> v{&pool};
v.push_back(1);   // 从 buffer 分配，不碰堆
```

典型用法是「处理一批数据，用完整体丢弃」：

```c++
void processRequest(const Request& req)
{
    std::pmr::monotonic_buffer_resource pool;

    // 这一堆临时对象都从 pool 分配
    std::pmr::vector<Item> items{&pool};
    std::pmr::string name{&pool};
    std::pmr::map<int, Detail> details{&pool};

    // ... 处理

}   // pool 析构，所有内存一次性释放
```

这样避免了多次 `new` / `delete`，也避免了内存碎片。

## 六、对齐分配

有时需要特定对齐的内存，比如 SIMD 类型要求 16 或 32 字节对齐。

C++17 提供了对齐版本的分配：

```c++
void* p = operator new(size, std::align_val_t{64});
operator delete(p, std::align_val_t{64});
```

C 风格接口：

```c++
// C11 / C++17
void* p = std::aligned_alloc(64, size);
std::free(p);
```

或者用 C++11 的 `std::align` 手动调整：

```c++
void* ptr = buffer;
std::size_t space = sizeof(buffer);
void* aligned = std::align(64, size, ptr, space);
```

## 七、什么时候值得自定义分配

| 场景 | 是否值得 |
| :--- | :--- |
| 通用业务代码 | ❌ 用默认分配器 |
| 大量同大小小对象（> 10 万） | ✅ 固定大小池 |
| 频繁创建销毁的短期对象 | ✅ 对象池 |
| 批处理、请求级临时数据 | ✅ `monotonic_buffer_resource` |
| 实时系统、不能有分配抖动 | ✅ 预分配池 |
| 嵌入式、内存受限 | ✅ 定制分配器 |

**先测量再优化**。用 profiler 看分配热点，确认 `malloc` 真的占了可观的时间，再考虑动手。

一个常见的误判是「我的程序分配很频繁，所以需要内存池」。但如果每次分配只花 50 纳秒，100 万次也才 50 毫秒——可能根本不是瓶颈。

## 八、常见误区

**「自定义分配器一定更快」** —— 通用分配器经过几十年优化，对一般场景已经很好。自定义分配器的优势来自**针对特定模式**的优化，模式不匹配反而更慢。

**「内存池可以随便释放单块」** —— 大多数池不支持单独归还给系统，只能整池释放。需要单独释放就得用更复杂的结构（如 slab 分配器）。

**「`allocate` 会构造对象」** —— 不会。它只给原始内存，构造由容器负责。

**「pmr 比普通分配器慢」** —— 多一层虚函数调用，但省去了分配器类型污染。在批处理场景下，`monotonic_buffer_resource` 通常比默认分配快得多。

**「内存池不会有碎片」** —— 固定大小池不会有外部碎片，但可能有内部碎片（块大小不匹配）。而且池本身如果长期运行、反复分配释放，空闲链表也可能变得分散。

## 九、相关章节

- [动态内存分配](./Allocation.md)：`new` / `delete` 与 `operator new`
- [内存布局与模型](./Layout.md)：对齐与对象布局
- [内存调试与诊断](./Debugging.md)：分配器的调试支持
- [RAII 与资源管理](./RAII.md)：分配器也需要 RAII 封装