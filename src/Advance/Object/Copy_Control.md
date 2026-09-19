# 拷贝控制

一个类如果持有资源（堆内存、文件句柄、socket），就必须自己决定：**拷贝它时会发生什么？销毁它时该做什么？**

编译器会为每个类自动生成一组特殊成员函数，但默认行为是「逐成员拷贝」——对指针来说就是拷贝地址，而不是拷贝指向的内容。这往往不是我们想要的。

拷贝控制就是关于这组函数的规则：什么时候该自己写，怎么写才不出错。

## 一、五个特殊成员函数

| 函数 | 签名 | 何时调用 |
| :--- | :--- | :--- |
| **拷贝构造** | `T(const T&)` | 用同类型对象初始化新对象 |
| **移动构造** | `T(T&&)` | 用右值初始化新对象 |
| **拷贝赋值** | `T& operator=(const T&)` | 已有对象被赋值 |
| **移动赋值** | `T& operator=(T&&)` | 已有对象被右值赋值 |
| **析构** | `~T()` | 对象生命周期结束 |

```c++
class Widget
{
public:
    Widget(const Widget& other);              // 拷贝构造
    Widget(Widget&& other) noexcept;          // 移动构造
    Widget& operator=(const Widget& other);   // 拷贝赋值
    Widget& operator=(Widget&& other) noexcept;  // 移动赋值
    ~Widget();                                // 析构
};
```

编译器生成的默认版本是这样的：

- **拷贝构造/赋值**：逐个成员拷贝
- **移动构造/赋值**：逐个成员移动
- **析构**：逐个成员析构

对只含 `int`、`std::string` 这类成员的类，默认版本完全正确。问题出在持有**裸资源**的类上。

## 二、默认行为为什么出错

看一个自己管理内存的类：

```c++
class Buffer
{
public:
    explicit Buffer(std::size_t n) : data_(new int[n]), size_(n) {}
    ~Buffer() { delete[] data_; }

private:
    int* data_;
    std::size_t size_;
};
```

这个类用了默认的拷贝构造和赋值，会出大问题：

```c++
Buffer a(10);
Buffer b = a;      // 默认拷贝构造：b.data_ 和 a.data_ 指向同一块内存

// 函数结束，b 先析构，delete[] b.data_
// 然后 a 析构，delete[] a.data_ —— 重复释放！
```

即使侥幸没崩溃，`a` 和 `b` 共享同一块内存也是灾难：修改 `b` 会影响 `a`，而这显然不是「拷贝」该有的语义。

这叫**浅拷贝**问题——拷贝了指针本身，而不是指针指向的内容。

## 三、拷贝构造与拷贝赋值

### 1. 拷贝构造

```c++
Buffer(const Buffer& other)
    : data_(new int[other.size_])    // 分配新内存
    , size_(other.size_)
{
    std::copy(other.data_, other.data_ + size_, data_);   // 拷贝内容
}
```

要点：

- 参数必须是**引用**。如果按值传参，会无限递归（拷贝实参时又要调用拷贝构造）
- 通常加 `const`，因为不应该修改源对象
- 用**初始化列表**初始化成员，而不是在函数体里赋值

### 2. 拷贝赋值

```c++
Buffer& operator=(const Buffer& other)
{
    if (this != &other)              // 1. 自赋值检查
    {
        int* newData = new int[other.size_];   // 2. 先分配新内存
        std::copy(other.data_, other.data_ + other.size_, newData);

        delete[] data_;              // 3. 再释放旧内存
        data_ = newData;
        size_ = other.size_;
    }
    return *this;                    // 4. 返回 *this 以支持链式赋值
}
```

四个关键点：

**自赋值检查**。`a = a` 是合法的，如果不检查，会先 `delete[] data_` 再拷贝 `other.data_`——但 `other` 就是自己，数据已经被删了。

**先分配再释放**。如果先 `delete[]` 再 `new`，而 `new` 抛异常，对象就处于「data_ 悬垂」的无效状态。反过来则异常安全——分配失败时对象原封不动。

**返回 `*this` 的引用**。这样才能支持 `a = b = c` 这样的链式赋值。返回值而非引用会导致额外的拷贝（对某些类型还可能编译失败）。

**不能返回 `const` 引用**。否则 `(a = b) = c` 无法编译，与内置类型行为不一致。

### 3. 更简洁的写法：copy-and-swap

```c++
Buffer& operator=(Buffer other)   // 注意：按值传参
{
    swap(*this, other);
    return *this;
}

friend void swap(Buffer& a, Buffer& b) noexcept
{
    using std::swap;
    swap(a.data_, b.data_);
    swap(a.size_, b.size_);
}
```

这个写法自动处理了自赋值（因为 `other` 是副本），而且天然异常安全。缺点是可能多一次拷贝，但现代编译器通常能优化掉。

## 四、移动构造与移动赋值

拷贝的代价是「分配 + 复制全部内容」。但如果源对象马上就要销毁，我们完全可以**直接接管它的资源**：

```c++
Buffer(Buffer&& other) noexcept
    : data_(other.data_)          // 接管指针
    , size_(other.size_)
{
    other.data_ = nullptr;        // 关键：置空，避免重复释放
    other.size_ = 0;
}

Buffer& operator=(Buffer&& other) noexcept
{
    if (this != &other)
    {
        delete[] data_;           // 释放自己的资源
        data_ = other.data_;      // 接管对方的
        size_ = other.size_;
        other.data_ = nullptr;
        other.size_ = 0;
    }
    return *this;
}
```

两个要点：

**必须置空源对象**。否则源对象析构时会 `delete[]` 一块已经不属于它的内存。

**应该标记 `noexcept`**。这直接影响 `std::vector` 的行为——如果移动构造可能抛异常，`vector` 扩容时会退化为拷贝，性能差很多。

```c++
class Good { Good(Good&&) noexcept; };   // vector 会移动
class Bad  { Bad(Bad&&); };              // vector 会选择拷贝
```

### 移动后的对象状态

被移动的对象处于「**有效但未指定**」状态：

- 可以安全析构 ✅
- 可以重新赋值 ✅
- 不应该假设它的内容 ❌

```c++
Buffer a(10);
Buffer b = std::move(a);
// a.data_ 是 nullptr，a.size_ 是 0
// 但这是实现细节，不要依赖
a = Buffer(5);   // 合法：重新赋值
```

## 五、Rule of Three / Five / Zero

### Rule of Three（C++98）

> 如果需要自定义**析构函数**、**拷贝构造**、**拷贝赋值**中的任何一个，那么通常三个都需要。

原因：需要自定义析构，说明持有需要手动释放的资源；那么默认的浅拷贝会导致重复释放，必须一并处理。

### Rule of Five（C++11）

> 加上**移动构造**和**移动赋值**，变成五个。

如果定义了拷贝操作但没定义移动操作，编译器**不会**生成移动版本——此时「移动」会静默退化为拷贝：

```c++
class OnlyCopy
{
public:
    OnlyCopy(const OnlyCopy&);              // 自定义拷贝构造
    OnlyCopy& operator=(const OnlyCopy&);
    // 没有移动构造 → std::move 会调用拷贝构造
};

OnlyCopy a;
OnlyCopy b = std::move(a);   // 实际调用的是拷贝构造，不是移动
```

### Rule of Zero

> **最好的做法是一个都不写。**

用现成的 RAII 类型作为成员，编译器生成的默认操作就已经正确：

```c++
class Widget
{
    std::string name_;                 // 自己管理内存
    std::vector<int> data_;            // 自己管理内存
    std::unique_ptr<Impl> impl_;       // 独占所有权
    std::shared_ptr<Config> config_;   // 共享所有权
};
// 不需要写任何特殊成员函数
```

这是现代 C++ 推荐的风格：**让成员类型去管理资源，自己只管组合**。

判断标准很简单：**如果你的类里有裸指针、裸文件句柄、裸 socket，才需要写这五个函数**。否则遵循 Rule of Zero。

## 六、什么时候编译器不生成

编译器生成默认版本是有条件的：

| 情况 | 影响 |
| :--- | :--- |
| 定义了拷贝构造 | 不生成移动构造（但拷贝构造仍默认生成） |
| 定义了移动构造 | 拷贝构造被**删除** |
| 定义了析构 | 不生成移动操作（C++11 起仍生成拷贝，但已弃用） |
| 成员不可拷贝 | 拷贝操作被删除 |
| 成员不可移动 | 移动操作被删除 |

一个容易踩的坑：**只写了析构函数，移动操作就没了**。

```c++
class Widget
{
public:
    ~Widget() { /* 清理 */ }
    // 移动构造不会被生成！
};
```

此时 `std::vector<Widget>` 扩容会退化为拷贝。修复方法是显式声明：

```c++
class Widget
{
public:
    ~Widget() { /* 清理 */ }
    Widget(Widget&&) noexcept = default;
    Widget& operator=(Widget&&) noexcept = default;
};
```

## 七、= default 与 = delete

C++11 允许显式控制这些函数：

```c++
class Widget
{
public:
    Widget() = default;                              // 使用默认实现
    Widget(const Widget&) = default;
    Widget& operator=(const Widget&) = default;
    ~Widget() = default;
};
```

`= delete` 则用于禁止某个操作：

```c++
class NonCopyable
{
public:
    NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete;             // 禁止拷贝
    NonCopyable& operator=(const NonCopyable&) = delete;
};

NonCopyable a;
// NonCopyable b = a;   // 编译错误
```

这比 C++98 时代「声明为 private 且不实现」的做法清晰得多——错误在编译期就暴露，而不是链接期。

## 八、常见误区

**「拷贝构造的参数可以按值传递」** —— 会导致无限递归。必须是引用。

**「赋值运算符不需要自赋值检查」** —— `a = a` 合法且常见（比如通过引用间接赋值），不检查会先释放自己的资源再读取已释放的数据。

**「移动构造不需要置空源对象」** —— 会导致源对象析构时重复释放。

**「移动构造不标 noexcept 也没关系」** —— `vector` 会检测这个标记，没标就退化为拷贝，性能损失可能很大。

**「Rule of Three 只是建议，可以不遵守」** —— 违反它会导致重复释放、内存泄漏等未定义行为，不是风格问题。

**「写了析构函数，编译器还会生成移动操作」** —— 不会。这是 C++11 的一个常见陷阱。

**「`= default` 和什么都不写是一样的」** —— 不完全一样。`= default` 是显式声明，会影响其他特殊成员函数的生成规则。

## 九、相关章节

- [RAII 与资源管理](../Memory/RAII.md)：拷贝控制背后的设计思想
- [C++ 新特性](./CPP_Modern_Features.md)：移动语义与右值引用的细节
- [运算符重载](./Operator_Overloading.md)：赋值运算符的重载
- [虚函数与多态](./Virtual_Function.md)：为什么基类析构函数要声明为 virtual
