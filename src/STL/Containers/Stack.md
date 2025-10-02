# std::stack

`std::stack` (栈) 是一个**容器适配器 (Container Adapter)**，它不是一个独立的容器，而是对现有顺序容器（如 `std::deque`、`std::list` 或 `std::vector`）的封装，限制了对其元素的访问，使其遵循 **LIFO (Last-In, First-Out)** 的数据结构规则。

想象一下堆叠起来的盘子，你只能在**顶部**放一个新盘子，也只能从**顶部**取走盘子。在 `std::stack` 中，这个“顶部”对应于底层容器的**尾部**。

## 1\. 引入

```c++
#include <stack>
```

## 2\. 存储方式

`std::stack` 通过封装一个**底层容器 (Underlying Container)** 来存储数据。它只暴露了符合栈（LIFO）操作的方法，例如：

- **推入 (Push)：** 插入元素到栈顶（对应底层容器的 `push_back()`）。
- **弹出 (Pop)：** 移除栈顶元素（对应底层容器的 `pop_back()`）。
- **查看 (Top)：** 访问栈顶元素（对应底层容器的 `back()`）。

**默认底层容器**：如果创建 `std::stack` 时不指定底层容器，它默认使用 **`std::deque<T>`**。

**可选底层容器**：任何满足 LIFO 访问要求的顺序容器都可以作为底层容器，包括：

1. **`std::deque<T>` (默认)**：通常是栈操作的最佳选择，因为它在两端操作（`push_front/pop_front` 和 `push_back/pop_back`）都很快。
2. **`std::vector<T>`**：如果内存是连续的且不需要在底部（前端）操作，这也是一个高效的选择。
3. **`std::list<T>`**：如果需要存储不支持连续存储或拷贝构造/赋值操作的复杂对象，可以使用它。

**声明带指定底层容器的栈：**

```c++
// 使用 vector 作为底层容器
std::stack<int, std::vector<int>> stack_vec;

// 使用 list 作为底层容器
std::stack<int, std::list<int>> stack_list;
```

## 3\. 方法

由于 `std::stack` 是一个适配器，它不提供任何迭代器（如 `begin()`/`end()`），也不支持随机访问（如 `operator[]`），只能通过 LIFO 接口进行操作。

### (1) 构造方法

1. `stack()`: 创建一个空的 `std::stack`。

    ```c++
    std::stack<int> s; // 使用默认的 std::deque 作为底层容器
    ```

2. `stack(const Container& cont)`: 使用一个已存在的底层容器副本进行初始化。

    ```c++
    std::deque<int> d = {1, 2, 3};
    std::stack<int> s(d);  // s 变为 {1, 2, 3} (1 是底，3 是顶)
    ```

3. `stack(const stack&)`: 复制构造函数。

### (2) 大小函数

1. `size_t size() const`: 返回 `std::stack` 中元素的个数。

    ```c++
    std::stack<int> s = {1, 2, 3};
    std::cout << s.size();  // 输出 3
    ```

2. `bool empty() const`: 检查 `std::stack` 是否为空。若为空返回 `true`。

    ```c++
    std::stack<int> s;
    if (s.empty()) {
        std::cout << "Stack is empty." << std::endl;
    }
    ```

### (3) 元素访问函数

`std::stack` **只允许访问顶部元素**。

1. `reference top()`: 返回栈顶元素（即最近 `push` 进去的元素）的引用。调用前**必须**确保栈不为空。

    ```c++
    std::stack<int> s;
    s.push(10);
    s.push(20);
    std::cout << s.top(); // 输出 20
    ```

### (4) 增加函数 (推入)

1. `void push(const T& value)`: 将元素 `value` 复制或移动到栈的**顶部**。

    ```c++
    std::stack<int> s;
    s.push(10); // 栈变为 {10}
    s.push(20); // 栈变为 {10, 20} (20 在顶)
    ```

2. `template<class... Args> void emplace(Args&&... args)`: 在栈的顶部**就地构造**一个元素，无需拷贝或移动。这是推荐的推入方式，尤其是对于复杂对象。

    ```c++
    std::stack<std::pair<int, int>> s;
    s.emplace(1, 2); // 直接在栈顶构造 pair {1, 2}
    ```

### (5) 删除函数 (弹出)

1. `void pop()`: 移除栈顶的元素。此函数**不返回**被移除的元素。调用前**必须**确保栈不为空。

    ```c++
    std::stack<int> s;
    s.push(10);
    s.push(20); // 栈为 {10, 20}
    s.pop();    // 移除 20，栈变为 {10}
    // 注意：要获取并移除元素，需要先调用 top()，再调用 pop()
    ```

### (6) 其他函数

1. `void swap(stack& other)`: 交换当前 `std::stack` 和另一个 `std::stack` 的内容。底层容器的内容也会随之交换。

    ```c++
    std::stack<int> s1; s1.push(1);
    std::stack<int> s2; s2.push(2);
    s1.swap(s2); // s1 变为 {2}，s2 变为 {1}
    ```
