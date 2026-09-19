# 原子操作

原子操作是比互斥量更轻量的同步手段。它保证单个操作**不可分割**——要么完全执行，要么完全没执行，不存在中间状态。

对于简单的共享变量（计数器、标志位、指针），原子操作比加锁快得多，因为它不需要内核参与。

## 一、std::atomic

```c++
#include <atomic>

std::atomic<int> counter{0};

void increment()
{
    ++counter;   // 原子操作，无需加锁
}
```

### 1. 支持的类型

| 类型 | 说明 |
| :--- | :--- |
| `std::atomic<bool>` | 布尔标志 |
| `std::atomic<int>` 等整型 | 计数器 |
| `std::atomic<T*>` | 指针 |
| `std::atomic<float>` / `double` | C++20 起支持 |
| `std::atomic<T>` | 自定义类型，要求可平凡复制 |

对自定义类型，`std::atomic<T>` 可能内部使用锁（因为无法用单条指令完成），可以用 `is_lock_free()` 检查：

```c++
struct Big { char data[64]; };
std::atomic<Big> a;

if (!a.is_lock_free())
    std::cout << "内部使用了锁，性能不如预期\n";
```

### 2. 常用接口

```c++
std::atomic<int> value{0};

// 读
int v = value.load();

// 写
value.store(42);

// 读-改-写
int old = value.fetch_add(1);      // 返回旧值，然后加 1
int old2 = value.fetch_sub(1);
int old3 = value.fetch_and(0xFF);
int old4 = value.fetch_or(0x01);
int old5 = value.fetch_xor(0x01);

// 交换
int prev = value.exchange(100);    // 设为 100，返回旧值

// 隐式转换与赋值
int x = value;      // 等价于 load()
value = 42;         // 等价于 store(42)
++value;            // 等价于 fetch_add(1) + 1
```

### 3. atomic_flag

`std::atomic_flag` 是唯一保证无锁的原子类型，常用于实现自旋锁：

```c++
class SpinLock
{
    std::atomic_flag flag_ = ATOMIC_FLAG_INIT;

public:
    void lock()
    {
        while (flag_.test_and_set(std::memory_order_acquire))
        {
            // 自旋
        }
    }

    void unlock()
    {
        flag_.clear(std::memory_order_release);
    }
};
```

C++20 起 `atomic_flag` 还支持 `test()`（只读）和 `wait()` / `notify_one()`。

## 二、比较并交换（CAS）

CAS 是无锁编程的基础操作：**如果当前值等于期望值，就替换为新值**。

```c++
std::atomic<int> value{0};

int expected = 0;
bool success = value.compare_exchange_strong(expected, 42);
// 如果 value == 0，则设为 42，返回 true
// 否则把 value 的当前值写入 expected，返回 false
```

注意 `expected` 是**引用传递**——失败时会被更新为当前值，这是为了支持重试循环。

### 1. weak 与 strong

| 版本 | 特点 |
| :--- | :--- |
| `compare_exchange_weak` | 可能**虚假失败**（值相等也返回 false），但更快 |
| `compare_exchange_strong` | 不会虚假失败，但可能更慢 |

`weak` 版本在循环中使用，因为它虚假失败后重试的代价很小：

```c++
std::atomic<int> value{0};

void increment()
{
    int expected = value.load();
    // 失败时 expected 会被更新，直接重试即可
    while (!value.compare_exchange_weak(expected, expected + 1))
    {
        // expected 已被更新为当前值，循环继续
    }
}
```

`strong` 版本适合单次尝试（比如「只在值未变时才更新」）。

### 2. CAS 循环的通用模式

```c++
template<class T>
void atomicUpdate(std::atomic<T>& atom, std::function<T(T)> transform)
{
    T expected = atom.load();
    while (!atom.compare_exchange_weak(expected, transform(expected)))
    {
        // expected 已更新，重试
    }
}
```

### 3. ABA 问题

CAS 只比较值，无法察觉「值被改回原样」的情况：

```
线程 A：读取 value = 100
线程 B：value = 200
线程 B：value = 100
线程 A：CAS(100 → 300) 成功   ← 但它基于的是过期的假设
```

在指针场景下这可能导致严重后果（指针指向的对象已被释放又分配了新的）。

解决办法是加**版本号**：

```c++
struct VersionedPtr
{
    Node* ptr;
    uint64_t version;
};

std::atomic<VersionedPtr> head;
// 每次修改都递增 version，ABA 就无法通过比较
```

或者用 C++20 的 `std::atomic<std::shared_ptr<T>>`（内部处理了这个问题）。

## 三、内存序

原子操作默认使用 `std::memory_order_seq_cst`（顺序一致），这是最严格也最慢的选项。

### 1. 六种内存序

| 内存序 | 可用于 | 保证 |
| :--- | :--- | :--- |
| `memory_order_relaxed` | 读/写 | 只保证原子性，无同步 |
| `memory_order_consume` | 读 | 数据依赖顺序（已不推荐） |
| `memory_order_acquire` | 读 | 之后的读写不能重排到它之前 |
| `memory_order_release` | 写 | 之前的读写不能重排到它之后 |
| `memory_order_acq_rel` | 读-改-写 | 兼具 acquire 和 release |
| `memory_order_seq_cst` | 全部 | 全局统一顺序（默认） |

### 2. relaxed：只要原子性

```c++
std::atomic<int> counter{0};

void increment()
{
    counter.fetch_add(1, std::memory_order_relaxed);
}
```

`relaxed` 保证计数不会丢失，但不建立任何同步关系。适合纯粹的统计计数。

### 3. acquire / release：建立同步

最常用的一对，用于「写完数据后发布」：

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
    while (!ready.load(std::memory_order_acquire)) { }   // 等待发布
    std::cout << payload;   // 保证看到 42
}
```

**关键**：`release` 之前的所有写入，对执行 `acquire` 的线程可见。如果这里用 `relaxed`，消费者可能看到 `ready == true` 但 `payload` 还是 0。

### 4. seq_cst：全局一致顺序

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

`seq_cst` 保证所有线程看到同一个全局操作顺序，让推理变简单。代价是需要内存屏障，在 ARM 等弱内存模型平台上性能损失明显。

**建议**：默认用 `seq_cst`。确认它是瓶颈后再降级到 `acquire` / `release`。

## 四、C++20 的新工具

### 1. atomic 的等待与通知

C++20 给 `std::atomic` 加了 `wait()` / `notify_one()` / `notify_all()`，可以替代简单的条件变量：

```c++
std::atomic<int> value{0};

// 等待线程
void waiter()
{
    int expected = 0;
    value.wait(expected);   // 阻塞直到 value != 0
    std::cout << "值变为 " << value.load() << '\n';
}

// 通知线程
void notifier()
{
    value.store(42);
    value.notify_one();
}
```

它比条件变量更轻量——不需要互斥量。适合「等待某个标志变化」的场景。

### 2. atomic_ref

`std::atomic_ref` 让**非原子对象**支持原子操作：

```c++
int data = 0;

void increment()
{
    std::atomic_ref<int> ref(data);   // 对 data 的原子引用
    ref.fetch_add(1);
}
```

这在与已有数据结构交互时很有用——你无法改变数据的类型，但需要原子访问。

**注意**：使用 `atomic_ref` 期间，所有对 `data` 的访问都必须通过 `atomic_ref`，否则仍有数据竞争。

### 3. 浮点原子操作

C++20 起 `std::atomic<float>` 和 `std::atomic<double>` 支持 `fetch_add` / `fetch_sub`：

```c++
std::atomic<double> sum{0.0};
sum.fetch_add(3.14);
```

不过很多平台上这不是无锁的（浮点 CAS 不常见）。

## 五、无锁编程的限度

「无锁」听起来很美，但实际使用要谨慎。

### 1. 无锁不等于更快

- 无锁算法通常需要 CAS 循环，争用时可能反复重试
- 内存屏障的开销可能比锁还大
- 代码复杂度高，正确性难验证

**基准测试表明**：在中等争用下，`std::mutex` 往往比手写无锁结构更快。

### 2. 正确性极难保证

无锁数据结构需要考虑：

- **ABA 问题**
- **内存回收**（节点何时能安全释放）
- **内存序**（每个操作该用什么序）
- **虚假失败**（weak CAS）

一个看似简单的无锁栈，正确实现需要几百行代码和严格的证明。

### 3. 什么时候值得用

| 场景 | 建议 |
| :--- | :--- |
| 简单计数器、标志位 | 用 `std::atomic` |
| 一般共享数据保护 | 用 `std::mutex` |
| 极短临界区、高争用 | 考虑自旋锁或原子操作 |
| 需要无锁队列/栈 | 优先用成熟库（如 folly、boost::lockfree） |
| 实时系统、不能阻塞 | 无锁是必要的，但要充分测试 |

## 六、常见误区

**「`std::atomic` 一定无锁」** —— 对自定义类型可能内部用锁。用 `is_lock_free()` 检查。

**「原子操作可以保护多个变量」** —— 不能。每个原子变量独立，多变量一致性仍需互斥量。

**「复合操作是原子的」** —— 不是：

```c++
std::atomic<int> x{0};
if (x > 0)      // 原子读
    x--;        // 原子写
// 但「检查再减」整体不是原子的
```

**「`volatile` 可以替代 `std::atomic`」** —— 不能。`volatile` 不提供原子性，也不建立同步关系。

**「`relaxed` 没有用」** —— 它保证了原子性，适合纯计数器场景，性能最好。

**「CAS 失败就是出错了」** —— CAS 失败是正常情况（说明有其它线程修改了值），循环重试即可。

**「无锁一定比加锁快」** —— 中等争用下往往更慢。先测量再优化。

## 七、相关章节

- [C++ 内存模型](../Memory/Memory_Model.md)：happens-before 与内存序的完整语义
- [互斥量与锁](./Mutex.md)：更通用的同步方式
- [条件变量](./Condition_Variable.md)：复杂的等待条件
- [并发模式与陷阱](./Patterns.md)：无锁结构的实际应用
