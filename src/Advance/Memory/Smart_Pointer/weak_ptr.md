# std::weak_ptr

`std::weak_ptr` 是对 `shared_ptr` 管理对象的**弱引用**。它能看到对象，但不参与所有权——不增加强引用计数，因此不会阻止对象被销毁。

它主要解决两个问题：打破 `shared_ptr` 的循环引用，以及在缓存等场景中安全地观察对象是否还活着。

## 一、基本用法

`weak_ptr` 不能直接构造，必须从 `shared_ptr` 得到：

```c++
auto sp = std::make_shared<MyClass>();
std::weak_ptr<MyClass> wp = sp;      // 从 shared_ptr 构造

std::cout << sp.use_count() << '\n';   // 1，weak_ptr 不影响计数
```

访问对象必须先转成 `shared_ptr`：

```c++
if (auto locked = wp.lock())     // 尝试提升
{
    locked->doSomething();       // 对象还活着
}
else
{
    // 对象已被销毁
}
```

`lock()` 是唯一的访问途径，这个设计是刻意的——它把「检查是否存在」和「使用对象」合成一个原子操作，避免了先检查后使用之间的竞态。

## 二、解决循环引用

这是 `weak_ptr` 最经典的用途。看一个双向链表或树结构：

```c++
struct Node
{
    std::shared_ptr<Node> parent;      // 指向父节点
    std::vector<std::shared_ptr<Node>> children;
    ~Node() { std::cout << "销毁节点\n"; }
};

void leak()
{
    auto parent = std::make_shared<Node>();
    auto child = std::make_shared<Node>();

    parent->children.push_back(child);   // child 计数 = 2
    child->parent = parent;              // parent 计数 = 2
}
// 函数结束，两个计数都停在 1，对象永不销毁
```

父节点持有子节点，子节点又持有父节点，形成环。解决办法是把「反向」的那条边改成弱引用：

```c++
struct Node
{
    std::weak_ptr<Node> parent;          // 改成 weak_ptr
    std::vector<std::shared_ptr<Node>> children;
    ~Node() { std::cout << "销毁节点\n"; }
};
```

现在 `child->parent` 不影响 `parent` 的计数，函数结束时两个对象都能正常销毁。

**怎么判断哪条边该用 `weak_ptr`？**

- 树结构：父持有子用 `shared_ptr`，子指回父用 `weak_ptr`
- 观察者模式：被观察者持有观察者用 `shared_ptr`，观察者持有被观察者用 `weak_ptr`
- 一般原则：**让「拥有」的方向用强引用，「引用」的方向用弱引用**

如果两个对象确实需要互相强引用，通常说明设计本身有问题——考虑能否提取出第三个对象来持有它们。

## 三、常用接口

| 接口 | 作用 |
| :--- | :--- |
| `lock()` | 返回 `shared_ptr`，对象已销毁则返回空 |
| `expired()` | 对象是否已销毁 |
| `use_count()` | 强引用计数（调试用） |
| `reset()` | 放弃观察 |
| `swap(other)` | 交换 |
| `owner_before(other)` | 比较控制块顺序 |

```c++
auto sp = std::make_shared<int>(42);
std::weak_ptr<int> wp = sp;

std::cout << wp.expired() << '\n';   // false
std::cout << wp.use_count() << '\n'; // 1

sp.reset();
std::cout << wp.expired() << '\n';   // true
std::cout << wp.lock() << '\n';      // 空指针
```

> `expired()` 和 `use_count()` 在多线程下都不可靠——返回值可能立刻失效。要访问对象，永远用 `lock()` 再判断。

## 四、缓存场景

`weak_ptr` 的另一个常见用途是**缓存**：缓存不应该决定对象的生命周期，只应该「顺便记住」还活着的对象。

```c++
class ImageCache
{
public:
    std::shared_ptr<Image> get(const std::string& path)
    {
        auto it = cache_.find(path);
        if (it != cache_.end())
        {
            if (auto img = it->second.lock())   // 缓存还有效
                return img;
            cache_.erase(it);                    // 已失效，清掉
        }

        auto img = loadImage(path);              // 重新加载
        cache_[path] = img;                      // 存弱引用
        return img;
    }

private:
    std::unordered_map<std::string, std::weak_ptr<Image>> cache_;
};
```

如果缓存用 `shared_ptr`，那么只要缓存还在，图片就永远不会被释放——缓存变成了内存泄漏。用 `weak_ptr` 则不会：没人用图片时它自然销毁，缓存项随之失效。

这个模式也适用于对象池、连接池等场景。

## 五、观察者模式

观察者需要知道被观察者是否还活着，但不应该影响它的生命周期：

```c++
class Subject;

class Observer
{
public:
    virtual void onNotify() = 0;
};

class Subject
{
public:
    void subscribe(std::shared_ptr<Observer> obs)
    {
        observers_.push_back(obs);
    }

    void notify()
    {
        // 清理已销毁的观察者，同时通知存活的
        std::erase_if(observers_, [](auto& wp) {
            if (auto obs = wp.lock()) {
                obs->onNotify();
                return false;
            }
            return true;   // 已失效，移除
        });
    }

private:
    std::vector<std::weak_ptr<Observer>> observers_;
};
```

如果这里用 `shared_ptr`，被观察者会一直持有观察者，导致观察者无法销毁。

## 六、与 shared_ptr 的关系

理解两者关系的关键是**控制块**：

```
shared_ptr ──┐
             ├──→ 控制块 ──→ 对象
weak_ptr ────┘    ├─ 强引用: 1
                  ├─ 弱引用: 1
                  └─ 删除器
```

- 强引用归零 → **对象**被销毁（析构函数调用）
- 强引用和弱引用都归零 → **控制块**被释放

所以只要还有 `weak_ptr` 在观察，控制块就不会释放。这也是 `make_shared` 的一个副作用：对象和控制块在同一块内存里，控制块不释放，对象的内存也就无法归还给系统。

```c++
auto sp = std::make_shared<BigObject>();
std::weak_ptr<BigObject> wp = sp;

sp.reset();   // BigObject 析构，但内存还没还给系统
wp.reset();   // 现在控制块也释放了，内存才真正回收
```

对象大小较大、且存在长期 `weak_ptr` 时，用 `shared_ptr<T>(new T)` 可能比 `make_shared` 更合适。

## 七、常见误区

**「`weak_ptr` 可以直接解引用」** —— 不行，没有 `operator*` 和 `operator->`。必须先 `lock()`。

**「`expired()` 返回 false 就可以安全使用」** —— 不行。检查和使用之间对象可能被销毁，这是典型的竞态。要用 `lock()` 一次完成。

**「`weak_ptr` 会增加引用计数」** —— 不增加**强**引用计数，只增加弱引用计数。对象该销毁时照样销毁。

**「`use_count()` 可以用来判断对象是否存活」** —— 多线程下不可靠。判断存活用 `lock()` 或 `expired()`，且前者更安全。

**「循环引用只能靠 `weak_ptr` 解决」** —— 也可以重新设计对象关系，比如用非拥有指针（裸指针、引用）替代。`weak_ptr` 只是最方便的方案。

**「`weak_ptr` 可以独立存在」** —— 它必须从 `shared_ptr` 或另一个 `weak_ptr` 构造。凭空创建的 `weak_ptr` 是空的。

## 八、相关章节

- [智能指针](../Smart_Pointer.md)：三种智能指针的对比与选择
- [shared_ptr](./shared_ptr.md)：引用计数与控制块
- [unique_ptr](./unique_ptr.md)：独占所有权
- [RAII 与资源管理](../RAII.md)：所有权设计的一般原则

```cpp
std::shared_ptr<MyClass> ptr = wp.lock();  // 转换为 shared_ptr
if (ptr) {
    // 使用 ptr 来访问 MyClass 对象
}
```

通过这种方式，我们可以在必要时访问由 `std::weak_ptr` 管理的对象，但前提是该对象仍然存在。
