# 异步任务

前面几节讲的都是「多个线程同时干活」。这一节换个角度：**把一件事交给别处去做，自己继续往下走，需要结果时再取回来**。

这就是异步。它和「多线程」不是一回事——异步是关于**任务的组织方式**，多线程只是实现它的手段之一。一个异步任务可能跑在另一个线程上，也可能跑在线程池里，甚至可能根本不并发（延迟执行）。

## 一、三种异步风格

「异步」不是一个具体的 API，而是一类**代码组织方式**。按「怎么拿到结果」来分，主流有三种风格：

| 风格 | 怎么拿结果 | 典型代表 |
| :--- | :--- | :--- |
| **回调式** | 传一个函数进去，完成时被调用 | `readAsync(cb)`、`epoll`、Node.js |
| **Future/Promise 式** | 拿到一个「未来会有值」的凭证，`get()` 取回 | `std::future`、`std::async` |
| **协程式** | 用 `co_await` 挂起，看起来像同步代码 | C++20 协程、`std::generator` |

这三种风格解决的是同一件事，但**控制流的写法完全不同**。下面先讲标准库提供的 Future/Promise 风格（这是 C++ 里最常用的），再讲回调，最后讲协程。

### 1. std::async：最省事的写法

```c++
#include <future>

int compute(int x)
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return x * x;
}

int main()
{
    std::future<int> fut = std::async(std::launch::async, compute, 10);

    // 主线程继续做别的事
    std::cout << "doing other work\n";

    int result = fut.get();   // 需要结果时取回，会阻塞直到完成
    std::cout << result << '\n';   // 100
}
```

`std::async` 返回一个 `std::future<T>`，`get()` 会阻塞直到结果就绪。如果任务抛了异常，`get()` 会重新抛出。

**启动策略**：

| 策略 | 行为 |
| :--- | :--- |
| `std::launch::async` | 立即在新线程上启动 |
| `std::launch::deferred` | 延迟到 `get()` / `wait()` 时才在当前线程执行 |
| 默认（两者都不指定） | 由实现选择 |

**默认策略是个陷阱**。如果实现选择了 `deferred`，任务根本不会异步执行——`get()` 时才同步跑一遍。这会悄悄破坏你的并发假设：

```c++
// 危险：可能根本没并发
auto f1 = std::async(task1);
auto f2 = std::async(task2);
f1.get();
f2.get();
```

如果两个任务都是 `deferred`，它们会串行执行。**要并发就显式写 `std::launch::async`**。

### 2. std::promise / std::future：手动传递结果

`std::promise` 是「结果的写端」，`std::future` 是「结果的读端」。它们通过一个共享状态连接。

```c++
#include <future>
#include <thread>

void worker(std::promise<int> prom)
{
    try
    {
        int result = doSomething();
        prom.set_value(result);        // 设置结果
    }
    catch (...)
    {
        prom.set_exception(std::current_exception());   // 传递异常
    }
}

int main()
{
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();

    std::thread t(worker, std::move(prom));

    int result = fut.get();   // 等待并取回
    t.join();
}
```

**promise 必须被移动**（不可拷贝），因为结果只能被设置一次。

适合的场景：任务的结果不是由「函数返回值」自然产生的，比如回调式 API 里在回调中设置结果：

```c++
std::future<int> asyncRead(Connection& conn)
{
    auto prom = std::make_shared<std::promise<int>>();
    auto fut = prom->get_future();

    conn.readAsync([prom](int value) {
        prom->set_value(value);   // 回调里设置结果
    });

    return fut;
}
```

### 3. std::packaged_task：把任务包装成可调用对象

`std::packaged_task` 把「一个可调用对象 + 一个 future」绑在一起，任务执行时自动把返回值（或异常）送进 future。

```c++
#include <future>

std::packaged_task<int(int)> task([](int x) {
    return x * x;
});

std::future<int> fut = task.get_future();

// 可以在任意线程、任意时机执行
std::thread t(std::move(task), 10);
t.join();

std::cout << fut.get();   // 100
```

它的价值在于**任务可以被存储、传递、排队**——这正是线程池需要的：

```c++
class ThreadPool
{
    std::queue<std::packaged_task<void()>> tasks_;
    // ...

public:
    template<class F>
    auto submit(F&& f) -> std::future<decltype(f())>
    {
        using ReturnType = decltype(f());

        std::packaged_task<ReturnType()> task(std::forward<F>(f));
        std::future<ReturnType> fut = task.get_future();

        {
            std::lock_guard lock(mtx_);
            tasks_.emplace(std::move(task));
        }
        cv_.notify_one();

        return fut;   // 调用者拿到 future，稍后取结果
    }
};
```

### 4. 三种 Future 工具对比

`std::async`、`std::promise`、`std::packaged_task` 都属于 Future/Promise 风格，区别在于「结果是怎么产生的」：

| 方式 | 抽象层次 | 结果从哪来 | 适用场景 |
| :--- | :--- | :--- | :--- |
| `std::async` | 最高 | 函数返回值，库自动处理 | 一次性任务，不关心线程管理 |
| `std::promise` | 最低 | 手动 `set_value` | 结果来自回调、需要手动控制时机 |
| `std::packaged_task` | 中等 | 执行时自动填入 | 任务需要排队、存储、传递 |

三者最终都产生 `std::future`，取结果的方式一致。选哪个取决于**结果是怎么产生的**。

> 注意区分两个维度：这里是「Future/Promise 风格内部的三种工具」，而本节开头讲的是「回调 / Future / 协程三种风格」。前者是同一风格下的不同工具，后者是根本不同的代码组织方式。

## 二、future 的细节

### 1. shared_future

`std::future` 的 `get()` 只能调用一次，且会移动结果。多个线程需要读同一结果时用 `std::shared_future`：

```c++
std::promise<int> prom;
std::shared_future<int> sfut = prom.get_future().share();

auto reader = [sfut] {
    std::cout << sfut.get() << '\n';   // 可以多次调用
};

std::thread t1(reader), t2(reader);
prom.set_value(42);
t1.join();
t2.join();
```

### 2. 超时等待

```c++
std::future<int> fut = std::async(std::launch::async, slowTask);

// 等待最多 100ms
if (fut.wait_for(std::chrono::milliseconds(100)) == std::future_status::ready)
{
    std::cout << fut.get() << '\n';
}
else
{
    std::cout << "还没完成\n";
}
```

三种状态：

| 状态 | 含义 |
| :--- | :--- |
| `std::future_status::ready` | 结果就绪 |
| `std::future_status::timeout` | 超时 |
| `std::future_status::deferred` | 任务是延迟执行的 |

### 3. future 的局限

标准库的 `future` 有几个明显的短板：

**没有 `then()`**。无法链式组合：

```c++
// 标准库做不到这样
fut.then([](int x) { return x * 2; })
   .then([](int x) { std::cout << x; });
```

C++ 标准委员会曾推动 `std::future` 的扩展（`.then()`、`when_all`、`when_any`），但最终没有进入标准。需要这些功能得用第三方库（如 folly::Future、boost::future）。

**析构可能阻塞**。`std::async` 返回的 future，如果从未调用 `get()` 或 `wait()`，析构时会**同步等待任务完成**：

```c++
{
    auto fut = std::async(std::launch::async, longTask);
    // 忘记 get()
}   // 这里会阻塞，直到 longTask 完成
```

这常常出乎意料。要么确保调用 `get()`，要么用线程池自己管理。

**无法取消**。`future` 没有取消机制。任务一旦启动就必须跑完。

## 三、回调函数

回调是另一种组织异步的方式：不返回 future，而是**传入一个函数，任务完成时调用它**。

### 1. 基本形式

```c++
void readAsync(const std::string& path, std::function<void(std::string)> callback)
{
    std::thread([path, callback] {
        std::string content = readFile(path);
        callback(content);   // 完成后调用
    }).detach();
}

// 使用
readAsync("data.txt", [](std::string content) {
    std::cout << "读到 " << content.size() << " 字节\n";
});
```

回调比 future 更灵活——可以多次调用（流式数据）、可以在任意时机调用。但代价是**控制流被打散**。

### 2. 生命周期问题

回调最大的坑是**捕获对象的生命周期**。如果回调持有 `this`，而对象在回调触发前被销毁了：

```c++
class Downloader
{
public:
    void start()
    {
        // 危险：this 可能在回调前失效
        asyncFetch([this](Data d) {
            this->onData(d);   // 悬空指针
        });
    }

    void onData(Data d) { /* ... */ }
};

// 使用
{
    Downloader d;
    d.start();
}   // d 析构，但异步任务还在跑
```

**解决办法一：`enable_shared_from_this`**

```c++
class Downloader : public std::enable_shared_from_this<Downloader>
{
public:
    void start()
    {
        auto self = shared_from_this();   // 延长生命周期
        asyncFetch([self](Data d) {
            self->onData(d);   // 只要回调存在，对象就活着
        });
    }
};
```

**解决办法二：弱引用 + 检查**

```c++
class Downloader : public std::enable_shared_from_this<Downloader>
{
public:
    void start()
    {
        std::weak_ptr<Downloader> weak = shared_from_this();
        asyncFetch([weak](Data d) {
            if (auto self = weak.lock())   // 对象还活着吗？
                self->onData(d);
        });
    }
};
```

弱引用不会延长生命周期，适合「对象没了就放弃回调」的语义。

**解决办法三：取消机制**

让异步操作支持取消，对象析构时先取消：

```c++
class Downloader
{
    std::stop_source stopSrc_;
public:
    ~Downloader() { stopSrc_.request_stop(); }

    void start()
    {
        auto token = stopSrc_.get_token();
        asyncFetch([this, token](Data d) {
            if (token.stop_requested()) return;
            onData(d);
        });
    }
};
```

### 3. 回调地狱

回调嵌套多了会变成这样：

```c++
login(user, [](Result r1) {
    fetchProfile(r1.id, [](Profile p) {
        loadSettings(p.settingsId, [](Settings s) {
            applyTheme(s.theme, [](bool ok) {
                // 越来越深……
            });
        });
    });
});
```

这就是「回调地狱」。协程的动机之一就是解决它。

### 4. 回调与 future 的桥接

两种风格可以互相转换。前面见过回调 → future（用 `promise`）。反过来，future → 回调：

```c++
template<class T, class F>
void then(std::future<T> fut, F callback)
{
    std::thread([fut = std::move(fut), callback]() mutable {
        callback(fut.get());
    }).detach();
}

// 使用
then(std::async(std::launch::async, compute, 10), [](int x) {
    std::cout << x << '\n';
});
```

### 5. 异步回调里的 TOCTOU

异步代码天然存在「检查」和「使用」之间的时间差，这正是 TOCTOU（检查时刻到使用时刻）的温床。

看这个例子——检查时对象还活着，回调触发时已经没了：

```c++
class Connection
{
public:
    void send(const std::string& data)
    {
        if (!connected_)          // 检查：连接还在吗
            return;

        // ... 中间可能有 await / 回调 / 线程切换 ...

        socket_.write(data);      // 使用：连接可能已经断开！
    }
};
```

**在异步代码里，这个窗口可能非常长**——一次网络往返、一次磁盘 I/O，都是毫秒级甚至秒级。相比之下，多线程里的 TOCTOU 窗口通常只有几条指令。

回调场景的典型形态：

```c++
// 危险：回调触发时，检查的结论可能已经过期
void onData(Data d)
{
    if (buffer_.size() < MAX)     // 检查
    {
        process(d);                // 使用 —— 但 buffer_ 可能已被其它回调填满
        buffer_.push_back(d);
    }
}
```

**解决办法和同步代码一样：把检查和使用合并成一次原子操作。**

```c++
void onData(Data d)
{
    std::lock_guard lock(mtx_);
    if (buffer_.size() >= MAX)    // 检查和使用在同一临界区
        return;
    buffer_.push_back(d);
    process(d);
}
```

或者用「一次操作 + 错误处理」的 EAFP 风格：

```c++
void onData(Data d)
{
    if (!tryPush(d))              // 一次操作，失败就放弃
        return;
    process(d);
}
```

**异步代码里还要特别注意**：`co_await`、回调、`future.get()` 都是潜在的「检查点」。任何跨越这些点的「先判断、后操作」都需要重新审视。详见[互斥量与锁](./Mutex.md)中的 TOCTOU 一节。

## 四、协程

协程（coroutine）是 C++20 引入的语言特性。它让函数可以**暂停和恢复**，而不阻塞线程。

### 1. 三个关键字

```c++
#include <coroutine>

// co_await：等待一个异步操作完成
Task fetchAsync()
{
    auto data = co_await fetch("url");   // 暂停，让出线程
    co_return process(data);             // 返回结果
}

// co_yield：产生一个值并暂停
Generator<int> range(int n)
{
    for (int i = 0; i < n; ++i)
        co_yield i;      // 每次产生一个值
}

// co_return：结束协程并返回
```

只要函数体里出现这三个关键字之一，它就是协程。

### 2. 为什么协程能解决回调地狱

用协程重写前面的嵌套回调：

```c++
Task applyThemeForUser(User user)
{
    Result r1 = co_await login(user);
    Profile p = co_await fetchProfile(r1.id);
    Settings s = co_await loadSettings(p.settingsId);
    bool ok = co_await applyTheme(s.theme);
}
```

**看起来是同步代码，实际是异步执行**。每个 `co_await` 会暂停当前协程、把控制权还给调用者，等操作完成后再恢复。没有嵌套，没有回调地狱。

### 3. 标准库只给了机制

这是最容易误解的地方：**C++20 的协程是纯粹的编译器机制，标准库几乎没有提供可用的协程类型**。

编译器负责把协程函数转换成状态机，但「`Task` 是什么」「`Generator` 怎么实现」「`co_await` 一个网络请求意味着什么」——这些都要你自己定义，或者用库。

标准库唯一提供的协程类型是 C++23 的 `std::generator`：

```c++
#include <generator>

std::generator<int> fibonacci()
{
    int a = 0, b = 1;
    while (true)
    {
        co_yield a;
        std::tie(a, b) = std::pair{b, a + b};
    }
}

int main()
{
    for (int x : fibonacci())
    {
        if (x > 100) break;
        std::cout << x << ' ';
    }
}
```

### 4. 自己实现需要什么

一个最小的协程类型需要定义 `promise_type`：

```c++
struct Task
{
    struct promise_type
    {
        Task get_return_object() { return {}; }
        std::suspend_never initial_suspend() { return {}; }
        std::suspend_never final_suspend() noexcept { return {}; }
        void return_void() {}
        void unhandled_exception() { std::terminate(); }
    };
};
```

`promise_type` 是编译器与协程交互的接口——它决定协程何时挂起、结果如何传递、异常如何处理。

真正可用的协程库（如 cppcoro、libunifex）需要实现调度、内存分配、`co_await` 的 awaiter 协议等，工作量很大。

### 5. 协程与线程的关系

**协程不是线程**。协程在哪个线程上恢复，取决于调度器。同一个线程可以跑成千上万个协程，因为挂起时不占用栈。

这让协程非常适合**高并发 I/O 密集**场景：等待网络响应时挂起，线程去跑别的协程，而不是阻塞在 `read()` 上。

但协程**不解决 CPU 密集问题**——计算任务仍然需要多线程。

## 五、选择哪种方式

先选风格，再选工具：

| 风格 | 适合什么情况 | 不适合什么情况 |
| :--- | :--- | :--- |
| **回调** | 事件驱动、流式数据、需要多次通知 | 复杂的顺序流程（会变成回调地狱） |
| **Future** | 一次性任务、需要取一次结果 | 需要链式组合、需要取消 |
| **协程** | 复杂的异步流程、大量并发 I/O | 简单场景（引入的复杂度不划算） |

具体到工具：

| 需求 | 推荐 |
| :--- | :--- |
| 一次性并行计算 | `std::async` + `launch::async` |
| 结果来自回调式 API | `std::promise` |
| 任务需要排队 | `std::packaged_task` + 线程池 |
| 简单的完成通知 | 回调函数 |
| 复杂的异步流程 | 协程（配合库） |
| 需要 `.then()` 链式组合 | 第三方 future 库 |

## 六、常见误区

**「`std::async` 一定异步执行」** —— 默认策略可能是 `deferred`，任务会延迟到 `get()` 时同步执行。要并发就显式指定 `launch::async`。

**「`std::async` 返回的 future 析构不会阻塞」** —— 会。如果没调用 `get()`，析构时会等待任务完成。

**「`future::get()` 可以调用多次」** —— 只能一次。需要多次读用 `shared_future`。

**「回调比 future 好」** —— 各有场景。回调灵活但容易写出回调地狱和生命周期 bug；future 简单但缺少组合能力。

**「协程是并发的」** —— 协程是**并发模型**，不是并行。它在单线程上也能跑，靠挂起切换而非线程切换。

**「C++20 有协程就能直接写异步代码了」** —— 标准库只提供了语言机制，可用的协程类型要自己写或用库。

**「`co_await` 会阻塞线程」** —— 不会。它挂起协程、让出线程，这正是协程的价值。

**「异步代码里检查过就没问题了」** —— 检查和使用之间隔着 `co_await`、回调或 I/O，状态可能已经改变，这就是 TOCTOU。

## 七、相关章节

- [线程基础](./Thread.md)：异步任务背后的线程
- [互斥量与锁](./Mutex.md)：TOCTOU 与加锁的正确姿势
- [条件变量](./Condition_Variable.md)：future 内部的等待机制
- [并发模式与陷阱](./Patterns.md)：线程池与任务队列
- [现代 C++ 特性](../Modern_Cpp.md)：协程的语言机制
