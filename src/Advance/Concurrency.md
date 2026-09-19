# 多线程与并发

C++11 起，标准库提供了完整的并发支持：线程、互斥量、条件变量、原子操作和异步任务。这些设施定义在 `<thread>`、`<mutex>`、`<condition_variable>`、`<atomic>`、`<future>` 等头文件中，让并发程序不再依赖平台特定的 API。

并发编程的困难不在于 API，而在于**正确性**：数据竞争、死锁、虚假唤醒、内存序等问题往往在测试中难以复现，却会在生产中造成严重后果。

> [!WARNING]
> **并发问题的本质**
>
> 只要多个线程同时访问同一块内存，且其中至少一个是写操作，且没有同步措施，就构成**数据竞争（data race）**——这是未定义行为，编译器可以做任何假设，结果完全不可预测。
>
> 加锁或使用原子操作是消除数据竞争的唯一途径。

## 一、线程

### 1. std::thread

`std::thread` 在构造时立即启动线程，顶层函数的返回值被忽略：

```c++
#include <iostream>
#include <thread>

void hello(int id)
{
    std::cout << "线程 " << id << " 运行中\n";
}

int main()
{
    std::thread t1(hello, 1);              // 启动线程
    std::thread t2([] { hello(2); });      // 用 lambda

    t1.join();   // 等待 t1 结束
    t2.join();   // 等待 t2 结束
}
```

**必须显式 `join()` 或 `detach()`**。如果 `thread` 对象在析构时仍处于 joinable 状态，程序会直接调用 `std::terminate`：

```c++
{
    std::thread t(hello, 1);
}   // 错误：t 析构时未 join/detach，程序终止
```

| 操作 | 含义 |
| :--- | :--- |
| `join()` | 阻塞等待线程结束 |
| `detach()` | 让线程独立运行，不再与 `thread` 对象关联 |
| `joinable()` | 是否关联着一个线程 |
| `get_id()` | 返回线程 ID |
| `hardware_concurrency()` | 返回硬件支持的并发线程数（静态函数） |

### 2. std::jthread（C++20）

`std::jthread` 解决了 `thread` 的两大痛点：**析构时自动 join**，以及**支持协作式取消**：

```c++
#include <thread>

int main()
{
    std::jthread t([](std::stop_token st) {
        while (!st.stop_requested())
        {
            // 执行任务
        }
    });
}   // 离开作用域时自动请求停止并 join
```

`jthread` 内部持有一个 `std::stop_source`，构造时会把它对应的 `std::stop_token` 作为第一个参数传给线程函数。线程通过 `stop_requested()` 检查是否收到停止请求。

| 成员函数 | 作用 |
| :--- | :--- |
| `request_stop()` | 请求停止 |
| `get_stop_token()` | 获取停止令牌 |
| `get_stop_source()` | 获取停止源 |

**优先使用 `jthread`**：它不会因为忘记 `join` 而终止程序，且提供了标准的取消机制。

### 3. 当前线程操作

```c++
#include <thread>
#include <chrono>

std::this_thread::get_id();                          // 当前线程 ID
std::this_thread::sleep_for(std::chrono::seconds(1)); // 睡眠 1 秒
std::this_thread::sleep_until(tp);                   // 睡眠到指定时间点
std::this_thread::yield();                           // 让出 CPU
```

## 二、互斥量与锁

### 1. 互斥量类型

| 类型 | 头文件 | 说明 |
| :--- | :--- | :--- |
| `std::mutex` | `<mutex>` | 基本互斥量 |
| `std::recursive_mutex` | `<mutex>` | 同一线程可重复加锁 |
| `std::timed_mutex` | `<mutex>` | 支持超时加锁 |
| `std::shared_mutex` | `<shared_mutex>` | 读写锁，支持多个读者或一个写者（C++17） |

### 2. RAII 锁包装器

**不要手动调用 `lock()` / `unlock()`**，应使用 RAII 包装器，保证异常安全：

| 包装器 | 版本 | 特点 |
| :--- | :--- | :--- |
| `std::lock_guard` | C++11 | 最简单，构造时加锁，析构时解锁，不可移动 |
| `std::unique_lock` | C++11 | 可移动、可延迟加锁、可手动解锁，配合条件变量使用 |
| `std::scoped_lock` | C++17 | 可同时锁定**多个**互斥量，且避免死锁 |
| `std::shared_lock` | C++14 | 配合 `shared_mutex` 实现共享锁定 |

```c++
#include <mutex>

std::mutex mtx;
int counter = 0;

void increment()
{
    std::lock_guard<std::mutex> lock(mtx);   // 自动加锁
    ++counter;
}   // 自动解锁，即使中途抛异常
```

### 3. 同时锁定多个互斥量

需要锁定多个互斥量时，用 `std::scoped_lock`（C++17）可以避免死锁：

```c++
std::mutex m1, m2;

void transfer()
{
    // 同时锁定两个，内部使用死锁避免算法
    std::scoped_lock lock(m1, m2);
    // 操作共享数据
}
```

如果手动按不同顺序加锁，两个线程可能互相等待对方释放，形成**死锁**：

```c++
// 线程 A：lock(m1) 然后 lock(m2)
// 线程 B：lock(m2) 然后 lock(m1)
// 可能死锁
```

### 4. 延迟加锁

`std::unique_lock` 支持延迟加锁，这在配合条件变量时是必需的：

```c++
std::unique_lock<std::mutex> lock(mtx, std::defer_lock);   // 暂不加锁
// ... 做一些准备工作
lock.lock();   // 手动加锁
```

可用的标签：`std::defer_lock`、`std::try_to_lock`、`std::adopt_lock`。

### 5. call_once

`std::call_once` 保证某个函数在多线程环境下**只被执行一次**，常用于单例模式初始化：

```c++
std::once_flag flag;

void init()
{
    std::call_once(flag, [] {
        // 只会执行一次
    });
}
```

## 三、条件变量

条件变量用于让线程**等待某个条件成立**，需要配合互斥量和谓词使用：

```c++
#include <condition_variable>
#include <mutex>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> q;

void producer()
{
    {
        std::lock_guard<std::mutex> lock(mtx);
        q.push(42);
    }
    cv.notify_one();   // 通知等待的线程
}

void consumer()
{
    std::unique_lock<std::mutex> lock(mtx);

    // 必须用谓词循环，防止虚假唤醒
    cv.wait(lock, [] { return !q.empty(); });

    int value = q.front();
    q.pop();
}
```

> [!WARNING]
> **必须使用带谓词的 `wait`**
>
> 条件变量可能发生**虚假唤醒（spurious wakeup）**——即使没有 `notify`，`wait` 也可能返回。因此必须用循环重新检查条件：
>
> ```c++
> // 错误：可能被虚假唤醒
> cv.wait(lock);
> if (q.empty()) { /* 条件其实不成立 */ }
>
> // 正确：带谓词的版本内部就是循环
> cv.wait(lock, [] { return !q.empty(); });
> ```

`condition_variable` 只能配合 `std::unique_lock<std::mutex>`；如果需要配合其它锁类型，使用 `std::condition_variable_any`。

## 四、原子操作

### 1. std::atomic

对于简单的共享变量，原子操作比加锁更轻量：

```c++
#include <atomic>

std::atomic<int> counter{0};

void increment()
{
    ++counter;   // 原子操作，无需加锁
}
```

`std::atomic` 支持的类型包括整型、指针、布尔型，C++20 起还包括浮点型。对于自定义类型，可以用 `std::atomic<T>` 但要求 `T` 是可平凡复制的，且实现可能使用内部锁。

```c++
std::atomic<bool> flag{false};
flag.store(true);              // 写
bool v = flag.load();          // 读
bool old = flag.exchange(false);  // 交换并返回旧值
```

### 2. 比较并交换（CAS）

`compare_exchange_weak` / `compare_exchange_strong` 是无锁编程的基础，用于实现「读取-修改-写回」的原子版本：

```c++
std::atomic<int> value{0};

void update(int expected, int desired)
{
    // 如果 value == expected，则设为 desired，返回 true
    // 否则把 value 的当前值写入 expected，返回 false
    value.compare_exchange_strong(expected, desired);
}
```

`weak` 版本可能**虚假失败**（即使值相等也返回 false），因此在循环中使用；`strong` 版本不会虚假失败。

### 3. 内存序

原子操作默认使用 `std::memory_order_seq_cst`（顺序一致性），这是最严格也最慢的选项。其它选项包括：

| 内存序 | 含义 |
| :--- | :--- |
| `memory_order_seq_cst` | 顺序一致，所有线程看到相同的操作顺序（默认） |
| `memory_order_acquire` | 用于读，保证之后的读写不会被重排到它之前 |
| `memory_order_release` | 用于写，保证之前的读写不会被重排到它之后 |
| `memory_order_acq_rel` | 同时具备 acquire 和 release 语义 |
| `memory_order_relaxed` | 只保证操作的原子性，不提供同步 |

```c++
std::atomic<bool> ready{false};
int data = 0;

// 线程 A
data = 42;
ready.store(true, std::memory_order_release);   // 之前的写入对读者可见

// 线程 B
while (!ready.load(std::memory_order_acquire)) { }   // 之后的读取能看到 data = 42
std::cout << data;
```

内存序的细节见 [C++内存模型（并发）](./Memory/Memory_Model.md)。

### 4. 原子操作的局限

- **复合操作不是原子的**。`counter++` 是原子的，但 `if (counter > 0) counter--;` 不是——两步之间可能被其它线程插入。
- **无法保护多个变量**。需要同时修改多个相关变量时，仍应使用互斥量。
- **`std::atomic` 不可拷贝**。它没有拷贝构造和拷贝赋值（因为拷贝本身无法原子完成）。

## 五、异步任务

`<future>` 提供了更高层的并发抽象，让调用者可以「稍后取回结果」，而不必手动管理线程。

### 1. std::async

最简单的异步执行方式：

```c++
#include <future>
#include <iostream>

int compute(int x)
{
    return x * x;
}

int main()
{
    // 异步执行，返回 future
    std::future<int> f = std::async(std::launch::async, compute, 5);

    // ... 做其它事情

    int result = f.get();   // 阻塞等待结果
    std::cout << result << '\n';   // 25
}
```

启动策略：

| 策略 | 含义 |
| :--- | :--- |
| `std::launch::async` | 强制在新线程中执行 |
| `std::launch::deferred` | 延迟到调用 `get()` 时才执行（同步） |
| `std::launch::async \| std::launch::deferred` | 由实现决定（默认） |

> [!WARNING]
> **默认策略可能不启动新线程**
>
> 不指定策略时，实现可能选择 `deferred`，导致任务在 `get()` 时才同步执行。如果需要真正的并发，必须显式指定 `std::launch::async`。

### 2. promise 与 future

`std::promise` 是写入端，`std::future` 是读取端，通过共享状态传递值或异常：

```c++
#include <future>
#include <thread>

void worker(std::promise<int> p)
{
    try
    {
        p.set_value(42);                              // 设置值
        // p.set_exception(std::make_exception_ptr(e));  // 或设置异常
    }
    catch (...)
    {
        p.set_exception(std::current_exception());
    }
}

int main()
{
    std::promise<int> p;
    std::future<int> f = p.get_future();

    std::thread t(worker, std::move(p));

    std::cout << f.get() << '\n';   // 42，异常会在这里重新抛出
    t.join();
}
```

### 3. packaged_task

`std::packaged_task` 把一个可调用对象包装起来，调用它时结果会自动存入关联的 future：

```c++
#include <future>

int add(int a, int b) { return a + b; }

int main()
{
    std::packaged_task<int(int, int)> task(add);
    std::future<int> f = task.get_future();

    std::thread t(std::move(task), 3, 4);
    std::cout << f.get() << '\n';   // 7
    t.join();
}
```

### 4. 三种方式对比

| 方式 | 适用场景 |
| :--- | :--- |
| `std::async` | 简单的「发射后取结果」，最易用 |
| `std::promise` / `future` | 需要手动控制何时设置结果 |
| `std::packaged_task` | 把已有函数包装成任务，可放入队列由线程池执行 |

## 六、C++20 的同步原语

### 1. latch（单次闩锁）

`std::latch` 是一个**一次性**的倒计数器，用于等待多个线程完成：

```c++
#include <latch>
#include <thread>
#include <vector>

int main()
{
    std::latch done(3);   // 计数为 3

    auto worker = [&done] {
        // 执行任务
        done.count_down();   // 计数减 1
    };

    std::vector<std::thread> threads;
    for (int i = 0; i < 3; ++i)
        threads.emplace_back(worker);

    done.wait();   // 等待计数归零

    for (auto& t : threads) t.join();
}
```

### 2. barrier（可重用屏障）

`std::barrier` 与 `latch` 类似，但**可以重复使用**，适合多轮迭代的并行算法：

```c++
#include <barrier>

std::barrier sync(4);   // 每 4 个线程同步一次

void phase_worker()
{
    for (int i = 0; i < 10; ++i)
    {
        // 第一阶段工作
        sync.arrive_and_wait();   // 等待所有线程到达

        // 第二阶段工作
        sync.arrive_and_wait();
    }
}
```

### 3. semaphore（信号量）

`std::counting_semaphore` 用于限制对资源的并发访问数量：

```c++
#include <semaphore>

// 最多允许 3 个线程同时访问
std::counting_semaphore<3> sem(3);

void limited_access()
{
    sem.acquire();     // 获取许可，计数为 0 时阻塞
    // 访问受限资源
    sem.release();     // 释放许可
}
```

`std::binary_semaphore` 是计数为 1 的特例，可用于线程间的信号通知。

## 七、常见并发问题

| 问题 | 原因 | 避免方式 |
| :--- | :--- | :--- |
| **数据竞争** | 多线程无同步地访问同一内存 | 加锁或使用原子操作 |
| **死锁** | 多个线程循环等待对方持有的锁 | 用 `scoped_lock` 同时加锁；统一加锁顺序 |
| **活锁** | 线程不断重试但无法推进 | 引入随机退避 |
| **虚假唤醒** | 条件变量无故返回 | 使用带谓词的 `wait` |
| **ABA 问题** | 值从 A 改为 B 又改回 A，CAS 无法察觉 | 使用带版本号的原子类型 |
| **迭代器失效** | 一个线程修改容器，另一个在遍历 | 遍历时加锁，或使用不可变数据结构 |

## 八、注意事项

- **`std::thread` 必须 join 或 detach**。否则析构时程序终止。优先用 `std::jthread`。
- **不要手动 lock/unlock**。始终使用 RAII 包装器，异常安全才有保障。
- **条件变量必须用带谓词的 `wait`**。否则虚假唤醒会导致逻辑错误。
- **`std::async` 默认策略不确定**。需要真并发时必须显式指定 `std::launch::async`。
- **`future` 的析构会阻塞**。`std::async` 返回的 `future` 如果未调用 `get()` 就析构，会等待任务完成。
- **原子操作不等于无锁**。`std::atomic<T>` 对某些类型可能内部使用锁，可用 `is_lock_free()` 检查。
- **内存序默认最严格**。`seq_cst` 最慢但最安全，优化前应先确认正确性。
- **并发 bug 难以复现**。应借助 ThreadSanitizer（`-fsanitize=thread`）等工具检测。

## 九、相关章节

- [C++内存模型（并发）](./Memory/Memory_Model.md)：内存序与 happens-before 关系
- [异常处理](./Exception.md)：线程函数中异常的传播
- [智能指针](./Memory/Smart_Pointer.md)：`shared_ptr` 的线程安全性
- [C++实践 · 多线程](../Practice.md)：工程中的并发模式
