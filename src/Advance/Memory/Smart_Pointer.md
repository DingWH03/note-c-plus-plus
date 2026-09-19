# Smart Pointer

智能指针是 RAII 在内存管理上的标准实现。它把「指向对象的指针」和「负责释放」这两件事绑在一起，让堆对象的生命周期由对象自己管理，而不是靠程序员记得写 `delete`。

C++11 引入了三种智能指针，各有明确的使用场景：

| 类型 | 所有权 | 拷贝 | 适用场景 |
| :--- | :--- | :--- | :--- |
| [`std::unique_ptr`](./Smart_Pointer/unique_ptr.md) | 独占 | 不可拷贝，可移动 | **默认选择**，单一所有者 |
| [`std::shared_ptr`](./Smart_Pointer/shared_ptr.md) | 共享 | 可拷贝，引用计数 | 多个对象共享同一资源 |
| [`std::weak_ptr`](./Smart_Pointer/weak_ptr.md) | 不拥有 | 可拷贝 | 打破 `shared_ptr` 的循环引用 |

此外还有 `std::auto_ptr`，它是 C++98 的产物，因为拷贝时隐式转移所有权这一设计缺陷，在 C++11 被弃用、C++17 被移除。现在只作为历史知识了解即可，见 [auto_ptr](./Smart_Pointer/auto_ptr.md)。

## 怎么选

大部分情况下答案很简单：

- **优先用 `unique_ptr`**。它没有额外开销（大小和裸指针一样），语义清晰。需要转移所有权时用 `std::move`。
- **确实需要共享时才用 `shared_ptr`**。它要维护控制块和引用计数，有原子操作开销。
- **出现循环引用时用 `weak_ptr` 打破**。它不增加引用计数，只观察对象是否还活着。

一个常见的误区是「`shared_ptr` 更安全所以都用它」。实际上共享所有权会让生命周期变得难以推理——你不知道对象什么时候被销毁。能用独占就用独占。

## 本章内容

- [unique_ptr](./Smart_Pointer/unique_ptr.md)：独占所有权、自定义删除器、工厂函数
- [shared_ptr](./Smart_Pointer/shared_ptr.md)：引用计数、控制块、线程安全、`enable_shared_from_this`
- [weak_ptr](./Smart_Pointer/weak_ptr.md)：打破循环引用、缓存场景、`lock()` 用法
- [auto_ptr](./Smart_Pointer/auto_ptr.md)：被移除的旧智能指针，以及它为什么被移除
