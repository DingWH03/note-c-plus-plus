# C++ 内存模型（并发）

单线程程序里，「代码按顺序执行」是理所当然的。但在多线程环境下，编译器会重排指令、CPU 会乱序执行、缓存会让不同核心看到不同的值——你以为的顺序，在硬件层面完全不存在。

C++11 引入的**内存模型**（memory model）就是用来约束这些行为的：它规定了多线程程序在什么条件下能观察到什么值。理解它是写出正确并发代码的前提。

这一章偏理论，但每个概念都会落到实际用法上。

## 一、数据竞争

**数据竞争**（data race）的定义很明确：

> 两个线程同时访问同一内存位置，其中至少一个是写操作，且没有同步措施。

数据竞争的后果是**未定义行为**——不是「读到旧值」这么简单，而是编译器可以做任何假设，程序行为完全不可预测。

```c++
int counter = 0;   // 普通 int

void increment()
{
    for (int i = 0; i < 100000; ++i)
        ++counter;    // 数据竞争
}

int main()
{
    std::thread t1(increment);
    std::thread t2(increment);
    t1.join();
    t2.join();

    std::cout << counter << '\n';   // 大概率不是 200000
}
```

`++counter` 实际是「读-改-写」三步，两个线程可能同时读到相同的值，导致丢失更新。

消除数据竞争只有两条路：**加锁**或**用原子操作**。

## 二、happens-before 关系

内存模型的核心概念是 **happens-before**。如果操作 A happens-before 操作 B，那么 A 的所有内存写入对 B 都可见。

happens-before 由几种关系组合而成：

### 1. 同线程内的顺序

同一线程中，前面的语句 happens-before 后面的语句：

```c++
int x = 1;      // A
int y = x + 1;  // B：A happens-before B
```

### 2. 锁的同步

一个线程解锁，另一个线程加同一个锁，则解锁 happens-before 加锁：

```c++
std::mutex mtx;
int data = 0;

// 线程 A
{
    std::lock_guard<std::mutex> lock(mtx);
    data = 42;          // 写
}                       // 解锁

// 线程 B
{
    std::lock_guard<std::mutex> lock(mtx);
    std::cout << data;  // 保证读到 42
}                       // 加锁
```

这是互斥量能保证可见性的原因——它不只是「互斥」，还建立了同步关系。

### 3. 原子操作的同步

原子操作可以建立跨线程的 happens-before：

```c++
std::atomic<bool> ready{false};
int data = 0;

// 线程 A
data = 42;
ready.store(true, std::memory_order_release);   // 释放

// 线程 B
while (!ready.load(std::memory_order_acquire)) { }  // 获取
std::cout << data;   // 保证读到 42
```

`release` 和 `acquire` 配对，建立起 A 到 B 的同步关系。

### 4. 传递性

happens-before 具有传递性：如果 A → B 且 B → C，那么 A → C。

## 三、内存序

原子操作默认使用 `std::memory_order_seq_cst`（顺序一致），这是最严格也最慢的。C++ 提供了六种内存序：

| 内存序 | 用途 | 保证 |
| :--- | :--- | :--- |
| `memory_order_relaxed` | 只需要原子性 | 无同步，只保证操作不可分割 |
| `memory_order_consume` | 数据依赖（很少用） | 已不推荐 |
| `memory_order_acquire` | 读操作 | 之后的读写不能重排到它之前 |
| `memory_order_release` | 写操作 | 之前的读写不能重排到它之后 |
| `memory_order_acq_rel` | 读-改-写操作 | 兼具 acquire 和 release |
| `memory_order_seq_cst` | 默认，最安全 | 全局统一顺序 |

### 1. relaxed：只要原子性

```c++
std::atomic<int> counter{0};

void increment()
{
    for (int i = 0; i < 100000; ++i)
        counter.fetch_add(1, std::memory_order_relaxed);
}
// 结果是确定的 200000，但不保证与其他变量的顺序关系
```

`relaxed` 只保证「计数不会丢失」，不建立任何同步。适合纯粹的计数器。

### 2. acquire / release：建立同步

这是最常用的一对，用于「生产者写完数据，通知消费者」：

```c++
std::atomic<bool> ready{false};
int payload = 0;

// 生产者
void produce()
{
    payload = 42;                                    // 先写数据
    ready.store(true, std::memory_order_release);    // 再发布
}

// 消费者
void consume()
{
    while (!ready.load(std::memory_order_acquire)) { }  // 等待发布
    std::cout << payload;   // 保证看到 42
}
```

关键点：**`release` 之前的写入，对执行 `acquire` 的线程可见**。如果这里用 `relaxed`，消费者可能看到 `ready == true` 但 `payload` 还是 0。

### 3. seq_cst：全局一致顺序

```c++
std::atomic<bool> x{false}, y{false};
std::atomic<int> z{0};

// 线程 A
x.store(true);   // 默认 seq_cst

// 线程 B
y.store(true);

// 线程 C
while (!x.load()) { }
if (y.load())
    ++z;
```

`seq_cst` 保证所有线程看到**同一个全局操作顺序**。这让推理变简单，但代价是需要内存屏障，性能比 `acquire`/`release` 差。

**建议**：默认用 `seq_cst`，确认是瓶颈后再降级到 `acquire`/`release`。

## 四、内存屏障

有时需要在没有原子操作的情况下插入同步点：

```c++
std::atomic_thread_fence(std::memory_order_release);
```

屏障本身不操作数据，只是限制重排。典型用法：

```c++
// 生产者
payload = 42;
std::atomic_thread_fence(std::memory_order_release);
flag.store(true, std::memory_order_relaxed);   // 用 relaxed 存
```

效果和直接在 `store` 上用 `release` 类似，但把同步点独立出来了。实际代码中很少需要，直接用 `acquire`/`release` 更清晰。

## 五、volatile 不是同步手段

这是最经典的误解。`volatile` 在 C++ 中的含义是：

- 告诉编译器「这个变量可能被外部修改」，不要优化掉对它的访问
- **不提供任何原子性**
- **不提供任何线程间同步**
- **不建立 happens-before 关系**

```c++
volatile int counter = 0;

void increment()
{
    ++counter;   // 仍然是数据竞争！
}
```

`volatile` 的正确用途是访问**内存映射的硬件寄存器**，或者配合 `setjmp`/`longjmp`。多线程同步必须用 `std::atomic` 或互斥量。

## 六、实际应用

### 1. 双重检查锁定

单例模式中常见的写法，但很容易写错：

```c++
class Singleton
{
public:
    static Singleton& instance()
    {
        if (!instance_)                              // 第一次检查（无锁）
        {
            std::lock_guard<std::mutex> lock(mtx_);
            if (!instance_)                          // 第二次检查
                instance_ = std::make_unique<Singleton>();
        }
        return *instance_;
    }

private:
    static std::unique_ptr<Singleton> instance_;
    static std::mutex mtx_;
};
```

这里的 `instance_` 是普通指针，第一次检查是**数据竞争**。正确做法是用原子指针：

```c++
static std::atomic<Singleton*> instance_;

static Singleton& instance()
{
    Singleton* p = instance_.load(std::memory_order_acquire);
    if (!p)
    {
        std::lock_guard<std::mutex> lock(mtx_);
        p = instance_.load(std::memory_order_relaxed);
        if (!p)
        {
            p = new Singleton();
            instance_.store(p, std::memory_order_release);
        }
    }
    return *p;
}
```

不过更简单的做法是用 C++11 的**局部静态变量**——标准保证它是线程安全的：

```c++
static Singleton& instance()
{
    static Singleton inst;   // 线程安全的初始化
    return inst;
}
```

### 2. 无锁队列

用原子操作实现的生产者-消费者队列，核心是 CAS 循环：

```c++
template<class T>
class LockFreeStack
{
    struct Node
    {
        T data;
        Node* next;
    };

    std::atomic<Node*> head_{nullptr};

public:
    void push(T value)
    {
        Node* newNode = new Node{std::move(value), nullptr};
        newNode->next = head_.load(std::memory_order_relaxed);

        // CAS 循环：如果 head_ 还是 newNode->next，就换成 newNode
        while (!head_.compare_exchange_weak(newNode->next, newNode,
                                            std::memory_order_release,
                                            std::memory_order_relaxed))
        { }
    }
};
```

无锁编程极其困难，上面这个栈还有 ABA 问题。**除非有明确的性能需求，否则用互斥量**。

### 3. 原子计数器

最简单的场景：

```c++
std::atomic<std::size_t> requestCount{0};

void handleRequest()
{
    requestCount.fetch_add(1, std::memory_order_relaxed);   // 只需原子性
    // ...
}
```

这里 `relaxed` 就够了——我们只关心计数正确，不关心它和其他变量的顺序。

## 七、常见误区

**「`volatile` 可以用于多线程同步」** —— 不能。它不提供原子性，也不建立 happens-before。这是 Java 程序员最常带入 C++ 的错误。

**「原子操作就是无锁」** —— 不一定。`std::atomic<T>` 对某些类型可能内部用锁实现，可以用 `is_lock_free()` 检查。

**「`relaxed` 是没用的」** —— 它保证了原子性，适合纯计数器场景，性能最好。

**「`seq_cst` 太慢，应该都用 `acquire`/`release`」** —— 在 x86 上两者性能差异很小，在 ARM 上才明显。先保证正确性，再考虑优化。

**「`atomic` 变量的所有操作都是原子的」** —— 单个操作是原子的，但复合操作不是：

```c++
std::atomic<int> x{0};

if (x > 0)      // 原子读
    x--;        // 原子写
// 但「检查再减」整体不是原子的，中间可能被插入
```

**「多个原子变量之间有一致性」** —— 没有。两个独立的原子变量之间不保证任何顺序关系，除非用 `seq_cst` 或显式同步。

## 八、相关章节

- [多线程与并发](../Concurrency.md)：线程、互斥量、条件变量的使用
- [内存布局与模型](./Layout.md)：单线程视角的内存布局
- [内存调试与诊断](./Debugging.md)：用 TSan 检测数据竞争
- [智能指针](./Smart_Pointer.md)：`shared_ptr` 的线程安全性