# std::shared\_ptr

`std::shared_ptr` 是 C++11 中引入的智能指针类型，用于管理动态分配的内存。它与 `std::unique_ptr` 和 `std::auto_ptr` 的主要区别在于，`std::shared_ptr` 支持 **共享所有权**，即多个 `shared_ptr` 可以共同拥有同一个资源。当最后一个持有该资源的 `shared_ptr` 被销毁时，资源会被自动释放。因此，`std::shared_ptr` 适用于需要多个指针共同管理同一资源的场景。

## 定义与基本功能

`std::shared_ptr` 是一个模板类，能够在多个 `shared_ptr` 之间共享资源的所有权, 其定义如下：

```cpp
template< class T > class shared_ptr;
```

它通过 **引用计数** 机制来管理资源，确保在多个指针指向同一个对象时，资源只有在最后一个指针被销毁时才会被释放。每当一个新的 `shared_ptr` 被创建或拷贝时，引用计数会增加；当一个 `shared_ptr` 被销毁或被重置时，引用计数会减少。当引用计数降到 0 时，`std::shared_ptr` 会自动释放资源。

这种共享所有权的特性使得 `std::shared_ptr` 特别适用于需要多个对象共同管理资源的场景，而不必担心资源在多个指针之间的管理冲突。

## 引用计数机制

`std::shared_ptr` 的核心特点是它的 **引用计数** 机制。每个 `shared_ptr` 都维护着一个与资源关联的引用计数，记录当前有多少个 `shared_ptr` 正在共享同一资源。每当一个新的 `shared_ptr` 被拷贝或赋值时，引用计数增加；当一个 `shared_ptr` 被销毁时，引用计数减少。当引用计数降到 0 时，`std::shared_ptr` 会自动删除所指向的对象，从而避免了内存泄漏。

例如，考虑以下代码：

```cpp
std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>();
std::shared_ptr<MyClass> ptr2 = ptr1;  // 引用计数增加
```

此时，`ptr1` 和 `ptr2` 都指向同一个 `MyClass` 对象，引用计数为 2。当 `ptr1` 或 `ptr2` 被销毁时，引用计数会减少。只有当最后一个 `shared_ptr` 被销毁时，所管理的对象才会被释放。

## 共享所有权

`std::shared_ptr` 允许 **多个指针共享同一资源的所有权**。这种共享所有权的特性使得它在一些需要多个对象共同访问资源的场景中非常有用。例如，在多线程环境中，多个线程可能需要访问共享资源，而 `std::shared_ptr` 可以确保该资源在没有明确的所有权控制的情况下得到正确管理。

```cpp
std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>();
std::shared_ptr<MyClass> ptr2 = ptr1;  // 多个 shared_ptr 共享同一资源
```

在上面的例子中，`ptr1` 和 `ptr2` 都指向同一个对象，它们共享对资源的所有权，引用计数增加为 2。即使某个 `shared_ptr` 被销毁，另一个 `shared_ptr` 仍然可以访问资源。

## 自动销毁与内存管理

与 `std::unique_ptr` 相似，`std::shared_ptr` 会在其生命周期结束时自动释放它所管理的资源。资源的释放是通过引用计数机制来实现的，当最后一个指向资源的 `shared_ptr` 被销毁时，资源会自动释放，从而避免内存泄漏的风险。

这一点非常适合需要共享资源的场景，因为 `shared_ptr` 会自动管理资源的生命周期，不需要开发者显式地调用 `delete`。

```cpp
void createObject() {
    std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>();
    std::shared_ptr<MyClass> ptr2 = ptr1;  // 引用计数增加

    // ptr1 和 ptr2 都会在函数结束时自动销毁
    // 当最后一个 shared_ptr 被销毁时，MyClass 对象将被自动删除
}
```

在上述示例中，`ptr1` 和 `ptr2` 都指向同一个 `MyClass` 对象。当这两个 `shared_ptr` 都超出作用域并被销毁时，引用计数变为 0，`MyClass` 对象会被自动销毁。

## 线程安全

`std::shared_ptr` 的引用计数机制是线程安全的，即多个线程可以安全地同时操作同一个 `shared_ptr`，而不必担心引用计数的竞争条件。这使得 `std::shared_ptr` 非常适合多线程程序中的共享资源管理。

然而，值得注意的是，虽然 `std::shared_ptr` 在引用计数方面是线程安全的，但并不是对资源本身的访问是线程安全的。如果多个线程需要同时访问资源的成员，则需要额外的同步机制（如互斥锁）来保护资源。

## 不支持循环引用

一个需要注意的事项是，`std::shared_ptr` 可能会导致 **循环引用** 的问题。如果两个或多个 `shared_ptr` 互相持有对方的引用，且它们的引用计数永远不会降到 0，那么这些资源将无法释放，从而导致内存泄漏。

例如，考虑以下代码：

```cpp
struct Node {
    std::shared_ptr<Node> next;
};

void createCircularReference() {
    std::shared_ptr<Node> node1 = std::make_shared<Node>();
    std::shared_ptr<Node> node2 = std::make_shared<Node>();

    node1->next = node2;
    node2->next = node1;  // 形成循环引用
}
```

在这个例子中，`node1` 和 `node2` 互相持有对方的 `shared_ptr`，这将导致它们的引用计数永远不为 0，因此资源无法自动释放，造成内存泄漏。为了避免这种情况，可以使用 `std::weak_ptr` 来解决循环引用问题。
