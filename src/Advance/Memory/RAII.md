# RAII 与资源管理

RAII 是 **Resource Acquisition Is Initialization** 的缩写，直译是「资源获取即初始化」。这个名字起得不太好——它听起来像是在讲初始化，实际上讲的是**析构**。

它的核心思想只有一句话：**把资源的生命周期绑定到对象的生命周期上**。资源在构造函数里获取，在析构函数里释放。由于栈对象的析构是编译器自动插入的，无论函数是正常返回、提前 return 还是抛异常，资源都会被正确释放。

这是 C++ 区别于大多数语言的核心特性，也是整个标准库的设计基石。

## 一、为什么需要 RAII

先看一段没有 RAII 的代码：

```c++
void process()
{
    FILE* f = fopen("data.txt", "r");
    if (!f) return;

    char* buffer = (char*)malloc(1024);
    if (!buffer) {
        fclose(f);          // 每个错误分支都要手动清理
        return;
    }

    if (parse(f, buffer) < 0) {
        free(buffer);       // 又一处
        fclose(f);
        return;
    }

    free(buffer);
    fclose(f);
}
```

问题很明显：

- 每个提前返回的分支都要重复写清理代码，漏一个就泄漏
- 如果 `parse` 抛出异常，两处清理都不会执行
- 清理顺序必须和获取顺序相反，容易写错

用 RAII 改写：

```c++
void process()
{
    std::ifstream file("data.txt");      // 构造时打开
    if (!file) return;                   // 自动关闭

    std::vector<char> buffer(1024);      // 构造时分配

    parse(file, buffer);                 // 抛异常也没关系
}   // 离开作用域，buffer 和 file 自动释放
```

清理代码消失了，因为编译器会在每个退出路径上插入析构调用。

## 二、RAII 的完整形态

一个 RAII 类通常包含四部分：

```c++
class FileHandle
{
public:
    // 1. 构造函数获取资源
    explicit FileHandle(const char* path)
        : file_(fopen(path, "r"))
    {
        if (!file_)
            throw std::runtime_error("打开文件失败");
    }

    // 2. 析构函数释放资源
    ~FileHandle()
    {
        if (file_)
            fclose(file_);
    }

    // 3. 禁止拷贝（否则会双重释放）
    FileHandle(const FileHandle&) = delete;
    FileHandle& operator=(const FileHandle&) = delete;

    // 4. 允许移动（转移所有权）
    FileHandle(FileHandle&& other) noexcept
        : file_(other.file_)
    {
        other.file_ = nullptr;
    }

    FileHandle& operator=(FileHandle&& other) noexcept
    {
        if (this != &other) {
            if (file_) fclose(file_);
            file_ = other.file_;
            other.file_ = nullptr;
        }
        return *this;
    }

    FILE* get() const { return file_; }

private:
    FILE* file_;
};
```

这四步是 RAII 类的标准模板：**获取、释放、禁拷贝、允许移动**。

## 三、RAII 的常见形态

RAII 不只用于内存，任何「获取后必须释放」的资源都适用。

### 1. 锁

```c++
std::mutex mtx;

void safe_increment()
{
    std::lock_guard<std::mutex> lock(mtx);   // 构造时加锁
    ++shared_counter;
}   // 析构时自动解锁，异常也安全
```

手写 `lock()` / `unlock()` 的话，中间任何异常都会导致死锁。

### 2. 文件与流

```c++
{
    std::ofstream out("log.txt");
    out << "写点东西\n";
}   // 自动关闭并 flush
```

### 3. 数据库连接、网络连接

```c++
class Connection
{
public:
    Connection(const std::string& url) : conn_(connect(url)) {}
    ~Connection() { disconnect(conn_); }
    // ...
};
```

### 4. 临时状态恢复

RAII 也可以用来「保证状态被还原」：

```c++
class ScopedFlag
{
public:
    ScopedFlag(bool& flag) : flag_(flag), old_(flag) { flag_ = true; }
    ~ScopedFlag() { flag_ = old_; }
private:
    bool& flag_;
    bool old_;
};

void process()
{
    ScopedFlag guard(is_processing);   // 进入时设为 true
    // ... 无论怎么退出，都会恢复原值
}
```

### 5. 性能计时

```c++
class Timer
{
public:
    Timer(const char* name)
        : name_(name), start_(std::chrono::steady_clock::now()) {}

    ~Timer()
    {
        auto end = std::chrono::steady_clock::now();
        auto ms = std::chrono::duration_cast<std::chrono::milliseconds>(end - start_);
        std::cout << name_ << " 耗时 " << ms.count() << " ms\n";
    }

private:
    const char* name_;
    std::chrono::steady_clock::time_point start_;
};

void work()
{
    Timer t("work");   // 离开作用域自动打印耗时
    // ...
}
```

## 四、RAII 与异常安全

RAII 之所以重要，是因为它让**异常安全**变得可行。

考虑这个函数：

```c++
void bad()
{
    Resource* r1 = acquire();
    Resource* r2 = acquire();   // 抛异常 → r1 泄漏

    use(r1, r2);

    release(r2);
    release(r1);
}
```

改成 RAII：

```c++
void good()
{
    auto r1 = acquire();        // 返回 RAII 对象
    auto r2 = acquire();        // 抛异常 → r1 自动释放

    use(r1, r2);
}
```

不需要写 `try` / `catch`，资源也不会泄漏。这就是为什么标准库的异常安全保证能够成立——它建立在 RAII 之上。

### 析构函数不能抛异常

栈展开过程中，如果某个析构函数又抛出异常，而此时已有异常在传播，程序会直接调用 `std::terminate`。

```c++
~Bad()
{
    throw std::runtime_error("oops");   // 危险
}
```

C++11 起析构函数默认是 `noexcept` 的，在其中抛异常会导致终止。如果析构中确实可能失败（比如 `fclose` 返回值），应该吞掉异常或记录下来，而不是往外抛。

## 五、RAII 与所有权

RAII 的核心是「谁负责释放」。这引出了**所有权**的概念：

| 所有权形式 | 含义 | 典型类型 |
| :--- | :--- | :--- |
| **独占** | 只有一个对象负责释放 | `std::unique_ptr`、`std::ifstream` |
| **共享** | 多个对象共同负责，最后一个释放 | `std::shared_ptr` |
| **不拥有** | 只使用，不负责释放 | 裸指针、引用、`std::string_view` |

设计接口时应该明确表达所有权：

```c++
// 明确：函数接管所有权
void take(std::unique_ptr<Widget> w);

// 明确：函数只借用，不负责释放
void use(const Widget& w);
void use(Widget* w);   // 也常见，但不如引用清晰

// 明确：函数共享所有权
void share(std::shared_ptr<Widget> w);
```

裸指针作为参数本身没有错，但要清楚它表示「不拥有」——调用者负责保证对象在调用期间存活。

## 六、Rule of Three / Five / Zero

RAII 类需要正确处理拷贝和移动，这引出了几个经验法则。

### Rule of Three（C++98）

如果类需要自定义**析构函数**、**拷贝构造函数**、**拷贝赋值运算符**中的任何一个，那么通常三个都需要。

原因：需要自定义析构，说明持有需要手动释放的资源；那么默认的拷贝行为（浅拷贝指针）会导致双重释放，必须一并处理。

### Rule of Five（C++11）

加上**移动构造函数**和**移动赋值运算符**，变成五个：

```c++
class Buffer
{
public:
    Buffer(const Buffer&);              // 拷贝构造
    Buffer& operator=(const Buffer&);   // 拷贝赋值
    Buffer(Buffer&&) noexcept;          // 移动构造
    Buffer& operator=(Buffer&&) noexcept;  // 移动赋值
    ~Buffer();                          // 析构
};
```

如果定义了拷贝操作但没定义移动操作，编译器不会自动生成移动版本——此时「移动」会退化为拷贝，损失性能。

### Rule of Zero

**最好的做法是一个都不写**。

用现成的 RAII 类型作为成员，编译器生成的默认操作就已经正确了：

```c++
class Widget
{
    std::string name_;                    // 自己管理内存
    std::vector<int> data_;               // 自己管理内存
    std::unique_ptr<Impl> impl_;          // 独占所有权
    std::shared_ptr<Config> config_;      // 共享所有权
};
// 不需要写析构、拷贝、移动——编译器生成的都对
```

这是现代 C++ 推荐的风格：**让成员类型去管理资源，自己只管组合**。

## 七、常见误区

**「RAII 就是智能指针」** —— 智能指针只是 RAII 的一种应用。锁、文件、连接、临时状态恢复都是 RAII。

**「RAII 会拖慢程序」** —— 析构调用是编译期确定的，和手写 `delete` 没有性能差异。反而因为减少了错误分支，代码往往更快。

**「析构函数里可以随便做事」** —— 析构函数不能抛异常，也不应该做可能失败的操作。复杂的清理逻辑应该提供显式的 `close()` 方法，析构只做兜底。

**「移动后源对象不能再用」** —— 移动后的对象处于「有效但未指定」状态，可以重新赋值或析构，但不能假设它的内容。

**「拷贝构造和移动构造可以只写一个」** —— 定义了拷贝构造后，移动构造不会自动生成，此时移动会退化为拷贝。反之亦然。

## 八、相关章节

- [动态内存分配](./Allocation.md)：为什么不该手写 `delete`
- [智能指针](./Smart_Pointer.md)：RAII 在内存管理上的标准实现
- [异常处理](../Exception.md)：栈展开与异常安全保证
- [拷贝控制](../Object/Copy_Control.md)：Rule of Three / Five / Zero 的细节
- [多线程与并发](../Concurrency.md)：`lock_guard` 等 RAII 锁
