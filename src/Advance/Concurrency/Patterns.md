# 并发模式与陷阱

前面几节讲的是工具：线程、锁、条件变量、原子操作、异步任务。这一节讲怎么把它们组装成能用的东西。

## 一、线程池

### 1. 为什么需要线程池

每来一个任务就 `std::thread` 创建一个，看起来简单，实际代价很大：

- **创建开销**：一个线程的创建涉及内核对象、栈分配（默认 1~8 MB），大约几十微秒
- **销毁开销**：线程退出也要清理
- **数量失控**：任务多了线程就多，可能耗尽内存或让调度器崩溃

线程池的思路是：**预先创建固定数量的线程，任务来了放进队列，空闲线程去取**。

```
任务提交 → [任务队列] → 工作线程 1
                      → 工作线程 2
                      → 工作线程 3
```

### 2. 一个可用的实现

```c++
#include <condition_variable>
#include <functional>
#include <future>
#include <mutex>
#include <queue>
#include <thread>
#include <vector>

class ThreadPool
{
public:
    explicit ThreadPool(std::size_t n = std::thread::hardware_concurrency())
    {
        for (std::size_t i = 0; i < n; ++i)
        {
            workers_.emplace_back([this] { workerLoop(); });
        }
    }

    ~ThreadPool()
    {
        {
            std::lock_guard lock(mtx_);
            stop_ = true;
        }
        cv_.notify_all();
        for (auto& t : workers_)
            t.join();
    }

    // 提交任务，返回 future
    template<class F, class... Args>
    auto submit(F&& f, Args&&... args)
        -> std::future<std::invoke_result_t<F, Args...>>
    {
        using ReturnType = std::invoke_result_t<F, Args...>;

        // 用 shared_ptr 包装，因为 packaged_task 不可拷贝
        auto task = std::make_shared<std::packaged_task<ReturnType()>>(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
        );

        std::future<ReturnType> fut = task->get_future();

        {
            std::lock_guard lock(mtx_);
            if (stop_)
                throw std::runtime_error("线程池已停止");
            tasks_.emplace([task] { (*task)(); });
        }
        cv_.notify_one();

        return fut;
    }

private:
    void workerLoop()
    {
        while (true)
        {
            std::function<void()> task;

            {
                std::unique_lock lock(mtx_);
                cv_.wait(lock, [this] { return stop_ || !tasks_.empty(); });

                if (stop_ && tasks_.empty())
                    return;   // 停止且无剩余任务

                task = std::move(tasks_.front());
                tasks_.pop();
            }

            task();   // 在锁外执行
        }
    }

    std::vector<std::thread> workers_;
    std::queue<std::function<void()>> tasks_;
    std::mutex mtx_;
    std::condition_variable cv_;
    bool stop_ = false;
};
```

使用：

```c++
ThreadPool pool(4);

std::vector<std::future<int>> results;
for (int i = 0; i < 10; ++i)
{
    results.push_back(pool.submit([i] {
        return i * i;
    }));
}

for (auto& f : results)
    std::cout << f.get() << '\n';
```

### 3. 几个设计要点

**任务在锁外执行**。`task()` 必须在解锁后调用，否则工作线程持锁执行任务，其它线程无法取任务，线程池退化成了单线程。

**`packaged_task` 不可拷贝**。用 `shared_ptr` 包装才能放进 `std::function`（它要求可拷贝）。

**析构要等所有任务完成**。`stop_` 置位后 `notify_all`，工作线程取完剩余任务才退出。如果直接 `detach`，析构后任务还在跑，可能访问已销毁的对象。

**线程数怎么定**：

| 任务类型 | 建议线程数 |
| :--- | :--- |
| CPU 密集 | `hardware_concurrency()` |
| I/O 密集 | 更多（2~4 倍） |
| 混合 | 分成两个池，分别调优 |

### 4. 有界队列与背压

上面的实现队列无界——任务提交太快会耗尽内存。生产环境需要背压：

```c++
template<class T>
class BoundedQueue
{
public:
    explicit BoundedQueue(std::size_t cap) : cap_(cap) {}

    void push(T item)
    {
        std::unique_lock lock(mtx_);
        notFull_.wait(lock, [this] { return q_.size() < cap_; });
        q_.push(std::move(item));
        notEmpty_.notify_one();
    }

    T pop()
    {
        std::unique_lock lock(mtx_);
        notEmpty_.wait(lock, [this] { return !q_.empty(); });
        T item = std::move(q_.front());
        q_.pop();
        notFull_.notify_one();
        return item;
    }

private:
    std::mutex mtx_;
    std::condition_variable notEmpty_, notFull_;
    std::queue<T> q_;
    std::size_t cap_;
};
```

**背压**的意思是：队列满了就让提交者等待，而不是无限堆积。这是保护系统不被打垮的关键机制。

## 二、回调与事件驱动

### 1. 回调注册模式

事件驱动系统里，组件不直接调用彼此，而是注册回调：

```c++
class EventLoop
{
public:
    using Callback = std::function<void()>;

    void onRead(int fd, Callback cb)
    {
        readHandlers_[fd] = std::move(cb);
    }

    void onWrite(int fd, Callback cb)
    {
        writeHandlers_[fd] = std::move(cb);
    }

    void run()
    {
        while (!stop_)
        {
            auto events = poll();   // 等待事件

            for (auto& ev : events)
            {
                auto it = (ev.type == Event::Read)
                    ? readHandlers_.find(ev.fd)
                    : writeHandlers_.find(ev.fd);

                if (it != readHandlers_.end() && it->second)
                    it->second();   // 调用回调
            }
        }
    }

private:
    std::unordered_map<int, Callback> readHandlers_;
    std::unordered_map<int, Callback> writeHandlers_;
    bool stop_ = false;
};
```

这就是 `epoll` / `kqueue` / `IOCP` 这类 I/O 多路复用库的抽象方式。一个线程处理成千上万个连接，靠的是「事件来了才处理」。

### 2. 回调的线程安全

回调可能在任意线程被调用，这带来两个问题：

**回调内部访问共享数据要加锁**：

```c++
void onData(Data d)
{
    std::lock_guard lock(mtx_);
    buffer_.push_back(d);
}
```

**回调注册与调用可能并发**。如果 `run()` 在遍历 handler 时另一个线程修改了 map，会崩溃。常见做法是：

- 用「注册到事件循环线程」的队列，避免跨线程修改
- 或者用读写锁保护

```c++
class EventLoop
{
    std::shared_mutex mtx_;

    void onRead(int fd, Callback cb)
    {
        std::unique_lock lock(mtx_);
        readHandlers_[fd] = std::move(cb);
    }

    void dispatch(int fd)
    {
        std::shared_lock lock(mtx_);   // 读锁，多个分发可并发
        auto it = readHandlers_.find(fd);
        if (it != readHandlers_.end())
            it->second();
    }
};
```

### 3. 事件驱动的优势与代价

**优势**：

- 单线程处理大量连接，无线程切换开销
- 内存占用小（每个连接不需要一个线程栈）
- 没有锁竞争（如果都在一个线程里）

**代价**：

- 一个回调阻塞，所有连接都卡住
- 无法利用多核（除非跑多个事件循环）
- 控制流被事件切碎，调试困难

## 三、其它常见模式

### 1. 读写锁保护缓存

```c++
class Cache
{
    std::shared_mutex mtx_;
    std::unordered_map<int, std::string> data_;

public:
    std::optional<std::string> get(int key)
    {
        std::shared_lock lock(mtx_);   // 读并发
        auto it = data_.find(key);
        if (it == data_.end()) return std::nullopt;
        return it->second;
    }

    void put(int key, std::string value)
    {
        std::unique_lock lock(mtx_);   // 写独占
        data_[key] = std::move(value);
    }
};
```

### 2. 双重检查锁定（DCLP）

延迟初始化的经典写法：

```c++
class Singleton
{
public:
    static Singleton* instance()
    {
        Singleton* tmp = instance_.load(std::memory_order_acquire);
        if (!tmp)   // 第一次检查，避免每次都加锁
        {
            std::lock_guard lock(mtx_);
            tmp = instance_.load(std::memory_order_relaxed);
            if (!tmp)   // 第二次检查，防止重复初始化
            {
                tmp = new Singleton();
                instance_.store(tmp, std::memory_order_release);
            }
        }
        return tmp;
    }

private:
    static std::atomic<Singleton*> instance_;
    static std::mutex mtx_;
};
```

**但通常不需要这么写**。C++11 起局部静态变量是线程安全的：

```c++
Singleton& instance()
{
    static Singleton inst;
    return inst;
}
```

一行搞定，编译器保证只初始化一次。

### 3. Future 模式

把「计算」和「取结果」解耦：

```c++
std::future<Image> loadAsync(const std::string& path)
{
    return std::async(std::launch::async, [path] {
        return decodeImage(readFile(path));
    });
}

// 调用者可以先去干别的
auto fut = loadAsync("photo.jpg");
doOtherWork();
Image img = fut.get();
```

## 四、常见陷阱汇总

### 1. 数据竞争

| 问题 | 表现 |
| :--- | :--- |
| 未同步的共享读写 | 结果不确定、偶发崩溃 |
| 用 `volatile` 代替 `atomic` | 编译器优化导致读写丢失 |
| 复合操作非原子 | `if (x) x--` 整体不加锁 |

**排查工具**：ThreadSanitizer（`-fsanitize=thread`）。

### 2. 死锁

| 问题 | 表现 |
| :--- | :--- |
| 加锁顺序不一致 | 两个线程互相等待 |
| 异常路径未解锁 | 未用 RAII |
| 持锁调用外部代码 | 外部代码又加锁 |

**排查工具**：GDB 查看各线程栈、`pstack`、死锁检测器。

### 3. TOCTOU

| 问题 | 表现 |
| :--- | :--- |
| 检查和使用拆在两个临界区 | 中间状态被改变，逻辑错误 |
| 返回共享数据的引用 | 指针逃出锁的保护范围 |
| 异步回调里「先判断后操作」 | 回调触发时判断已过期 |

**特点**：加锁了也可能存在。加锁保证单个操作的原子性，不保证「检查 + 使用」这一对操作的原子性。

**应对**：扩大临界区、返回拷贝而非引用、改用 EAFP 风格。详见[互斥量与锁](./Mutex.md)。

### 4. 生命周期

| 问题 | 表现 |
| :--- | :--- |
| 线程访问已销毁对象 | 悬空指针、随机崩溃 |
| 回调捕获 `this` | 对象先于回调销毁 |
| `detach` 后不管 | 程序退出时线程仍在跑 |

**原则**：线程和回调的生命周期必须被明确管理，不要 `detach` 了事。

### 5. 性能

| 问题 | 表现 |
| :--- | :--- |
| 锁粒度过大 | 线程大量串行等待 |
| 伪共享 | 不同线程写同一缓存行的不同变量 |
| 线程过多 | 上下文切换开销超过计算收益 |
| 频繁创建线程 | 创建/销毁开销 |

**伪共享**值得单独说：两个线程分别写相邻的两个 `int`，如果它们落在同一个缓存行（通常 64 字节），CPU 会反复使缓存行失效：

```c++
// 不好：两个计数器可能在同一缓存行
struct Counters
{
    std::atomic<int> a;
    std::atomic<int> b;
};

// 好：用填充隔开
struct Counters
{
    alignas(64) std::atomic<int> a;
    alignas(64) std::atomic<int> b;
};
```

C++17 还提供了 `std::hardware_destructive_interference_size` 表示缓存行大小。

### 6. 调试困难

并发 bug 的特点：**偶发、不可复现、换个机器就消失**。

应对方法：

- **ThreadSanitizer**：检测数据竞争，编译时加 `-fsanitize=thread`
- **压力测试**：加大并发度和迭代次数
- **确定性调度**：用工具（如 rr）记录重放
- **代码审查**：重点看共享数据的访问路径

## 五、几条经验

**能用简单方案就别用复杂的**。`std::mutex` 保护一切，往往比精心设计的无锁结构更可靠。

**先测量再优化**。并发性能问题常常不在你猜的地方。

**共享可变状态越少越好**。如果数据可以只被一个线程访问，就根本不需要同步。

**优先用消息传递而非共享内存**。线程间通过队列传数据，比共享数据结构加锁更容易推理。

**给线程和回调明确的归属**。谁创建、谁销毁、什么时候能安全访问，这些必须清楚。

## 六、常见误区

**「线程池线程越多越快」** —— 超过核心数后，上下文切换开销会抵消收益。

**「`detach` 可以省去 join 的麻烦」** —— 它把生命周期问题变成了崩溃。除非线程真的独立于所有对象，否则别 detach。

**「双重检查锁定比局部静态变量好」** —— C++11 起局部静态变量就是线程安全的，且更简洁。

**「无锁队列一定比加锁队列快」** —— 中等争用下往往更慢，且正确性极难保证。

**「伪共享只是理论问题」** —— 在高频写入场景下可以造成数倍性能差距。

**「并发 bug 可以靠加日志调试」** —— 日志会改变时序，bug 可能因此消失。用专门的检测工具。

**「加了锁就不会有并发问题了」** —— 锁只解决数据竞争。检查和操作分离导致的 TOCTOU、以及生命周期问题，加锁都挡不住。

## 七、相关章节

- [线程基础](./Thread.md)：线程的创建、管理与生命周期
- [互斥量与锁](./Mutex.md)：锁的选择与死锁避免
- [条件变量](./Condition_Variable.md)：任务队列的等待机制
- [原子操作](./Atomic.md)：无锁与伪共享
- [异步任务](./Async.md)：future、回调与协程
- [内存模型](../Memory/Memory_Model.md)：happens-before 与可见性
