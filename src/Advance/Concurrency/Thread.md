# 线程基础

线程是操作系统调度的基本单位。一个进程可以包含多个线程，它们共享地址空间，各自拥有独立的栈和寄存器状态。

共享地址空间是线程轻量的原因，也是并发 bug 的根源——所有全局变量、堆对象都是共享的。

## 一、std::thread

`std::thread` 在**构造时立即启动**线程，不需要单独调用 `start()`：

```c++
#include <iostream>
#include <thread>

void hello(int id)
{
    std::cout << "线程 " << id << " 运行中\n";
}

int main()
{
    std::thread t1(hello, 1);              // 启动线程，参数按值传递
    std::thread t2([] { hello(2); });      // 用 lambda

    t1.join();   // 等待 t1 结束
    t2.join();   // 等待 t2 结束
}
```

参数默认是**拷贝**到线程内部的。需要传引用时必须用 `std::ref`：

```c++
void modify(std::vector<int>& v) { v.push_back(1); }

std::vector<int> data;
std::thread t(modify, std::ref(data));   // 不加 ref 会编译失败
t.join();
```

不加 `std::ref` 会编译失败其实是好事——它提醒你引用传递有生命周期风险。如果线程比被引用对象活得久，就会访问已销毁的对象。

### 1. 必须 join 或 detach

`std::thread` 析构时如果仍处于 joinable 状态，程序会直接调用 `std::terminate`：

```c++
{
    std::thread t(hello, 1);
}   // 错误：t 析构时未 join/detach，程序终止
```

这个设计是刻意的：标准委员会认为「线程还在跑但句柄没了」是严重错误，与其静默泄漏不如直接崩掉。

```c++
std::thread t(hello, 1);
t.join();      // 等待结束
// 或
t.detach();    // 放手不管，线程在后台运行
```

`detach()` 要慎用——分离后的线程无法再被等待或控制，程序退出时它可能还在访问已销毁的全局对象。

### 2. 常用接口

| 接口 | 含义 |
| :--- | :--- |
| `join()` | 阻塞等待线程结束 |
| `detach()` | 让线程独立运行，不再与 `thread` 对象关联 |
| `joinable()` | 是否关联着一个线程 |
| `get_id()` | 返回线程 ID |
| `hardware_concurrency()` | 硬件支持的并发线程数（静态函数） |

`hardware_concurrency()` 常用于决定线程池大小：

```c++
unsigned n = std::thread::hardware_concurrency();
if (n == 0) n = 4;   // 无法确定时的兜底值
```

它返回的只是「建议值」，可能受 CPU 亲和性、容器配额等因素影响。

### 3. 异常与线程

线程函数中抛出的异常**不会传播到创建它的线程**。如果异常逃出线程函数，会直接调用 `std::terminate`：

```c++
std::thread t([] {
    throw std::runtime_error("出错了");
    // 异常逃出 → std::terminate
});
t.join();
```

需要把异常传回主线程，得用 `std::promise` 或 `std::exception_ptr`：

```c++
std::promise<void> p;
std::thread t([&p] {
    try {
        doWork();
        p.set_value();
    } catch (...) {
        p.set_exception(std::current_exception());
    }
});

try {
    p.get_future().get();   // 异常在这里重新抛出
} catch (const std::exception& e) {
    std::cerr << e.what() << '\n';
}
t.join();
```

## 二、std::jthread（C++20）

`std::jthread` 解决了 `thread` 的两个痛点：**析构时自动 join**，以及**内置协作式取消**。

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
}   // 离开作用域时自动 request_stop() 并 join
```

### 1. 自动 join

`jthread` 析构时会先请求停止，再 join。因此不会出现「忘记 join 导致 terminate」的问题：

```c++
{
    std::jthread t(doWork);
}   // 自动停止并等待
```

### 2. 协作式取消

`jthread` 内部持有一个 `std::stop_source`，构造时把对应的 `std::stop_token` 作为**第一个参数**传给线程函数：

```c++
void worker(std::stop_token st, int id)
{
    while (!st.stop_requested())
    {
        // 干活
    }
    std::cout << "线程 " << id << " 收到停止请求\n";
}

std::jthread t(worker, 1);

// 主线程可以主动请求停止
t.request_stop();
```

| 成员函数 | 作用 |
| :--- | :--- |
| `request_stop()` | 请求停止，返回是否成功 |
| `get_stop_token()` | 获取停止令牌 |
| `get_stop_source()` | 获取停止源 |

**注意「协作式」的含义**：`request_stop()` 只是设置一个标志，线程必须**主动检查** `stop_requested()` 才能响应。如果线程卡在阻塞调用里，停止请求不会生效。

### 3. stop_callback

可以在停止请求发出时执行回调：

```c++
std::stop_token st = t.get_stop_token();

std::stop_callback cb(st, [] {
    std::cout << "收到停止请求\n";
});
```

这在需要唤醒阻塞操作时很有用——比如让 `condition_variable` 停止等待。C++20 的 `condition_variable_any` 提供了支持 `stop_token` 的 `wait` 重载：

```c++
std::condition_variable_any cv;
std::mutex mtx;

void worker(std::stop_token st)
{
    std::unique_lock lock(mtx);
    // 收到停止请求时自动唤醒
    cv.wait(lock, st, [] { return /* 条件 */; });
}
```

## 三、当前线程操作

`std::this_thread` 命名空间提供了一组操作当前线程的函数：

```c++
#include <thread>
#include <chrono>

std::this_thread::get_id();                            // 当前线程 ID
std::this_thread::sleep_for(std::chrono::seconds(1));  // 睡眠 1 秒
std::this_thread::sleep_until(tp);                     // 睡眠到指定时间点
std::this_thread::yield();                             // 让出 CPU
```

### 1. sleep_for 与 sleep_until

```c++
// 相对时间
std::this_thread::sleep_for(std::chrono::milliseconds(100));

// 绝对时间
auto deadline = std::chrono::steady_clock::now() + std::chrono::seconds(1);
std::this_thread::sleep_until(deadline);
```

在循环中做定时任务时，`sleep_until` 比 `sleep_for` 更准——它不会累积每次循环的执行时间误差：

```c++
// 不好：误差会累积
while (running) {
    doWork();
    std::this_thread::sleep_for(100ms);
}

// 好：固定周期
auto next = std::chrono::steady_clock::now();
while (running) {
    doWork();
    next += 100ms;
    std::this_thread::sleep_until(next);
}
```

### 2. yield

`yield()` 建议调度器切换到其它线程，但**不保证**真的切换：

```c++
while (!flag.load()) {
    std::this_thread::yield();   // 自旋等待时让出 CPU
}
```

在自旋锁或忙等待中加 `yield()` 可以降低 CPU 占用，但不如用条件变量或原子等待高效。

## 四、线程局部存储

`thread_local` 声明的变量**每个线程各有一份独立副本**：

```c++
thread_local int counter = 0;   // 每个线程独立

void increment()
{
    ++counter;   // 不构成数据竞争
}
```

这是避免共享状态的利器。典型用途：

**随机数引擎**（引擎不是线程安全的）：

```c++
void worker()
{
    thread_local std::mt19937 gen(std::random_device{}());
    // 每个线程独立的引擎
}
```

**线程专属的缓存**：

```c++
thread_local std::vector<char> buffer;   // 复用缓冲区，避免反复分配
```

**错误状态**（类似 `errno`）：

```c++
thread_local int last_error = 0;
```

几点说明：

- `thread_local` 变量的初始化在**首次使用时**发生，每个线程各初始化一次
- 线程结束时销毁，析构顺序与构造顺序相反
- 有性能开销：访问 `thread_local` 需要查 TLS 表，比普通全局变量慢
- `thread_local` 与 `static` 可以组合，表示「每个线程的静态变量」

## 五、线程数量与开销

线程不是免费的：

| 开销项 | 说明 |
| :--- | :--- |
| **栈空间** | 每个线程默认 1~8 MB（Linux 通常 8 MB） |
| **创建成本** | 涉及系统调用，通常几十微秒 |
| **上下文切换** | 每次切换几微秒，频繁切换会显著拖慢程序 |
| **缓存失效** | 切换后 CPU 缓存可能失效 |

因此：

- **不要为每个任务创建一个线程**。任务数量大时用线程池。
- **线程数通常设在 CPU 核心数附近**。计算密集型任务，线程数超过核心数没有收益。
- **I/O 密集型任务可以多设一些**。线程阻塞在 I/O 上时不占 CPU。

```c++
unsigned n = std::thread::hardware_concurrency();
std::vector<std::jthread> pool;
for (unsigned i = 0; i < n; ++i)
    pool.emplace_back(worker, i);
```

## 六、常见误区

**「`std::thread` 会自动 join」** —— 不会。析构时如果还 joinable，程序直接终止。用 `jthread` 可以避免。

**「`detach()` 之后就安全了」** —— 分离的线程可能访问已销毁的对象。除非线程完全自包含，否则不要 detach。

**「线程函数抛异常会传播到主线程」** —— 不会，会直接 `terminate`。需要手动传递。

**「`sleep_for(0)` 等于 `yield()`」** —— 不完全等价。`sleep_for(0)` 可能触发系统调用，`yield()` 只是建议调度。

**「`hardware_concurrency()` 返回的就是最佳线程数」** —— 它只是硬件支持的并发数，实际最优值取决于任务类型（计算密集还是 I/O 密集）。

**「`thread_local` 没有开销」** —— 有。访问 TLS 变量比普通变量慢，热路径上要谨慎。

**「线程越多越快」** —— 超过核心数后，上下文切换和缓存失效会拖慢程序。

## 七、相关章节

- [互斥量与锁](./Mutex.md)：保护共享数据
- [条件变量](./Condition_Variable.md)：线程间等待与通知
- [异步任务](./Async.md)：更高层的线程管理方式
- [并发模式与陷阱](./Patterns.md)：线程池与工程实践
- [C++ 内存模型](../Memory/Memory_Model.md)：线程间的可见性保证
