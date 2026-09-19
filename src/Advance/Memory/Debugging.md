# 内存调试与诊断

内存错误的特点是**症状和原因往往离得很远**。程序可能在一个完全无关的地方崩溃，或者运行几小时后才慢慢泄漏光内存。这一章讲几类常见错误的成因，以及怎么把它们揪出来。

## 一、常见内存错误

### 1. 内存泄漏

分配了内存却忘记释放：

```c++
void leak()
{
    int* p = new int(42);
    // 忘记 delete
}
```

后果是进程内存持续增长。短命程序可能无所谓，但服务端程序会慢慢吃光内存。

**隐蔽的泄漏**往往来自异常路径：

```c++
void subtleLeak()
{
    auto* a = new Resource();
    auto* b = new Resource();   // 如果这里抛异常，a 泄漏了
    // ...
    delete b;
    delete a;
}
```

还有一种是**逻辑泄漏**——内存还被引用着，但已经没用了：

```c++
std::vector<Connection> connections;

void cleanup()
{
    connections.clear();   // 如果 Connection 持有裸指针，指针指向的资源泄漏了
}
```

### 2. 悬垂指针

指针指向的对象已经销毁：

```c++
int* dangling()
{
    int local = 42;
    return &local;   // 栈帧已回收
}

int* p = dangling();
std::cout << *p;     // 未定义行为
```

堆上的版本更隐蔽：

```c++
int* p = new int(42);
int* q = p;
delete p;
std::cout << *q;   // q 悬垂
```

释放后内存可能被重新分配给别的对象，读到的值无法预料。这类 bug 在测试环境常常「看起来正常」，上线后才爆发。

### 3. 缓冲区越界

读写超出分配范围：

```c++
int* arr = new int[10];
arr[10] = 1;        // 越界写，破坏了相邻内存

char buf[8];
strcpy(buf, "这是一个很长的字符串");   // 栈溢出
```

越界写会破坏相邻对象的元数据或内容，症状可能在很久之后才出现。

`std::vector` 的 `operator[]` 不做检查，`at()` 会检查并抛异常：

```c++
std::vector<int> v(10);
v[10] = 1;      // 未定义行为
v.at(10) = 1;   // 抛 std::out_of_range
```

### 4. 重复释放

```c++
int* p = new int(42);
delete p;
delete p;   // 未定义行为
```

如果两个指针指向同一块内存，各自都以为该由自己释放：

```c++
void bad(std::unique_ptr<int> a, int* raw)
{
    // raw 是 a.get()，函数结束时 a 释放了内存
}
// 调用者又 delete raw → 重复释放
```

### 5. 释放方式不匹配

```c++
int* p1 = new int[10];
delete p1;          // 应该用 delete[]

int* p2 = (int*)malloc(sizeof(int));
delete p2;          // 应该用 free

int* p3 = new int(42);
free(p3);           // 应该用 delete
```

### 6. 未初始化内存

```c++
int* p = new int;      // 未初始化
std::cout << *p;       // 读到垃圾值

int arr[10];
std::cout << arr[0];   // 同上
```

用 `new int()` 或 `new int{}` 可以值初始化：

```c++
int* p = new int();    // 初始化为 0
int* q = new int{};    // 同上
```

## 二、检测工具

### 1. AddressSanitizer（ASan）

最推荐的入门工具，GCC 和 Clang 都内置：

```bash
g++ -fsanitize=address -g main.cpp -o main
./main
```

它能检测：

- 堆缓冲区越界
- 栈缓冲区越界
- 释放后使用（use-after-free）
- 重复释放
- 内存泄漏（配合 `-fsanitize=leak`）

输出示例：

```
==12345==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x602000000018
WRITE of size 4 at 0x602000000018 thread T0
    #0 0x400abc in main main.cpp:5
```

ASan 会给出**精确的出错位置和调用栈**，代价是程序变慢 2~3 倍、内存占用增加。适合在测试环境常开。

### 2. UndefinedBehaviorSanitizer（UBSan）

检测未定义行为：

```bash
g++ -fsanitize=undefined -g main.cpp -o main
```

能捕获整数溢出、空指针解引用、越界移位等。可以和 ASan 一起用：

```bash
g++ -fsanitize=address,undefined -g main.cpp -o main
```

### 3. Valgrind

老牌工具，不需要重新编译：

```bash
valgrind --leak-check=full ./main
```

输出：

```
==12345== HEAP SUMMARY:
==12345==     in use at exit: 4 bytes in 1 blocks
==12345==   total heap usage: 1 allocs, 0 frees, 4 bytes allocated
==12345==
==12345== 4 bytes in 1 blocks are definitely lost in loss record 1 of 1
==12345==    at 0x4C2A2BB: operator new(unsigned long) (vg_replace_malloc.c:334)
==12345==    by 0x400ABC: main (main.cpp:4)
```

优点是能检测「可能泄漏」（still reachable）等 ASan 不报的情况，缺点是慢 10~50 倍。

### 4. 工具对比

| 工具 | 速度 | 检测能力 | 需要重编译 |
| :--- | :--- | :--- | :--- |
| **ASan** | 慢 2~3 倍 | 越界、UAF、重复释放、泄漏 | 是 |
| **UBSan** | 慢约 20% | 未定义行为 | 是 |
| **Valgrind** | 慢 10~50 倍 | 全面，含未初始化读取 | 否 |
| **TSan** | 慢 5~10 倍 | 数据竞争 | 是 |

**建议**：日常开发用 ASan + UBSan，CI 里跑一遍 Valgrind 兜底，多线程代码用 TSan。

### 5. ThreadSanitizer（TSan）

专门检测数据竞争：

```bash
g++ -fsanitize=thread -g main.cpp -o main -pthread
```

输出会指出竞争的两个位置：

```
WARNING: ThreadSanitizer: data race
  Write of size 4 at 0x7b0400000000 by thread T1:
    #0 increment() main.cpp:8
  Previous read of size 4 at 0x7b0400000000 by main thread:
    #0 main main.cpp:15
```

注意 TSan 和 ASan **不能同时使用**，需要分开跑。

## 三、静态分析

编译器的警告是第一道防线，务必打开：

```bash
g++ -Wall -Wextra -Wpedantic -Wshadow -Wconversion main.cpp
```

Clang 的静态分析器能发现更深的问题：

```bash
clang++ --analyze main.cpp
```

它能识别出「分配后未释放」「空指针解引用」等模式，不需要运行程序。

## 四、自己写分配器钩子

重载全局 `operator new` / `operator delete` 可以记录所有分配：

```c++
#include <cstdlib>
#include <iostream>
#include <map>
#include <mutex>

namespace {
std::mutex g_mutex;
std::map<void*, std::size_t> g_allocations;
}

void* operator new(std::size_t size)
{
    void* p = std::malloc(size);
    if (!p) throw std::bad_alloc();

    std::lock_guard<std::mutex> lock(g_mutex);
    g_allocations[p] = size;
    return p;
}

void operator delete(void* p) noexcept
{
    if (!p) return;

    std::lock_guard<std::mutex> lock(g_mutex);
    g_allocations.erase(p);
    std::free(p);
}

// 程序退出时打印未释放的分配
struct Reporter
{
    ~Reporter()
    {
        if (!g_allocations.empty())
        {
            std::cerr << "泄漏 " << g_allocations.size() << " 处分配:\n";
            for (const auto& [p, size] : g_allocations)
                std::cerr << "  " << p << " (" << size << " 字节)\n";
        }
    }
};
Reporter g_reporter;
```

这个简易追踪器能告诉你**有没有泄漏、泄漏了多少处**，但拿不到调用栈。生产级的实现（如 ASan）会记录栈回溯。

## 五、预防措施

工具能帮你抓 bug，但更好的做法是**让 bug 写不出来**。

### 1. 用智能指针

```c++
// 容易出错
Widget* w = new Widget();
// ...
delete w;

// 不可能忘记释放
auto w = std::make_unique<Widget>();
```

### 2. 用容器代替裸数组

```c++
// 容易越界
int* arr = new int[n];
arr[i] = 1;
delete[] arr;

// 有边界检查（at）、自动管理
std::vector<int> arr(n);
arr.at(i) = 1;
```

### 3. 用 string 代替 char*

```c++
// 容易溢出
char buf[100];
strcpy(buf, input.c_str());

// 自动扩容
std::string buf = input;
```

### 4. 遵循 Rule of Zero

让成员类型管理资源，自己不写析构和拷贝：

```c++
class Widget
{
    std::string name_;
    std::vector<int> data_;
    std::unique_ptr<Impl> impl_;
    // 不需要写析构、拷贝、移动
};
```

### 5. 编译期检查

```c++
// 数组大小用常量表达式
constexpr std::size_t N = 10;
std::array<int, N> arr;

// 用 span 传递数组，避免退化成指针丢失大小
void process(std::span<int> data);
```

## 六、排查思路

遇到内存问题时，按这个顺序排查：

1. **打开所有警告**，先看编译器报什么
2. **跑 ASan**，多数越界和 UAF 会立即暴露
3. **跑 Valgrind**，看有没有泄漏和未初始化读取
4. **如果是并发问题**，跑 TSan
5. **如果还找不到**，用 `operator new` 钩子记录分配，缩小范围
6. **最后手段**：二分注释代码，定位问题区间

一个实用技巧：**崩溃位置往往不是出错位置**。如果 ASan 报告「堆缓冲区溢出」，重点看那块内存是谁分配的、谁写的，而不是崩溃的那一行。

## 七、常见误区

**「测试通过就没内存问题」** —— 内存错误高度依赖具体的内存布局和执行时序，测试环境跑通不代表生产环境安全。

**「用 `delete` 后置空就没问题了」** —— 置空只防止重复释放，防止不了其他指针的悬垂。

**「ASan 太慢不能常开」** —— 在 CI 和本地测试环境开启完全可行，代价远小于线上排查一个内存 bug。

**「Valgrind 能替代 ASan」** —— 两者互补。Valgrind 不需要重编译、能检测未初始化读取；ASan 更快、栈信息更准。

**「内存泄漏只影响内存」** —— 泄漏的往往不只是内存。文件描述符、socket、锁、数据库连接泄漏同样致命。

**「智能指针能解决所有问题」** —— 智能指针解决的是「忘记释放」，解决不了循环引用、逻辑泄漏、越界访问。

## 八、相关章节

- [动态内存分配](./Allocation.md)：`new` / `delete` 的常见错误
- [RAII 与资源管理](./RAII.md)：从设计上避免泄漏
- [智能指针](./Smart_Pointer.md)：自动内存管理
- [自定义内存管理](./Custom_Allocator.md)：分配器中的调试钩子
- [C++ 内存模型（并发）](./Memory_Model.md)：数据竞争与 TSan