# 互斥量与锁

互斥量（mutex）是最基本的同步工具：它保证同一时刻只有一个线程能进入临界区。

「临界区」指的是访问共享数据的代码段。只要所有访问共享数据的地方都套上同一把锁，数据竞争就消除了。

## 一、互斥量类型

| 类型 | 头文件 | 说明 |
| :--- | :--- | :--- |
| `std::mutex` | `<mutex>` | 基本互斥量，最常用 |
| `std::recursive_mutex` | `<mutex>` | 同一线程可重复加锁 |
| `std::timed_mutex` | `<mutex>` | 支持超时加锁 |
| `std::recursive_timed_mutex` | `<mutex>` | 可重入 + 超时 |
| `std::shared_mutex` | `<shared_mutex>` | 读写锁（C++17） |
| `std::shared_timed_mutex` | `<shared_mutex>` | 读写锁 + 超时（C++14） |

### 1. 基本用法

```c++
#include <mutex>

std::mutex mtx;
int counter = 0;

void increment()
{
    mtx.lock();
    ++counter;
    mtx.unlock();
}
```

**但不要这样写**。如果 `++counter` 抛异常，`unlock()` 永远不会执行，其它线程会永久阻塞。应该用 RAII 包装器：

```c++
void increment()
{
    std::lock_guard<std::mutex> lock(mtx);
    ++counter;
}   // 自动解锁，异常也安全
```

### 2. recursive_mutex

同一线程可以重复加锁：

```c++
std::recursive_mutex rmtx;

void outer()
{
    std::lock_guard<std::recursive_mutex> lock(rmtx);
    inner();   // 会再次加锁，普通 mutex 会死锁
}

void inner()
{
    std::lock_guard<std::recursive_mutex> lock(rmtx);
    // ...
}
```

**但通常说明设计有问题**。需要递归加锁，往往意味着「哪些函数持锁」这个契约不清晰。更好的做法是把加锁的边界划清楚，让内部函数假设调用者已持锁。

`recursive_mutex` 还有性能开销——它要记录持有者和递归深度。

### 3. shared_mutex（读写锁）

允许多个读者并发，但写者独占：

```c++
#include <shared_mutex>

std::shared_mutex rwmtx;
std::map<int, std::string> cache;

// 读操作：多个线程可并发
std::string read(int key)
{
    std::shared_lock lock(rwmtx);   // 共享锁
    auto it = cache.find(key);
    return it != cache.end() ? it->second : "";
}

// 写操作：独占
void write(int key, const std::string& value)
{
    std::unique_lock lock(rwmtx);   // 独占锁
    cache[key] = value;
}
```

适合**读多写少**的场景。如果读写频率相当，普通 `mutex` 往往更快——`shared_mutex` 的内部实现更复杂。

## 二、RAII 锁包装器

标准库提供了四种包装器，各有用途：

| 包装器 | 版本 | 特点 |
| :--- | :--- | :--- |
| `std::lock_guard` | C++11 | 最简单，构造加锁、析构解锁，不可移动 |
| `std::unique_lock` | C++11 | 可移动、可延迟加锁、可手动解锁 |
| `std::scoped_lock` | C++17 | 可同时锁定多个互斥量，避免死锁 |
| `std::shared_lock` | C++14 | 配合 `shared_mutex` 的共享锁 |

### 1. lock_guard

最轻量，适合绝大多数场景：

```c++
{
    std::lock_guard<std::mutex> lock(mtx);
    // 临界区
}   // 自动解锁
```

C++17 起可以省略模板参数（CTAD）：

```c++
std::lock_guard lock(mtx);
```

### 2. unique_lock

更灵活，但开销略大（需要记录是否持有锁）：

```c++
std::unique_lock<std::mutex> lock(mtx);              // 立即加锁
std::unique_lock<std::mutex> lock2(mtx, std::defer_lock);   // 暂不加锁

lock2.lock();      // 手动加锁
lock2.unlock();    // 手动解锁
if (lock2.owns_lock()) { /* ... */ }
```

三个标签：

| 标签 | 含义 |
| :--- | :--- |
| `std::defer_lock` | 不立即加锁 |
| `std::try_to_lock` | 尝试加锁，失败不阻塞 |
| `std::adopt_lock` | 假设已加锁，接管所有权 |

`unique_lock` 是**条件变量必需的**——`condition_variable::wait` 需要在等待期间解锁、被唤醒后重新加锁。

### 3. scoped_lock

同时锁定多个互斥量，内部使用死锁避免算法：

```c++
std::mutex m1, m2;

void transfer(Account& from, Account& to, int amount)
{
    std::scoped_lock lock(from.mtx, to.mtx);   // 同时锁定，不会死锁
    from.balance -= amount;
    to.balance += amount;
}
```

如果手动按不同顺序加锁，两个线程可能互相等待：

```c++
// 线程 A：lock(m1) → lock(m2)
// 线程 B：lock(m2) → lock(m1)
// 可能死锁
```

`scoped_lock` 内部用 `std::lock` 算法，它会尝试加锁所有互斥量，失败则全部释放重试，从而避免循环等待。

### 4. 手动使用 std::lock

需要更细粒度控制时，可以用 `std::lock` 配合 `adopt_lock`：

```c++
std::unique_lock<std::mutex> lock1(m1, std::defer_lock);
std::unique_lock<std::mutex> lock2(m2, std::defer_lock);

std::lock(lock1, lock2);   // 同时加锁，避免死锁
```

## 三、call_once

`std::call_once` 保证函数在多线程环境下**只执行一次**：

```c++
std::once_flag flag;

void init()
{
    std::call_once(flag, [] {
        // 只会执行一次，即使多个线程同时调用 init()
    });
}
```

典型用途是延迟初始化：

```c++
class Config
{
public:
    static Config& instance()
    {
        std::call_once(initFlag_, [] {
            instance_.reset(new Config());
        });
        return *instance_;
    }

private:
    static std::once_flag initFlag_;
    static std::unique_ptr<Config> instance_;
};
```

不过 C++11 起，**局部静态变量的初始化本身就是线程安全的**，所以单例更简单的写法是：

```c++
Config& instance()
{
    static Config inst;   // 线程安全的初始化
    return inst;
}
```

## 四、死锁

死锁是多个线程互相等待对方释放资源。四个必要条件（Coffman 条件）：

1. **互斥**：资源同一时刻只能被一个线程持有
2. **持有并等待**：线程持有资源的同时等待其它资源
3. **不可抢占**：资源只能被持有者主动释放
4. **循环等待**：存在线程与资源的环形等待链

### 1. 常见死锁场景

**加锁顺序不一致**：

```c++
// 线程 A
lock(m1); lock(m2);

// 线程 B
lock(m2); lock(m1);   // 可能死锁
```

**忘记解锁**（异常路径）：

```c++
mtx.lock();
doSomething();   // 抛异常 → 永远不解锁
mtx.unlock();
```

**自死锁**（同一线程重复加锁）：

```c++
std::mutex mtx;
mtx.lock();
mtx.lock();   // 死锁
```

### 2. 避免方法

| 方法 | 说明 |
| :--- | :--- |
| **统一加锁顺序** | 所有线程按同一顺序加锁 |
| **用 `scoped_lock`** | 自动避免循环等待 |
| **用 RAII 包装器** | 异常路径也能解锁 |
| **缩小临界区** | 减少持锁时间，降低冲突概率 |
| **避免嵌套锁** | 尽量不在持锁时调用可能加锁的函数 |
| **用超时锁** | `try_lock_for` 失败后放弃并重试 |

## 五、性能考量

### 1. 锁的代价

一次未争用的加锁解锁大约几十纳秒，涉及：

- 原子操作修改锁状态
- 内存屏障（保证临界区内的写入对后续线程可见）
- 可能的系统调用（争用时）

**争用时代价急剧上升**——线程会被挂起，涉及上下文切换（几微秒）。

### 2. 减少争用

**缩小临界区**：

```c++
// 不好：整个函数持锁
void process()
{
    std::lock_guard lock(mtx);
    auto data = fetchFromNetwork();   // 耗时操作，不该持锁
    shared = data;
}

// 好：只在必要时持锁
void process()
{
    auto data = fetchFromNetwork();   // 锁外执行
    std::lock_guard lock(mtx);
    shared = data;
}
```

**分片（sharding）**：把一个大锁拆成多个小锁：

```c++
class ShardedCounter
{
    static constexpr int N = 16;
    std::array<std::mutex, N> mtxs_;
    std::array<int, N> counts_{};

public:
    void increment(int key)
    {
        int idx = key % N;                    // 按 key 分散到不同锁
        std::lock_guard lock(mtxs_[idx]);
        ++counts_[idx];
    }

    int total() const
    {
        int sum = 0;
        for (int c : counts_) sum += c;
        return sum;
    }
};
```

**用原子操作替代锁**：简单计数器用 `std::atomic` 就够了。

**读写锁**：读多写少时用 `shared_mutex`。

### 3. 自旋锁

互斥量在争用时会让线程睡眠。如果临界区极短，睡眠/唤醒的开销可能比自旋等待还大。这时可以用自旋锁：

```c++
class SpinLock
{
    std::atomic_flag flag_ = ATOMIC_FLAG_INIT;

public:
    void lock()
    {
        while (flag_.test_and_set(std::memory_order_acquire))
        {
            // 自旋等待
        }
    }

    void unlock()
    {
        flag_.clear(std::memory_order_release);
    }
};
```

**但自旋锁通常不是好选择**：

- 单核系统上自旋毫无意义（持有者无法运行）
- 临界区稍长就会浪费大量 CPU
- 需要配合 `yield()` 或退避策略

除非你确切知道临界区只有几条指令，否则用 `std::mutex`。

## 六、TOCTOU：加锁了也不一定安全

前面反复强调「访问共享数据要加锁」。但有一个更隐蔽的问题：**检查和使用之间隔了时间，这段空隙里状态可能被改变**。

这就是 TOCTOU（Time-Of-Check To Time-Of-Use，检查时刻到使用时刻）。它的危险之处在于——**即使每一处访问都加了锁，逻辑上仍然可能出错**。

### 1. 问题出在哪

TOCTOU 的经典形态是「先检查，再操作」：

```c++
if (fileExists(path))       // 检查
{
    auto f = openFile(path);   // 使用 —— 中间可能已被删除！
}
```

检查和使用是两次独立的操作，中间的时间窗口里，其它线程（或进程）可以改变状态。

关键在于：**加锁只能保证「单个操作」的原子性，不能保证「检查 + 使用」这一对操作的原子性**。

### 2. 加了锁也可能有 TOCTOU

看这个例子——每一行都规规矩矩地加了锁：

```c++
// 危险：锁的粒度不对，检查和使用之间被拆开了
bool tryConsume()
{
    {
        std::lock_guard lock(mtx_);
        if (queue_.empty())        // 检查
            return false;
    }                              // ← 锁在这里释放了

    {
        std::lock_guard lock(mtx_);
        auto item = queue_.front();   // 使用 —— 队列可能已空！
        queue_.pop();
        return true;
    }
}
```

两次加锁各自都是原子的，但**中间那段没持锁的时间**里，别的线程可能把队列清空了。`queue_.front()` 于是访问了空队列——未定义行为。

**正确的做法是把检查和操作放进同一个临界区**：

```c++
bool tryConsume()
{
    std::lock_guard lock(mtx_);
    if (queue_.empty())        // 检查
        return false;
    auto item = queue_.front();   // 使用 —— 同一把锁，中间无法插入
    queue_.pop();
    return true;
}
```

### 3. 更隐蔽的变体：返回引用

这种写法看起来没问题，其实也是 TOCTOU：

```c++
// 危险：返回的是指针/引用，锁在返回时就释放了
const std::string* find(int key)
{
    std::lock_guard lock(mtx_);
    auto it = map_.find(key);
    if (it == map_.end())
        return nullptr;
    return &it->second;   // 锁一释放，别的线程就能 erase 掉这个元素
}

// 调用方
const std::string* p = find(42);
if (p)                      // 检查
    std::cout << *p;        // 使用 —— 悬空指针！
```

锁保护了 `find` 本身，但**保护不了返回之后的使用**。解决办法是返回**值**（拷贝）而不是引用：

```c++
std::optional<std::string> find(int key)
{
    std::lock_guard lock(mtx_);
    auto it = map_.find(key);
    if (it == map_.end())
        return std::nullopt;
    return it->second;   // 拷贝一份，脱离锁的保护范围也安全
}
```

### 4. 文件系统的 TOCTOU（安全漏洞）

TOCTOU 最初是从**安全领域**被广泛讨论的。Unix 下的经典例子：

```c
// setuid 程序中的漏洞
if (access("file", W_OK) != 0)   // 检查：当前用户能否写
    exit(1);

fd = open("file", O_WRONLY);      // 使用 —— 中间可能被换成符号链接
write(fd, buffer, sizeof(buffer));
```

攻击者可以在 `access` 和 `open` 之间把 `file` 替换成指向 `/etc/passwd` 的符号链接，于是程序以特权身份写入了不该写的文件。

这类漏洞造成过真实事故：

- 早期 BSD 的 `mktemp()` 临时文件竞争
- 早期 OpenSSH 的 Unix domain socket 竞争
- 2019 年 Docker 的 TOCTOU 漏洞允许访问宿主机文件系统
- 2025 年 AWS US-EAST-1 因 DNS 管理系统的竞争条件导致大规模故障

**这类问题的根本困难**：`access` 和 `open` 是两个独立的系统调用，中间无法加锁。2004 年有研究证明，在 Unix 上**不存在**可移植的、确定性的方法来消除这种 TOCTOU。

### 5. 怎么防

| 策略 | 说明 |
| :--- | :--- |
| **扩大临界区** | 让「检查」和「使用」在同一把锁内完成 |
| **返回拷贝而非引用** | 不让共享数据的指针逃出锁的保护范围 |
| **EAFP 而非 LBYL** | 直接操作，用错误处理兜底，而不是先检查 |
| **用原子的复合操作** | 比如 `compare_exchange`、`fetch_add` |
| **用一次性句柄** | 文件场景用 `open` 返回的 fd，而不是路径 |

**EAFP**（Easier to Ask for Forgiveness than Permission）值得展开。与其「先检查再操作」，不如「直接操作，失败就处理」：

```c++
// LBYL：检查和操作之间有窗口
if (fileExists(path))
    openFile(path);

// EAFP：只有一次操作，没有窗口
if (auto f = tryOpenFile(path); f)
    use(*f);
```

「检查」本身就是一次操作，只要它和使用分离，就存在窗口。**能合并成一次操作就合并**。

### 6. 和其它并发问题的关系

TOCTOU 和前面讲的死锁、数据竞争是**不同维度**的问题：

| 问题 | 本质 |
| :--- | :--- |
| 数据竞争 | 同一时刻的并发访问未同步 |
| 死锁 | 互相等待资源 |
| **TOCTOU** | **两个时刻之间的状态变化** |

数据竞争是「同时」，TOCTOU 是「先后」。加锁解决的是前者，而后者需要**保证检查与使用之间不被打断**——这往往意味着重新设计接口，而不只是加一把锁。

## 七、常见误区

**「`std::mutex` 可以拷贝」** —— 不能。互斥量不可拷贝也不可移动。

**「加锁后忘记解锁只影响性能」** —— 会导致其它线程永久阻塞，是死锁。

**「`recursive_mutex` 能解决所有嵌套加锁问题」** —— 它掩盖了设计问题。更好的做法是明确加锁边界。

**「`shared_mutex` 总是比 `mutex` 快」** —— 只在读多写少时才有优势，且实现更复杂。读写均衡时 `mutex` 更快。

**「锁的粒度越细越好」** —— 每把锁都有开销（内存、加锁时间）。过度分片反而可能变慢。

**「`scoped_lock` 比 `lock_guard` 慢」** —— 单个互斥量时两者性能相当，`scoped_lock` 只是更通用。

**「`lock_guard` 可以手动解锁」** —— 不能。需要手动控制时用 `unique_lock`。

**「加了锁就绝对安全」** —— 不一定。检查和操作被拆到两个临界区，仍然存在 TOCTOU。

**「返回引用比返回值高效，所以更好」** —— 在并发环境下，返回引用会把共享数据暴露到锁的保护范围之外。

## 八、相关章节

- [线程基础](./Thread.md)：线程的创建与管理
- [条件变量](./Condition_Variable.md)：等待条件成立
- [原子操作](./Atomic.md)：更轻量的同步方式
- [并发模式与陷阱](./Patterns.md)：死锁排查与工程实践
- [RAII 与资源管理](../Memory/RAII.md)：锁包装器的设计思想
