# std::for_each

`std::for_each` 对范围 `[first, last)` 内的每个元素依次调用给定的一元函数对象 `f`，并按顺序返回该函数对象（C++11 起返回 `std::move(f)`）。

它是最基础的遍历算法，与手写 `for` 循环相比的优势在于：可以配合执行策略并行化，并且函数对象的状态会被保留并返回。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

`for_each` 的实现非常直接，等价于：

```c++
template<class InputIt, class UnaryFunction>
constexpr UnaryFunction for_each(InputIt first, InputIt last, UnaryFunction f)
{
    for (; first != last; ++first)
        f(*first);
    return f;   // C++11 起为 return std::move(f);
}
```

- **迭代器要求**：`InputIterator`，只要求单遍扫描能力。
- **复杂度**：恰好 \\(O(n)\\) 次函数调用。
- **返回值**：返回传入的函数对象。这一点很有用——如果函数对象内部维护了状态（如计数器、累加器），可以通过返回值取出。

`ranges::for_each` 额外支持**投影**（`Proj`），并且返回 `ranges::in_fun_result`，同时给出末尾迭代器和函数对象。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    std::for_each(v.begin(), v.end(), [](int x) {
        std::cout << x << ' ';   // 输出: 1 2 3 4 5
    });
    std::cout << '\n';

    // 修改元素：参数需要取引用
    std::for_each(v.begin(), v.end(), [](int& x) { x *= 2; });
    // v 变为 {2, 4, 6, 8, 10}
}
```

### (2) 谓词与投影

`for_each` 接受的是**一元可调用对象**，本身没有谓词概念。但可以利用返回值的特性，把状态收集出来：

```c++
struct Sum {
    int total = 0;
    void operator()(int x) { total += x; }
};

std::vector<int> v{1, 2, 3, 4, 5};
Sum s = std::for_each(v.begin(), v.end(), Sum{});
std::cout << s.total;   // 输出: 15
```

`ranges::for_each` 支持投影，可以只对元素的某个成员操作：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}};

std::ranges::for_each(people, [](int age) { std::cout << age << ' '; },
                      &Person::age);   // 输出: 30 25
```

### (3) 执行策略

`for_each` 和 `for_each_n` 是**唯一不要求元素可平凡复制**的并行算法——即使元素类型有非平凡的构造/析构函数，也可以安全地使用并行策略。

```c++
#include <execution>

std::for_each(std::execution::par, v.begin(), v.end(), [](int& x) { x *= 2; });
```

但要注意：并行执行时**调用顺序不确定**，且不同线程可能同时访问同一元素，必须自行保证线程安全。

## 4. 注意事项

- **函数对象按值传递**：`f` 是按值传入的，如果需要修改外部变量，应使用引用捕获的 lambda 或 `std::ref`。
- **不要修改容器结构**：在 `for_each` 中调用 `push_back`、`erase` 等会改变容器大小的操作会导致迭代器失效。
- **`for_each` 不保证顺序**（当使用执行策略时）；顺序执行时是严格按顺序的。
- **返回值易被忽略**：如果函数对象携带状态，忘记接收返回值就取不到结果。
- **与范围 `for` 的选择**：只是简单遍历时，基于范围的 `for` 循环更简洁；需要并行或需要取回函数对象状态时才用 `for_each`。

## 5. 相关算法

- [for_each_n](./For_each_n.md)：只处理前 \\( N \\) 个元素
- [transform](../Modifying/Transform.md)：遍历的同时把结果写入目标范围
- [generate](../Modifying/Generate.md)：用函数返回值填充范围
- [count / count_if](./Count.md)：统计满足条件的元素数量
