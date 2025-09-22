# std::list

List 是一个支持在容器的任意位置快速插入和删除元素的一个顺序容器，支持存储各种类型的对象，链表在存储时不需要重新分配内存或是初始化时预留大小，对象存储在内存中的不连续位置，因此随机访问能力较差。

## 1. 引入

```C++
#include <list>
```

## 2. 存储方式

std::list 是基于双向链表实现的容器。与 std::vector 的连续内存不同，std::list 的元素存储在不连续的内存块中。每个元素由一个节点（节点中包含元素和指向前后元素的指针）组成，这些节点通过指针连接成一个链表。由于每个节点都是独立分配的，std::list 不需要连续的内存空间来存储元素。

在 std::list 中插入或删除元素时，由于元素是通过指针连接的，因此插入和删除操作通常是非常高效的。插入、删除操作不会影响链表中其他元素的存储位置。

然而，由于每个元素都需要额外的内存来存储指向前后元素的指针，所以相较于 std::vector，std::list 在存储上较为低效，且不支持快速随机访问。访问元素时，必须从链表的起始或终止节点开始，逐步沿着指针遍历到目标元素。

### 3. 方法

### (1) 构造方法

1. `list()`: 创建一个空的 `std::list`，默认构造函数。

   ```c++
   std::list<int> lst;
   ```

2. `list(size_t nSize)`: 创建一个包含 `nSize` 个默认构造元素的 `std::list`。

   ```c++
   std::list<int> lst(10);  // 创建一个包含10个默认构造的元素的列表
   ```

3. `list(size_t nSize, const T& value)`: 创建一个包含 `nSize` 个元素，并且所有元素的值为 `value` 的 `std::list`。

   ```c++
   std::list<int> lst(10, 5);  // 创建一个包含10个元素，值都为5的列表
   ```

4. `list(const list&)`: 复制构造函数，创建一个与另一个 `std::list` 完全相同的副本。

   ```c++
   std::list<int> lst1 = {1, 2, 3, 4, 5};
   std::list<int> lst2(lst1);  // lst2 是 lst1 的一个复制版本
   ```

5. `list(begin, end)`: 使用另一个容器或迭代器区间中的元素初始化 `std::list`。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   std::list<int> lst(vec.begin(), vec.end());  // 复制 vector 的元素到 list
   ```

### (2) 大小函数

1. `size_t size() const`: 返回 `std::list` 中元素的个数。

   ```c++
   std::list<int> lst = {1, 2, 3, 4};
   std::cout << lst.size();  // 输出 4
   ```

2. `bool empty() const`: 检查 `std::list` 是否为空，若为空返回 `true`，否则返回 `false`。

   ```c++
   std::list<int> lst;
   if (lst.empty()) {
       std::cout << "List is empty." << std::endl;
   }
   ```

3. `void resize(size_t size)`: 调整 `std::list` 的大小。若新大小大于当前大小，`std::list` 会插入默认值的元素；若小于当前大小，则会删除多余的元素。

   ```c++
   std::list<int> lst = {1, 2, 3};
   lst.resize(5, 10);  // 增加两个元素，元素值为 10
   ```

### (3) 增加函数

1. `push_back(const T& value)`: 将元素 `value` 添加到 `std::list` 的末尾。

   ```c++
   std::list<int> lst = {1, 2, 3};
   lst.push_back(4);  // 向 lst 中添加元素 4，lst 变为 {1, 2, 3, 4}
   ```

2. `push_front(const T& value)`: 将元素 `value` 添加到 `std::list` 的开头。

   ```c++
   std::list<int> lst = {2, 3, 4};
   lst.push_front(1);  // 向 lst 中添加元素 1，lst 变为 {1, 2, 3, 4}
   ```

3. `emplace_back(Args&&... args)`: 在 `std::list` 的末尾就地构造一个元素，使用提供的参数直接构造该元素。

   ```c++
   std::list<std::pair<int, int>> lst;
   lst.emplace_back(1, 2);  // 在末尾构造一个 pair<int, int>，值为 {1, 2}
   ```

4. `emplace_front(Args&&... args)`: 在 `std::list` 的前端就地构造一个元素。

   ```c++
   std::list<std::pair<int, int>> lst;
   lst.emplace_front(1, 2);  // 在前端构造一个 pair<int, int>，值为 {1, 2}
   ```

5. `insert(iterator pos, const T& value)`: 在指定位置 `pos` 插入元素 `value`，插入操作会将 `pos` 后面的元素向后移动。

   ```c++
   std::list<int> lst = {1, 2, 4, 5};
   lst.insert(std::next(lst.begin(), 2), 3);  // 在位置2插入3，lst 变为 {1, 2, 3, 4, 5}
   ```

6. `insert(iterator pos, size_t count, const T& value)`: 在指定位置 `pos` 插入 `count` 个元素，所有的元素值为 `value`。

   ```c++
   std::list<int> lst = {1, 2, 4, 5};
   lst.insert(std::next(lst.begin(), 2), 2, 3);  // 在位置2插入两个3，lst 变为 {1, 2, 3, 3, 4, 5}
   ```

7. `insert(iterator pos, InputIterator first, InputIterator last)`: 将 `[first, last)` 区间的元素插入到指定位置 `pos`。

   ```c++
   std::list<int> lst1 = {1, 2, 3};
   std::list<int> lst2 = {4, 5};
   lst1.insert(std::next(lst1.begin(), 2), lst2.begin(), lst2.end());  // 在位置2插入 lst2 中的元素，lst1 变为 {1, 2, 4, 5, 3}
   ```

### (4) 删除函数

1. `pop_back()`: 删除 `std::list` 中的最后一个元素。

   ```c++
   std::list<int> lst = {1, 2, 3, 4};
   lst.pop_back();  // 删除最后一个元素，lst 变为 {1, 2, 3}
   ```

2. `pop_front()`: 删除 `std::list` 中的第一个元素。

   ```c++
   std::list<int> lst = {1, 2, 3, 4};
   lst.pop_front();  // 删除第一个元素，lst 变为 {2, 3, 4}
   ```

3. `erase(iterator pos)`: 删除指定位置 `pos` 处的元素，删除操作后，`pos` 后面的元素会向前移动。

   ```c++
   std::list<int> lst = {1, 2, 3, 4};
   lst.erase(std::next(lst.begin(), 2));  // 删除索引为2的元素，lst 变为 {1, 2, 4}
   ```

4. `erase(iterator first, iterator last)`: 删除区间 `[first, last)` 内的所有元素。

   ```c++
   std::list<int> lst = {1, 2, 3, 4, 5};
   lst.erase(std::next(lst.begin(), 1), std::next(lst.begin(), 4));  // 删除从索引1到3的元素，lst 变为 {1, 5}
   ```

5. `clear()`: 删除 `std::list` 中的所有元素，使容器变为空。

   ```c++
   std::list<int> lst = {1, 2, 3, 4};
   lst.clear();  // 删除所有元素，lst 变为空 {}
   ```

6. `void remove(const T& val)`: `remove` 函数用于删除链表中所有等于指定值 `val` 的元素。它会遍历整个链表，删除所有匹配的元素。

    ```c++
    std::list<int> lst = {1, 2, 3, 4, 3, 5};
    lst.remove(3);  // 删除所有值为3的元素，lst 变为 {1, 2, 4, 5}
    ```

7. `void remove_if (bool (*pred)(const T&));`: pred: 一个谓词（通常是一个函数或函数对象），用于判断元素是否满足删除条件。pred 返回 true 表示删除该元素，返回 false 表示保留该元素。用于删除满足特定条件的所有元素。与 remove 不同的是，remove_if 允许你使用自定义的条件来判断哪些元素需要被删除。

remove_if 会遍历整个容器，将所有满足谓词 pred 条件的元素移到容器的末尾，并返回一个指向容器中第一个不满足条件的元素的迭代器。然后可以使用 erase 来删除这些元素。

需要注意的是，remove_if 并不会实际删除元素，它只是将符合条件的元素移动到容器的末尾，删除操作需要通过 erase 来完成。

假设我们有一个 std::list，并希望删除所有大于 3 的元素：

```c++
#include <iostream>
#include <list>
#include <algorithm>  // 包含 std::remove_if

bool greater_than_three(int n) {
    return n > 3;  // 条件：删除大于3的元素
}

int main() {
    std::list<int> lst = {1, 2, 3, 4, 5, 6};

    // 使用 remove_if 删除大于 3 的元素
    lst.remove_if(greater_than_three);

    // 输出结果
    for (int n : lst) {
        std::cout << n << " ";  // 输出: 1 2 3
    }

    return 0;
}
```

也可以使用 lambda 表达式作为谓词：

```c++
#include <iostream>
#include <list>
#include <algorithm>

int main() {
    std::list<int> lst = {1, 2, 3, 4, 5, 6};

    // 使用 lambda 表达式删除大于 3 的元素
    lst.remove_if([](int n) { return n > 3; });

    // 输出结果
    for (int n : lst) {
        std::cout << n << " ";  // 输出: 1 2 3
    }

    return 0;
}
```

### (5) 遍历函数

1. `iterator begin()`: 返回指向 `std::list` 第一个元素的迭代器。

   ```c++
   std::list<int> lst = {1, 2, 3};
   std::list<int>::iterator it = lst.begin();  // 返回指向第一个元素的迭代器
   std::cout << *it << std::endl;  // 输出: 1
   ```

2. `iterator end()`: 返回指向 `std::list` 最后一个元素之后位置的迭代器。

   ```c++
   std::list<int> lst = {1, 2, 3};
   std::list<int>::iterator it = lst.end();  // 返回指向最后一个元素之后位置的迭代器
   --it;  // 移动到最后一个元素
   std::cout << *it << std::endl;  // 输出: 3
   ```

3. `reverse_iterator rbegin()`: 返回指向 `std::list` 最后一个元素的反向迭代器。

   ```c++
   std::list<int> lst = {1, 2, 3};
   std::list<int>::reverse_iterator rit = lst.rbegin();  // 返回指向最后一个元素的反向迭代器
   std::cout << *rit << std::endl;  // 输出: 3
   ```

4. `reverse_iterator rend()`: 返回指向 `std::list` 第一个元素之前位置的反向迭代器。

   ```c++
   std::list<int> lst = {1, 2, 3};
   std::list<int>::reverse_iterator rit = lst.rend();  // 返回指向第一个元素之前位置的反向迭代器
   ++rit;  // 移动到第一个元素
   std::cout << *rit << std::endl;  // 输出: 1
   ```

5. 使用基于范围的 `for` 循环（C++11 及以上）: 直接遍历 `std::list` 中的每个元素。

   ```c++
   std::list<int> lst = {1, 2, 3};
   for (const int& num : lst) {
       std::cout << num << " ";  // 输出: 1 2 3
   }
   ```

6. 使用传统的 `for` 循环（基于迭代器）: 使用迭代器来遍历 `std::list`。

   ```c++
   std::list<int> lst = {1, 2, 3};
   for (auto it = lst.begin(); it != lst.end(); ++it) {
       std::cout << *it << " ";  // 输出: 1 2 3
   }
   ```

### (6) 其他函数

1. `swap(list& other)`: 交换当前 `std::list` 和另一个 `std::list` 的内容。

   ```c++
   std::list<int> lst1 = {1, 2, 3};
   std::list<int> lst2 = {4, 5, 6};
   lst1.swap(lst2);  // lst1 变为 {4, 5, 6}，lst2 变为 {1, 2, 3}
   ```

2. `assign(size_t count, const T& value)`: 将 `count` 个 `value` 元素赋值给当前 `std::list`，替换原有内容。

   ```c++
   std::list<int> lst;
   lst.assign(5, 10);  // 将 lst 赋值为 {10, 10, 10, 10, 10}
   ```

3. `assign(InputIterator first, InputIterator last)`: 使用区间 `[first, last)` 的元素来填充当前 `std::list`，替换原有内容。

   ```c++
   std::list<int> lst1 = {1, 2, 3};
   std::list<int> lst2;
   lst2.assign(lst1.begin(), lst1.end());  // lst2 变为 {1, 2, 3}
   ```

4. `void sort()`: `sort` 函数用于对链表中的元素进行排序。默认情况下，它会按照元素的升序排列。如果需要按降序排序，可以提供自定义比较函数。

    ```c++
    std::list<int> lst = {5, 3, 4, 1, 2};
    lst.sort();  // 按升序排序，lst 变为 {1, 2, 3, 4, 5}
    ```

    如果要进行降序排序，可以使用自定义的比较函数：

    ```c++
    lst.sort(std::greater<int>());  // 按降序排序，lst 变为 {5, 4, 3, 2, 1}
    ```

5. `void merge(list& other)`: `merge` 函数用于将另一个已排序的链表 `other` 合并到当前链表中。合并后的链表仍然是有序的。注意，`merge` 函数要求两个链表必须是排序过的，否则结果无法保证是有序的。

    ```c++
    std::list<int> lst1 = {1, 3, 5};
    std::list<int> lst2 = {2, 4, 6};
    lst1.merge(lst2);  // lst1 合并 lst2 后，变为 {1, 2, 3, 4, 5, 6}
    ```

6. `void reverse()`: `reverse` 函数用于将链表中的元素反转。它会改变链表中元素的顺序。

    ```c++
    std::list<int> lst = {1, 2, 3, 4, 5};
    lst.reverse();  // 反转链表，lst 变为 {5, 4, 3, 2, 1}
    ```
