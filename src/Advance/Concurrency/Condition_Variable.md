# 条件变量

条件变量解决的是「线程需要等待某个条件成立」的问题。

没有它的话，等待只能靠轮询：

```c++
// 轮询：浪费 CPU，且响应有延迟
while (!ready) {
    std::this_thread::sleep_for(1ms);
}
```

条件变量让线程真正睡眠，条件成立时被唤醒——既不浪费 CPU，响应也及时。

## 一、基本用法

条件变量必须配合互斥量使用。三者缺一不可：**互斥量保护条件变量、条件变量负责睡眠、谓词判断条件**。

```c++
#include <condition_variable>
#include <mutex>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> q;

// 生产者
void producer()
{
    {
        std::lock_guard<std::mutex> lock(mtx);
        q.push(42);
    }                       // 先解锁
    cv.notify_one();        // 再通知
}

// 消费者
void consumer()
{
    std::unique_lock<std::mutex> lock(mtx);

    // 等待队列非空
    cv.wait(lock, [] { return !q.empty(); });

    int value = q.front();
    q.pop();
}
```

### 1. wait 做了什么

`cv.wait(lock, pred)` 等价于：

```c++
while (!pred())          // 1. 检查条件
{
    cv.wait(lock);       // 2. 原子地解锁并睡眠
}                        // 3. 被唤醒后重新加锁，回到 1
```

第 2 步的「原子地解锁并睡眠」是关键——如果解锁和睡眠不是原子的，通知可能在这两步之间发出，导致线程永远睡下去（丢失唤醒）。

### 2. 为什么必须用谓词版本

条件变量可能发生**虚假唤醒**（spurious wakeup）——即使没有 `notify`，`wait` 也可能返回。这不是 bug，而是实现允许的行为（源于底层系统调用的特性）。

```c++
// 错误：可能被虚假唤醒，此时条件并不成立
cv.wait(lock);
if (q.empty()) { /* 逻辑错误 */ }

// 正确：带谓词的版本内部是循环
cv.wait(lock, [] { return !q.empty(); });
```

**永远使用带谓词的版本**。手写 `while` 循环也可以，但谓词版本更简洁，也不容易写错。

### 3. notify_one 与 notify_all

| 方法 | 行为 |
| :--- | :--- |
| `notify_one()` | 唤醒一个等待的线程 |
| `notify_all()` | 唤醒所有等待的线程 |

**什么时候用 `notify_one`**：多个线程竞争同一份资源，唤醒一个就够（比如队列里加了一个元素）。

**什么时候用 `notify_all`**：

- 条件变化影响所有等待者（比如「关闭」标志置位）
- 不确定哪个线程能推进（唤醒的那个可能条件不满足，又睡回去）

```c++
std::atomic<bool> shutdown{false};

void shutdownAll()
{
    {
        std::lock_guard lock(mtx);
        shutdown = true;
    }
    cv.notify_all();   // 所有等待者都要退出
}
```

## 二、生产者消费者

这是条件变量的经典场景。一个完整的、有界队列实现：

```c++
#include <condition_variable>
#include <mutex>
#include <queue>

template<class T>
class BoundedQueue
{
public:
    explicit BoundedQueue(std::size_t capacity) : capacity_(capacity) {}

    void push(T value)
    {
        std::unique_lock<std::mutex> lock(mtx_);

        // 队列满则等待，直到有空间
        notFull_.wait(lock, [this] { return q_.size() < capacity_ || closed_; });
        if (closed_) return;

        q_.push(std::move(value));
        lock.unlock();       // 先解锁再通知，减少等待者被立即阻塞的概率

        notEmpty_.notify_one();
    }

    bool pop(T& value)
    {
        std::unique_lock<std::mutex> lock(mtx_);

        // 队列空则等待，直到有元素或已关闭
        notEmpty_.wait(lock, [this] { return !q_.empty() || closed_; });
        if (q_.empty()) return false;   // 已关闭且为空

        value = std::move(q_.front());
        q_.pop();
        lock.unlock();

        notFull_.notify_one();
        return true;
    }

    void close()
    {
        {
            std::lock_guard<std::mutex> lock(mtx_);
            closed_ = true;
        }
        notEmpty_.notify_all();
        notFull_.notify_all();
    }

private:
    std::mutex mtx_;
    std::condition_variable notEmpty_;
    std::condition_variable notFull_;
    std::queue<T> q_;
    std::size_t capacity_;
    bool closed_ = false;
};
```

几个设计要点：

**两个条件变量**。等待「非空」和等待「非满」是两个不同的条件，用同一个条件变量会导致 `notify_all` 唤醒一堆无关线程。

**`closed_` 标志**。让等待者能优雅退出，否则 `close()` 之后消费者会永远等待。

**先解锁再通知**。被唤醒的线程会立即尝试加锁，如果通知时还持有锁，它会先阻塞一次。解锁后通知能减少这次无谓的阻塞。

## 三、condition_variable_any

`std::condition_variable` 只能配合 `std::unique_lock<std::mutex>`。需要配合其它锁类型时用 `condition_variable_any`：

```c++
#include <condition_variable>
#include <shared_mutex>

std::shared_mutex rwmtx;
std::condition_variable_any cv;

void reader()
{
    std::shared_lock lock(rwmtx);
    cv.wait(lock, [] { return /* 条件 */; });
}
```

**代价**：`condition_variable_any` 的实现更复杂（需要类型擦除），性能不如 `condition_variable`。能用前者就用前者。

### 配合 stop_token（C++20）

`condition_variable_any` 支持 `stop_token`，可以在收到停止请求时自动唤醒：

```c++
void worker(std::stop_token st)
{
    std::unique_lock lock(mtx);
    // 停止请求发出时自动唤醒
    cv.wait(lock, st, [] { return /* 条件 */; });
}
```

这解决了「线程卡在 `wait` 里，`request_stop()` 无法生效」的问题。

## 四、常见陷阱

### 1. 丢失唤醒

**通知发生在等待之前**：

```c++
// 线程 A（消费者）
// 还没开始 wait

// 线程 B（生产者）
{
    std::lock_guard lock(mtx);
    q.push(42);
}
cv.notify_one();   // 此时没有等待者，通知丢失

// 线程 A
cv.wait(lock);     // 永远等待
```

**解决办法**：用谓词版本。`wait(lock, pred)` 会先检查谓词，即使通知已经丢失，条件已成立就不会等待。

```c++
cv.wait(lock, [] { return !q.empty(); });   // 安全
```

### 2. 持有锁时通知

```c++
// 不好
{
    std::lock_guard lock(mtx);
    q.push(42);
    cv.notify_one();   // 持有锁时通知
}
```

被唤醒的线程会立即尝试加锁，但锁还被持有，于是它又阻塞一次。虽然正确，但多了一次上下文切换。

```c++
// 好
{
    std::lock_guard lock(mtx);
    q.push(42);
}                      // 先解锁
cv.notify_one();       // 再通知
```

### 3. 用同一个条件变量等待不同条件

```c++
// 不好：两个不同的条件共用一个 cv
cv.wait(lock, [] { return !q.empty(); });      // 消费者
cv.wait(lock, [] { return q.size() < 10; });   // 生产者
```

`notify_all()` 会唤醒所有线程，其中大部分会发现条件不满足又睡回去。应该用两个独立的条件变量。

### 4. 忘记在循环外修改共享状态

```c++
// 错误：条件永远不成立
cv.wait(lock, [] { return ready; });
// ready 从来没有被设为 true
```

谓词检查的是共享状态，修改共享状态时**必须在持锁状态下进行**，否则可能产生数据竞争。

## 五、性能考量

条件变量的开销主要来自：

- **唤醒延迟**：从 `notify` 到被唤醒线程真正运行，通常几微秒
- **上下文切换**：涉及内核调度
- **惊群效应**：`notify_all` 唤醒多个线程，但只有一个能推进

优化建议：

**优先用 `notify_one`**。只有确实需要唤醒所有线程时才用 `notify_all`。

**减少通知频率**。批量处理时，不要每加一个元素就通知一次：

```c++
// 不好：每个元素通知一次
for (auto& item : items) {
    {
        std::lock_guard lock(mtx);
        q.push(item);
    }
    cv.notify_one();
}

// 好：批量加入后通知一次
{
    std::lock_guard lock(mtx);
    for (auto& item : items)
        q.push(item);
}
cv.notify_all();
```

**考虑无锁队列**。如果条件变量的开销成为瓶颈，可以用无锁队列配合原子等待（C++20 的 `atomic::wait` / `notify_one`）。

## 六、常见误区

**「`cv.wait(lock)` 不带谓词也可以」** —— 虚假唤醒会导致逻辑错误。永远用谓词版本。

**「`notify_one` 会唤醒特定的线程」** —— 不会。它唤醒任意一个等待者，无法指定。

**「通知必须在持锁时发出」** —— 不需要，而且通常应该先解锁再通知。

**「条件变量的谓词会在持锁状态下检查」** —— 是的，`wait` 返回时锁是持有的，谓词检查也在持锁状态下进行。

**「`condition_variable_any` 和 `condition_variable` 一样快」** —— 前者有类型擦除开销，能用后者就用后者。

**「等待超时可以用 `sleep` 模拟」** —— `sleep` 无法被通知打断，响应延迟大。用 `wait_for` / `wait_until`。

## 七、相关章节

- [互斥量与锁](./Mutex.md)：`unique_lock` 与条件变量的配合
- [线程基础](./Thread.md)：`jthread` 与 `stop_token`
- [异步任务](./Async.md)：`future` 内部的等待机制
- [并发模式与陷阱](./Patterns.md)：生产者消费者与其它模式
