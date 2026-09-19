# std::shared_ptr

`std::shared_ptr` 表示**共享所有权**：多个 `shared_ptr` 可以指向同一个对象，引用计数降到 0 时对象才被销毁。

它解决的是「这个对象到底该由谁释放」的问题——当多个模块都需要访问同一个对象，又无法确定谁活得最久时，共享所有权是自然的答案。代价是额外的控制块和原子操作开销。

## 一、基本用法

```c++
#include <memory>

// 推荐：用 make_shared 创建
auto p1 = std::make_shared<MyClass>(42);

// 也可以从裸指针构造
std::shared_ptr<MyClass> p2(new MyClass(42));

// 拷贝：引用计数增加
auto p3 = p1;
std::cout << p1.use_count();   // 3

p1.reset();                    // 计数减到 2
p3 = nullptr;                  // 计数减到 1
// p2 销毁时计数归零，对象被释放
```

`shared_ptr` 的大小通常是裸指针的两倍（16 字节）——一个指向对象，一个指向控制块。

## 二、引用计数与控制块

`shared_ptr` 内部维护一个**控制块**，记录：

- 强引用计数（`shared_ptr` 的数量）
- 弱引用计数（`weak_ptr` 的数量）
- 删除器
- 分配器

对象在**强引用计数归零**时销毁，控制块在**强弱引用计数都归零**时才释放。

```
shared_ptr p1 ──┐
                ├──→ 控制块 ──→ 对象
shared_ptr p2 ──┘    ├─ 强引用: 2
                     ├─ 弱引用: 0
                     └─ 删除器
```

### make_shared 与 new 的差别

```c++
auto p1 = std::make_shared<MyClass>();              // 一次分配
std::shared_ptr<MyClass> p2(new MyClass());         // 两次分配
```

`make_shared` 把对象和控制块放在**同一块内存**里，只分配一次。这更快，缓存局部性也更好。

但它有个副作用：**对象的内存要等控制块也释放后才能回收**。如果存在 `weak_ptr` 长期观察，对象已经析构但内存不会归还。

另外，`make_shared` 无法指定自定义删除器。需要自定义删除器时只能用 `new` 构造。

## 三、线程安全

这是最容易误解的一点。`shared_ptr` 的线程安全性要分两个层面看：

| 操作 | 是否线程安全 |
| :--- | :--- |
| 多个线程读取同一个 `shared_ptr` 对象 | ✅ 安全 |
| 多个线程操作**不同的** `shared_ptr` 对象（即使指向同一资源） | ✅ 安全 |
| 多个线程操作**同一个** `shared_ptr` 对象 | ❌ 不安全，需要加锁 |
| 通过 `shared_ptr` 访问对象内容 | ❌ 不安全，需要自行同步 |

引用计数本身是原子的，所以「拷贝/销毁不同的 `shared_ptr` 实例」是安全的。但如果多个线程同时读写**同一个** `shared_ptr` 变量（比如一个线程 `reset()`，另一个线程拷贝），就会产生数据竞争。

```c++
std::shared_ptr<Data> g_data;

// 线程 A
g_data = std::make_shared<Data>();   // 写

// 线程 B
auto local = g_data;                 // 读 —— 与 A 竞争！
```

正确做法是加锁，或者用 `std::atomic<std::shared_ptr<T>>`（C++20）：

```c++
std::atomic<std::shared_ptr<Data>> g_data;

g_data.store(std::make_shared<Data>());   // 原子写
auto local = g_data.load();               // 原子读
```

> 注意：`std::atomic<std::shared_ptr<T>>` 在 C++20 才被正式支持。在此之前只能用 `std::atomic_load` / `std::atomic_store` 这些自由函数。

至于**对象内容**的访问，`shared_ptr` 完全不提供保护。多个线程同时修改同一个对象，仍然需要 `mutex` 等同步手段。

## 四、循环引用

`shared_ptr` 最经典的坑是**循环引用**：

```c++
struct Node
{
    std::shared_ptr<Node> next;
    ~Node() { std::cout << "销毁\n"; }
};

void leak()
{
    auto a = std::make_shared<Node>();
    auto b = std::make_shared<Node>();

    a->next = b;   // b 的计数 = 2
    b->next = a;   // a 的计数 = 2
}
// 函数结束，a 和 b 各自减 1，计数都是 1
// 对象永远不会销毁，析构函数不会调用
```

两个对象互相持有对方的 `shared_ptr`，计数永远降不到 0。

解决办法是把其中一条边改成 `weak_ptr`：

```c++
struct Node
{
    std::weak_ptr<Node> next;   // 不增加计数
    ~Node() { std::cout << "销毁\n"; }
};
```

现在 `b->next` 不影响 `a` 的计数，函数结束时两个对象都能正常销毁。

**判断原则**：如果两个对象是「父子」或「主从」关系，让子/从指向父/主用 `weak_ptr`。如果确实需要双向强引用，说明设计可能有问题。

## 五、enable_shared_from_this

有时类内部需要返回指向自己的 `shared_ptr`：

```c++
class Widget
{
public:
    std::shared_ptr<Widget> getSelf()
    {
        return std::shared_ptr<Widget>(this);   // 错误！
    }
};
```

这样写会创建**第二个控制块**，导致同一个对象被释放两次。

正确做法是继承 `enable_shared_from_this`：

```c++
class Widget : public std::enable_shared_from_this<Widget>
{
public:
    std::shared_ptr<Widget> getSelf()
    {
        return shared_from_this();   // 复用已有的控制块
    }
};
```

使用时必须确保对象已经由 `shared_ptr` 管理：

```c++
auto p = std::make_shared<Widget>();
auto q = p->getSelf();   // 正确，q 和 p 共享控制块

Widget w;
auto bad = w.getSelf();  // 错误：w 不是 shared_ptr 管理的，抛 bad_weak_ptr
```

这个模式在异步回调中很常见——回调需要持有 `this` 的 `shared_ptr` 以保证对象存活。

## 六、常用接口

| 接口 | 作用 |
| :--- | :--- |
| `get()` | 返回裸指针 |
| `use_count()` | 返回强引用计数（调试用，多线程下不可靠） |
| `unique()` | 是否独占（C++20 弃用） |
| `reset()` | 释放当前对象 |
| `swap(other)` | 交换 |
| `operator bool` | 是否持有对象 |
| `owner_before(other)` | 比较控制块顺序（用于有序容器） |

```c++
auto p = std::make_shared<int>(42);

std::cout << p.use_count() << '\n';   // 1

auto q = p;
std::cout << p.use_count() << '\n';   // 2

p.reset();
std::cout << q.use_count() << '\n';   // 1
```

`use_count()` 只应用于调试。多线程环境下它的返回值随时可能失效，不要用它做逻辑判断。

## 七、类型转换

`shared_ptr` 之间的转换需要专用函数，不能用 `static_cast`：

```c++
std::shared_ptr<Base> base = std::make_shared<Derived>();

// 向下转换
auto derived = std::dynamic_pointer_cast<Derived>(base);   // 失败返回 nullptr
if (derived)
    derived->derivedMethod();

// 静态转换
auto d2 = std::static_pointer_cast<Derived>(base);

// const 转换
auto c = std::const_pointer_cast<Derived>(base);
```

这些函数会**共享控制块**，引用计数正确维护。直接用 `static_cast` 得到裸指针再构造新的 `shared_ptr` 会导致双重释放。

## 八、什么时候不该用

`shared_ptr` 很方便，但共享所有权会让生命周期推理变难：

- **对象什么时候销毁？** —— 计数归零时，但归零的时机可能很晚，甚至超出你的预期。
- **谁在持有？** —— 需要看所有拷贝点，代码量一大就很难追踪。

因此：

| 场景 | 建议 |
| :--- | :--- |
| 单一所有者 | `unique_ptr` |
| 明确的父子关系 | 父持有 `unique_ptr`，子持有裸指针或引用 |
| 需要跨模块共享且生命周期不定 | `shared_ptr` |
| 缓存、观察者 | `weak_ptr` |
| 函数参数只是借用 | `const T&` 或 `T*` |

**不要把 `shared_ptr` 当默认选择**。它应该用在确实需要共享所有权的地方，而不是「懒得想清楚生命周期」时。

## 九、常见误区

**「`shared_ptr` 是线程安全的」** —— 只有引用计数是原子的。同一个 `shared_ptr` 对象的并发读写、以及对象内容的并发访问都不安全。

**「`use_count()` 可以用来判断是否独占」** —— 多线程下不可靠，标准明确说它只用于调试。

**「`make_shared` 总是更好」** —— 它无法指定自定义删除器，且在有 `weak_ptr` 长期存在时会延迟内存回收。

**「用 `get()` 得到的指针可以构造新的 `shared_ptr`」** —— 会创建独立控制块，导致双重释放。要用 `enable_shared_from_this` 或 `shared_ptr` 的别名构造函数。

**「`shared_ptr` 可以替代 `unique_ptr`」** —— 技术上可以，但会带来不必要的开销和难以推理的生命周期。能用独占就用独占。

**「两个 `shared_ptr` 指向同一对象就共享控制块」** —— 只有通过拷贝或 `make_shared` 才共享。分别用同一个裸指针构造两次，会得到两个独立控制块。

## 十、相关章节

- [智能指针](../Smart_Pointer.md)：三种智能指针的对比与选择
- [unique_ptr](./unique_ptr.md)：独占所有权，默认选择
- [weak_ptr](./weak_ptr.md)：打破循环引用
- [RAII 与资源管理](../RAII.md)：共享所有权的设计思想
- [多线程与并发](../../Concurrency.md)：`shared_ptr` 在并发中的正确用法
