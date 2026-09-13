# std::remove / std::remove_if

这两个算法**逻辑移除**范围内满足条件的元素：把保留的元素移到范围前部，并返回新的逻辑末尾。它们**不会真正删除元素**，也不会改变容器大小。

- `remove`：移除所有等于给定值的元素
- `remove_if`：移除所有使谓词返回 `true` 的元素

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt, class T>
constexpr ForwardIt remove(ForwardIt first, ForwardIt last, const T& value);

template<class ForwardIt, class UnaryPred>
constexpr ForwardIt remove_if(ForwardIt first, ForwardIt last, UnaryPred p);
```

- **迭代器要求**：`ForwardIterator`。
- **复杂度**：恰好 \\( n \\) 次比较或谓词调用。
- **返回值**：指向新的逻辑末尾（最后一个保留元素之后的位置）。

实现思路（以 `remove` 为例）：

```c++
template<class ForwardIt, class T>
ForwardIt remove(ForwardIt first, ForwardIt last, const T& value)
{
    first = std::find(first, last, value);
    if (first != last)
        for (ForwardIt i = first; ++i != last; )
            if (!(*i == value))
                *first++ = std::move(*i);
    return first;
}
```

**真正删除必须配合容器的 `erase`**，这就是著名的 **erase-remove 惯用法**：

```c++
v.erase(std::remove(v.begin(), v.end(), value), v.end());
```

C++20 起可以直接用 `std::erase(v, value)` 或 `std::erase_if(v, pred)`，一步完成。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 2, 4, 2, 5};

    // 逻辑移除所有 2
    auto newEnd = std::remove(v.begin(), v.end(), 2);
    // v 的物理内容可能是 {1,3,4,5,4,2,5}（后面是残留元素）
    v.erase(newEnd, v.end());   // 真正删除
    // v = {1,3,4,5}

    // erase-remove 惯用法（一行）
    std::vector<int> v2{1, 2, 3, 4, 5, 6};
    v2.erase(std::remove_if(v2.begin(), v2.end(),
                            [](int x) { return x % 2 == 0; }),
             v2.end());
    // v2 = {1,3,5}

    // C++20 起更简洁
    std::vector<int> v3{1, 2, 3, 4, 5, 6};
    std::erase_if(v3, [](int x) { return x % 2 == 0; });   // v3 = {1,3,5}
}
```

### (2) 谓词与投影

`ranges::remove` / `ranges::remove_if` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 17}};

people.erase(std::ranges::remove_if(people,
                                    [](int age) { return age < 18; },
                                    &Person::age).begin(),
             people.end());
```

### (3) 执行策略

`remove` 和 `remove_if` 支持 C++17 执行策略：

```c++
#include <execution>

std::remove(std::execution::par, v.begin(), v.end(), 2);
```

## 4. 注意事项

- **不改变容器大小**：这是最关键的坑。忘记配合 `erase` 会留下残留元素。
- **残留元素的状态**：被「移除」的元素仍在容器中，处于有效但未指定的状态。
- **C++20 的替代**：`std::erase` / `std::erase_if` 更安全简洁，推荐优先使用。
- **对 `list` 用成员函数**：`std::list::remove` 和 `std::list::remove_if` 会真正删除元素，且是 \\( O(n) \\)。
- **`remove` 用 `operator==`**：自定义类型需要提供 `operator==`。

## 5. 相关算法

- [remove_copy / remove_copy_if](./Remove_copy.md)：复制时省略元素
- [unique](./Unique.md)：移除连续重复元素
- [replace / replace_if](./Replace.md)：替换而非移除
- [partition](../Sorting/Partition.md)：按谓词分组
