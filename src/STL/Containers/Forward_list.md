# std::forward\_list

`std::forward_list` 是一个支持在容器的任意位置快速插入和删除元素的顺序容器。它是一个**单向链表**（singly-linked list）实现，因此它**只支持向前遍历**。相比于 `std::list`，它不存储指向前一个节点的指针，从而提供更节省空间的存储方式，但代价是失去了双向迭代的能力以及快速获取 `size()` 的能力。

## 1\. 引入

```c++
#include <forward_list>
```

## 2\. 存储方式

`std::forward_list` 是基于**单向链表**实现的容器。它的元素存储在内存中的不连续位置，每个元素由一个节点（包含元素和指向**下一个**元素的指针）组成。

- **单向连接：** 每个节点只知道其后继节点，因此只能从头到尾单向遍历。
- **高效的插入和删除：** 与 `std::list` 类似，在任意位置插入和删除元素的时间复杂度为 \\(O(1)\\)，但操作通常需要一个指向**目标位置之前**元素的迭代器。
- **空间优化：** 由于每个元素只存储一个指针（指向下一个元素），它比双向链表 `std::list` 更节省内存。
- **不支持随机访问和 `size()`：** 由于是单向链表，它不支持 \\(O(1)\\) 复杂度的随机访问（例如 `operator[]`），并且为了保持 \\(O(1)\\) 插入/删除的效率，它通常**不提供** `size()` 成员函数（除非是 C++20 引入的 `std::ssize` 全局函数）。

## 3\. 方法

由于 `std::forward_list` 的单向特性，其插入和删除操作与 `std::list` 有显著区别：它**没有** `pop_back()`、`push_back()` 或 `back()`，并且 **`insert` 和 `erase` 操作是在指定位置的**后方**进行。**

### (1) 构造方法

1. `forward_list()`: 创建一个空的 `std::forward_list`，默认构造函数。

    ```c++
    std::forward_list<int> lst;
    ```

2. `forward_list(size_t nSize)`: 创建一个包含 `nSize` 个默认构造元素的 `std::forward_list`。

    ```c++
    std::forward_list<int> lst(10);  // 创建一个包含10个默认构造的元素的列表
    ```

3. `forward_list(size_t nSize, const T& value)`: 创建一个包含 `nSize` 个元素，并且所有元素的值为 `value` 的 `std::forward_list`。

    ```c++
    std::forward_list<int> lst(10, 5);  // 创建一个包含10个元素，值都为5的列表
    ```

4. `forward_list(const forward_list&)`: 复制构造函数，创建一个与另一个 `std::forward_list` 完全相同的副本。

    ```c++
    std::forward_list<int> lst1 = {1, 2, 3};
    std::forward_list<int> lst2(lst1);  // lst2 是 lst1 的一个复制版本
    ```

5. `forward_list(begin, end)`: 使用另一个容器或迭代器区间中的元素初始化 `std::forward_list`。

    ```c++
    std::vector<int> vec = {1, 2, 3};
    std::forward_list<int> lst(vec.begin(), vec.end());  // 复制 vector 的元素到 forward_list
    ```

### (2) 大小函数

1. `bool empty() const`: 检查 `std::forward_list` 是否为空。若为空返回 `true`，否则返回 `false`。

    ```c++
    std::forward_list<int> lst;
    if (lst.empty()) {
        std::cout << "List is empty." << std::endl;
    }
    ```

2. `size_t max_size() const`: 返回容器可能包含的最大元素数。

### (3) 增加函数

`std::forward_list` 的所有插入操作都围绕**前部**或**指定位置之后**进行。

1. `push_front(const T& value)`: 将元素 `value` 添加到 `std::forward_list` 的**开头**。

    ```c++
    std::forward_list<int> lst = {2, 3, 4};
    lst.push_front(1);  // lst 变为 {1, 2, 3, 4}
    ```

2. `emplace_front(Args&&... args)`: 在 `std::forward_list` 的**前端**就地构造一个元素。

    ```c++
    std::forward_list<std::pair<int, int>> lst;
    lst.emplace_front(1, 2);  // 在前端构造一个 pair<int, int>，值为 {1, 2}
    ```

3. `insert_after(const_iterator pos, const T& value)`: 在指定位置 `pos` 的**后面**插入元素 `value`。

    ```c++
    std::forward_list<int> lst = {1, 2, 4, 5};
    auto it = std::next(lst.begin()); // 指向元素 2
    lst.insert_after(it, 3);          // 在 2 后面插入 3，lst 变为 {1, 2, 3, 4, 5}
    ```

4. `insert_after(const_iterator pos, size_t count, const T& value)`: 在指定位置 `pos` 的**后面**插入 `count` 个元素，所有元素值为 `value`。

    ```c++
    std::forward_list<int> lst = {1, 2, 4, 5};
    auto it = std::next(lst.begin());
    lst.insert_after(it, 2, 3);  // 在 2 后面插入两个 3，lst 变为 {1, 2, 3, 3, 4, 5}
    ```

5. `insert_after(const_iterator pos, InputIterator first, InputIterator last)`: 将 `[first, last)` 区间的元素插入到指定位置 `pos` 的**后面**。

### (4) 删除函数

`std::forward_list` 的所有删除操作都围绕**前部**或**指定位置之后**进行。

1. `pop_front()`: 删除 `std::forward_list` 中的**第一个**元素。

    ```c++
    std::forward_list<int> lst = {1, 2, 3, 4};
    lst.pop_front();  // 删除第一个元素，lst 变为 {2, 3, 4}
    ```

2. `erase_after(const_iterator pos)`: 删除指定位置 `pos` **后面**的元素。

    ```c++
    std::forward_list<int> lst = {1, 2, 3, 4};
    auto it = lst.begin(); // 指向元素 1
    lst.erase_after(it);   // 删除 1 后面的元素 2，lst 变为 {1, 3, 4}
    ```

3. `erase_after(const_iterator first, const_iterator last)`: 删除区间 `(first, last)` 内的所有元素。

    ```c++
    std::forward_list<int> lst = {1, 2, 3, 4, 5};
    auto it1 = lst.begin(); // 指向 1
    auto it2 = std::next(it1, 3); // 指向 4
    lst.erase_after(it1, it2); // 删除 1 后面直到 4 之前的所有元素 {2, 3}，lst 变为 {1, 4, 5}
    ```

4. `clear()`: 删除 `std::forward_list` 中的所有元素，使容器变为空。

5. `void remove(const T& val)`: 删除链表中所有等于指定值 `val` 的元素。

6. `void remove_if (bool (*pred)(const T&))`: 删除满足谓词 `pred` 条件的所有元素。

    ```c++
    std::forward_list<int> lst = {1, 2, 3, 4, 5};
    lst.remove_if([](int n) { return n % 2 == 0; }); // 删除偶数，lst 变为 {1, 3, 5}
    ```

### (5) 遍历函数

`std::forward_list` **只支持前向迭代器**。它提供了一个特殊的迭代器 `before_begin()` 用于方便地在链表头部之前进行插入/删除操作。

1. `const_iterator before_begin()`: 返回指向**链表第一个元素之前**位置的特殊迭代器。用于配合 `insert_after` 或 `erase_after` 在链表头部操作。

    ```c++
    std::forward_list<int> lst = {1, 2, 3};
    // 在链表头部插入元素 0
    lst.insert_after(lst.before_begin(), 0); // lst 变为 {0, 1, 2, 3}
    ```

2. `iterator begin()`: 返回指向 `std::forward_list` **第一个**元素的迭代器。

3. `iterator end()`: 返回指向 `std::forward_list` **最后一个元素之后**位置的迭代器。

4. 使用基于范围的 `for` 循环（C++11 及以上）:

    ```c++
    std::forward_list<int> lst = {1, 2, 3};
    for (const int& num : lst) {
        std::cout << num << " ";  // 输出: 1 2 3
    }
    ```

### (6) 其他函数

1. `swap(forward_list& other)`: 交换当前 `std::forward_list` 和另一个 `std::forward_list` 的内容。

2. `assign(size_t count, const T& value)`: 将 `count` 个 `value` 元素赋值给当前 `std::forward_list`，替换原有内容。

3. `assign(InputIterator first, InputIterator last)`: 使用区间 `[first, last)` 的元素来填充当前 `std::forward_list`，替换原有内容。

4. `void sort()`: 对链表中的元素进行排序。默认升序。

    ```c++
    std::forward_list<int> lst = {5, 3, 4, 1, 2};
    lst.sort();  // 按升序排序，lst 变为 {1, 2, 3, 4, 5}
    ```

5. `void merge(forward_list& other)`: 将另一个已排序的链表 `other` 合并到当前链表中。要求两个链表都已排序。

6. `void reverse()`: 将链表中的元素**反转**。

    ```c++
    std::forward_list<int> lst = {1, 2, 3};
    lst.reverse();  // 反转链表，lst 变为 {3, 2, 1}
    ```

7. `void unique()`: 删除链表中所有**连续重复**的元素。

    ```c++
    std::forward_list<int> lst = {1, 2, 2, 3, 3, 3, 4};
    lst.unique(); // lst 变为 {1, 2, 3, 4}
    ```

8. `void splice_after(const_iterator pos, forward_list& other)`: 将 `other` 中的所有元素移动到当前列表 `*this` 中，放在 `pos` **之后**。`other` 变为**空**。

    ```c++
    std::forward_list<int> lst1 = {1, 4};
    std::forward_list<int> lst2 = {2, 3};
    lst1.splice_after(lst1.begin(), lst2); // lst1 变为 {1, 2, 3, 4}，lst2 变为空 {}
    ```
