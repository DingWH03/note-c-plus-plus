# std::unique_ptr

`std::unique_ptr` 是 C++11 引入的智能指针，表示对资源的**独占所有权**。它应该是你管理堆对象时的默认选择——没有额外开销，语义清晰，不会出现「不知道谁负责释放」的问题。

## 一、基本用法

```c++
#include <memory>

// 推荐：用 make_unique 创建（C++14）
auto p = std::make_unique<MyClass>(42);

// 也可以用裸指针构造（C++11）
std::unique_ptr<MyClass> p2(new MyClass(42));

p->doSomething();   // 像普通指针一样使用
(*p).doSomething();
```

`unique_ptr` 的大小和裸指针一样（64 位系统上 8 字节），因为它不需要存引用计数之类的额外信息——所有权是唯一的，销毁时机在编译期就确定了。

> 优先用 `std::make_unique` 而不是 `new`。除了更简洁，它还能避免一个微妙的异常安全问题：
>
> ```c++
> // 危险：如果 MyClass 构造抛异常，或者 g() 抛异常，
> // 都可能让 new 出来的内存泄漏
> f(std::unique_ptr<MyClass>(new MyClass()), g());
>
> // 安全
> f(std::make_unique<MyClass>(), g());
> ```

## 二、所有权转移

`unique_ptr` **不能拷贝**，只能移动：

```c++
auto p1 = std::make_unique<MyClass>();
// auto p2 = p1;                 // 编译错误：不能拷贝
auto p2 = std::move(p1);         // 正确：转移所有权

// 此时 p1 变为 nullptr
if (!p1)
    std::cout << "p1 已失去所有权\n";
```

这个限制是有意设计的。如果允许拷贝，两个 `unique_ptr` 会指向同一对象，销毁时就会重复释放。用 `std::move` 显式转移，代码里所有权流转一目了然。

### 作为函数参数与返回值

```c++
// 接管所有权：调用者不能再使用
void takeOwnership(std::unique_ptr<MyClass> p);

// 只借用：不涉及所有权转移
void useObject(const MyClass& obj);
void useObject(MyClass* obj);

// 返回所有权：把资源交给调用者
std::unique_ptr<MyClass> create()
{
    return std::make_unique<MyClass>();   // 移动语义，无拷贝开销
}
```

函数返回局部 `unique_ptr` 时不需要写 `std::move`，编译器会自动应用移动（NRVO 或隐式移动）。

## 三、常用接口

| 接口 | 作用 |
| :--- | :--- |
| `get()` | 返回裸指针，不转移所有权 |
| `release()` | 放弃所有权并返回裸指针，**需要手动 delete** |
| `reset(p)` | 释放当前对象，接管新指针 `p`（默认 `nullptr`） |
| `swap(other)` | 交换两个 `unique_ptr` |
| `operator bool` | 是否持有对象 |
| `operator*` / `operator->` | 访问对象 |

```c++
auto p = std::make_unique<int>(42);

int* raw = p.get();          // 只读，p 仍然拥有

p.reset();                   // 释放对象，p 变为空
p.reset(new int(100));       // 释放旧的，接管新的

if (p)                       // 判空
    std::cout << *p << '\n';
```

`release()` 要小心使用——它把所有权交还给裸指针，之后必须自己 `delete`：

```c++
auto p = std::make_unique<int>(42);
int* raw = p.release();   // p 变空，raw 需要手动管理
delete raw;               // 别忘了
```

它主要用于与 C 风格接口交接所有权。

## 四、数组支持

`unique_ptr` 有数组特化版本 `unique_ptr<T[]>`，它使用 `delete[]` 而非 `delete`：

```c++
auto arr = std::make_unique<int[]>(10);   // 10 个 int

arr[0] = 1;      // 支持 operator[]
arr[9] = 10;
// 自动 delete[]，无需手动管理
```

不过大多数情况下 `std::vector` 是更好的选择——它知道自己的大小，支持迭代器和算法。

## 五、自定义删除器

`unique_ptr` 的第二个模板参数是删除器，可以自定义释放方式：

```c++
template<class T, class Deleter = std::default_delete<T>>
class unique_ptr;
```

这让 `unique_ptr` 能管理任何「需要释放」的资源，不限于 `new` 出来的内存：

```c++
// 管理文件
auto fileDeleter = [](FILE* f) { if (f) fclose(f); };
std::unique_ptr<FILE, decltype(fileDeleter)> file(fopen("a.txt", "r"), fileDeleter);

// 管理 malloc 的内存
std::unique_ptr<int, decltype(&std::free)> buf(
    (int*)std::malloc(sizeof(int) * 10), &std::free);

// 管理 socket
auto socketDeleter = [](int* fd) { if (*fd >= 0) close(*fd); delete fd; };
std::unique_ptr<int, decltype(socketDeleter)> sock(new int(fd), socketDeleter);
```

注意：**使用自定义删除器后，`unique_ptr` 的大小可能变大**（需要存储删除器对象）。无状态的删除器（如函数指针、无捕获 lambda）通常不增加大小，因为编译器可以做空基类优化。

## 六、作为类成员

`unique_ptr` 常用于实现 **Pimpl 惯用法**（Pointer to Implementation），把实现细节隐藏起来：

```c++
// widget.h
class Widget
{
public:
    Widget();
    ~Widget();                          // 必须在 .cpp 中定义
    Widget(Widget&&) noexcept;          // 同上
    Widget& operator=(Widget&&) noexcept;

    void doSomething();

private:
    struct Impl;
    std::unique_ptr<Impl> impl_;        // 只暴露前置声明
};

// widget.cpp
struct Widget::Impl
{
    std::string data;
    // 复杂的实现细节
};

Widget::Widget() : impl_(std::make_unique<Impl>()) {}
Widget::~Widget() = default;            // Impl 在这里才是完整类型
```

这样做的好处：

- 头文件不需要包含实现依赖，加快编译
- 修改实现不会导致使用方重新编译
- 保持 ABI 稳定

注意析构函数必须在 `.cpp` 中定义——因为在头文件里 `Impl` 还是不完整类型，`unique_ptr` 的析构需要完整类型。

## 七、作为工厂函数返回值

工厂函数返回 `unique_ptr` 是标准做法，把所有权明确交给调用者：

```c++
class Shape
{
public:
    virtual ~Shape() = default;
    virtual void draw() const = 0;
};

std::unique_ptr<Shape> createShape(const std::string& type)
{
    if (type == "circle")
        return std::make_unique<Circle>();
    if (type == "square")
        return std::make_unique<Square>();
    return nullptr;
}

// 使用
auto shape = createShape("circle");
if (shape)
    shape->draw();
```

这里基类的析构函数必须是 `virtual`，否则通过基类指针删除派生类对象是未定义行为。

## 八、常见误区

**「`unique_ptr` 不能放进容器」** —— 可以，但容器需要支持移动。`std::vector<std::unique_ptr<T>>` 是合法的，只是不能用需要拷贝的操作（如 `std::copy`）。

**「返回 `unique_ptr` 一定要写 `std::move`」** —— 返回局部变量时不需要，编译器会隐式移动。写 `std::move` 反而可能阻止 NRVO 优化。

**「`get()` 返回的指针可以随便用」** —— `get()` 只是借用，`unique_ptr` 销毁后该指针立刻悬垂。不要保存它。

**「`release()` 之后 `unique_ptr` 还会释放」** —— 不会。`release()` 就是放弃所有权，之后必须自己管理。

**「自定义删除器不影响大小」** —— 有状态的删除器会增加 `unique_ptr` 的大小。无状态删除器通常不增加。

**「`unique_ptr` 可以拷贝到 `shared_ptr`」** —— 可以，用移动：`std::shared_ptr<T> sp = std::move(up);`。这是把独占所有权升级为共享所有权的唯一方式。

## 九、相关章节

- [智能指针](../Smart_Pointer.md)：三种智能指针的对比与选择
- [shared_ptr](./shared_ptr.md)：需要共享所有权时
- [RAII 与资源管理](../RAII.md)：`unique_ptr` 背后的设计思想
- [拷贝控制](../../Object/Copy_Control.md)：移动语义与 Rule of Five
