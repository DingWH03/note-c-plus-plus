# std::set

`std::set` 是一个排序的关联容器，它存储唯一的元素，并根据元素的顺序进行自动排序。与 `std::list` 不同，`std::set` 提供了对元素的高效查找、插入和删除操作，但不支持直接的索引访问。其底层通常是基于平衡二叉树实现（红黑树），因此在查找、插入和删除操作上的时间复杂度通常为 O(log n)。

## 1. 引入

```C++
#include <set>
```

## 2. 存储方式

`std::set` 是基于平衡二叉树（通常是红黑树）实现的容器。与 `std::list` 和 `std::vector` 不同，`std::set` 具有自动排序的特性，元素会按升序（默认情况下）进行排列。每个元素都是唯一的，无法存储重复的元素。

在 `std::set` 中，元素会以节点的形式存储，每个节点包含一个元素以及指向父节点、左子节点和右子节点的指针。通过这种方式，`std::set` 保证了所有元素的排序特性，并支持高效的查找、插入和删除操作。

* **插入**：由于元素在插入时会自动排序并且不允许重复，插入一个新元素的时间复杂度为 O(log n)，这也保证了元素始终保持有序。
* **查找**：在 `std::set` 中查找元素的时间复杂度为 O(log n)，这是因为它依赖于平衡二叉树的结构。
* **删除**：删除操作也有 O(log n) 的时间复杂度，因为删除节点时需要保持树的平衡。

与 `std::vector` 和 `std::list` 不同，`std::set` 的元素是按照某种顺序排列的，因此它支持基于顺序的迭代。由于不允许重复元素，`std::set` 会自动忽略重复的插入请求。

尽管 `std::set` 提供了很高效的插入、删除和查找操作，但它并不支持快速的随机访问。要访问集合中的某个元素，必须按顺序遍历元素，无法通过索引直接访问。

## 3. 方法

### (1)构造方法

1. `set()`: 创建一个空的 `std::set`，默认构造函数。

   ```c++
   std::set<int> s;
   std::set<int> s = {1, 2, 3, 4, 5};
   std::set<int> s{1, 2, 3, 4, 5};
   ```

2. `set(const set&)`: 复制构造函数，创建一个与另一个 `std::set` 完全相同的副本。

   ```c++
   std::set<int> s1 = {1, 2, 3, 4, 5};
   std::set<int> s2(s1);  // s2 是 s1 的一个复制版本
   std::set<int> s3(std::move(s3));  // 移动构造（s3 变为有效但未指定状态，移除所有权）
   ```

3. `set(begin, end)`: 使用另一个容器或迭代器区间中的元素初始化 `std::set`。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   std::set<int> s(vec.begin(), vec.end());  // 复制 vector 的元素到 set
   ```

4. `std::set<T&, std::greater<T&>>`: 按照降序创建set

### (2)大小函数

1. `size_t size() const`: 返回 `std::set` 中元素的个数。

   ```c++
   std::set<int> s = {1, 2, 3, 4};
   std::cout << s.size();  // 输出 4
   ```

2. `bool empty() const`: 检查 `std::set` 是否为空，若为空返回 `true`，否则返回 `false`。

   ```c++
   std::set<int> s;
   if (s.empty()) {
       std::cout << "Set is empty." << std::endl;
   }
   ```

3. `size_t max_size() const`: 返回最大可允许的set元素数量值

### (3)增加函数

1. `std::pair<iterator, bool> insert(const T& value)`: 插入元素到set中，返回一个pair，其first是指向插入元素所在位置的迭代器，其second是插入是否成功(存在重复元素几插入失败)

    ```c++
    s.insert({1,2,3});                     // initializer_list
    s.insert(s.begin(), 42);               // 带 hint 的插入
    s.insert(vec.begin(), vec.end());      // 区间插入
    ```

2. `std::pair<iterator, bool> emplace(_Args&&... __args)`: 就地构建并插入元素

3. `iterator emplace_hint(const_iterator __pos, _Args&&... __args)`: 就地构建并插入元素，且指定插入位置，但是如果指定位置错误会发生重排序，最坏情况下可能会降低插入的效率

### (4)删除元素

1. `void clear()`: 删除 `std::set` 中的所有元素，使容器变为空。

   ```c++
   std::set<int> s = {1, 2, 3, 4};
   s.clear();  // 删除所有元素，s 变为空 {}
   ```

2. `void erase(iterator pos)`: 删除指定位置 `pos` 处的元素

   ```c++
   std::set<int> s = {1, 2, 3, 4};
   s.erase(std::next(s.begin(), 2));  // 删除索引为2的元素，lst 变为 {1, 2, 4}
   ```

3. `size_t erase(const key_type& key)`: 删除指定键，返回删除个数。

4. `erase(first, last)`: 删除迭代器区间。

### (5)查找函数

1. `iterator find(const T& value)`: 查找某元素在set中的位置，返回一个指向该元素的迭代器，如果找不到会返回end()

2. `std::pair<const_iterator, const_iterator> equal_range(const key_type& __x) const`: 返回一对迭代器，表示所有具有等于k的键的元素的范围。由于set包含唯一元素，因此下界将是元素本身，上限将指向键k之后的下一个元素。如果没有与键K匹配的元素，则根据容器的内部比较对象(key_comp)，返回的范围的长度为0，两个迭代器均指向大于k的第一个元素。如果键超过了set容器中的最大元素，它将返回一个指向set容器中最后一个元素的迭代器。

    ```c++
        std::set<int> s = {1, 2, 3, 4};
        std::pair p = s.equal_range(2);
        std::cout<<*p.first<<std::endl; // 输出2
        std::cout<<*p.second<<std::endl;// 输出3
    ```

3. `size_t count(const T& value)`: 返回是否存在（0 或 1）。
4. `iterator lower_bound(const T& value)`: 返回第一个不小于 value 的迭代器。
5. `iterator upper_bound(const T& value)`: 返回第一个大于 value 的迭代器。
6. `bool contains(const T& value) const`（C++20）: 检查是否包含元素。

### (6)遍历函数

1. `iterator begin()`: 返回指向 `std::set` 第一个元素的迭代器。

   ```c++
   std::set<int> s = {1, 2, 3};
   std::set<int>::iterator it = s.begin();  // 返回指向第一个元素的迭代器
   std::cout << *it << std::endl;  // 输出: 1
   ```

2. `iterator end()`: 返回指向 `std::set` 最后一个元素之后位置的迭代器。

   ```c++
   std::set<int> s = {1, 2, 3};
   std::set<int>::iterator it = s.end();  // 返回指向最后一个元素之后位置的迭代器
   --it;  // 移动到最后一个元素
   std::cout << *it << std::endl;  // 输出: 3
   ```

3. `reverse_iterator rbegin()`: 返回指向 `std::set` 最后一个元素的反向迭代器。

   ```c++
   std::set<int> s = {1, 2, 3};
   std::set<int>::reverse_iterator rit = s.rbegin();  // 返回指向最后一个元素的反向迭代器
   std::cout << *rit << std::endl;  // 输出: 3
   ```

4. `reverse_iterator rend()`: 返回指向 `std::set` 第一个元素之前位置的反向迭代器。

   ```c++
   std::set<int> s = {1, 2, 3};
   std::set<int>::reverse_iterator rit = s.rend();  // 返回指向第一个元素之前位置的反向迭代器
   ++rit;  // 移动到第一个元素
   std::cout << *rit << std::endl;  // 输出: 1
   ```

5. 使用基于范围的 `for` 循环（C++11 及以上）: 直接遍历 `std::set` 中的每个元素。

   ```c++
   std::set<int> s = {1, 2, 3};
   for (const int& num : s) {
       std::cout << num << " ";  // 输出: 1 2 3
   }
   ```

6. 使用传统的 `for` 循环（基于迭代器）: 使用迭代器来遍历 `std::set`。

   ```c++
   std::set<int> s = {1, 2, 3};
   for (auto it = s.begin(); it != s.end(); ++it) {
       std::cout << *it << " ";  // 输出: 1 2 3
   }
   ```

### (7)其他函数

1. `swap(set& other)`: 交换当前 `std::set` 和另一个 `std::set` 的内容。

   ```c++
   std::set<int> s1 = {1, 2, 3};
   std::set<int> s2 = {4, 5, 6};
   s1.swap(s2);  // s1 变为 {4, 5, 6}，s2 变为 {1, 2, 3}
   ```

2. `void merge(set& other)`: `merge` 会将另一个 set 中不与当前容器重复的元素“移动”过来（O(log n) 插入），保持有序性。即使 `other` 是乱序的（但它本身仍然是一个 set，天然有序），结果依旧有序。

    ```c++
    std::set<int> s1 = {1, 3, 5};
    std::set<int> s2 = {2, 4, 6};
    s1.merge(s2);  // s1 合并 s2 后，变为 {1, 2, 3, 4, 5, 6}
    ```

3. `std::allocator<T> get_allocator() const`: 返回容器所使用的分配器（allocator）。分配器负责容器内存的管理，包括内存的分配、释放等操作。在默认情况下，C++ 标准库容器使用 std::allocator 作为默认的分配器。通过调用 get_allocator()，你可以获取容器使用的分配器，并利用该分配器进行自定义内存操作，或者检查容器如何管理内存。

    ```c++
    #include <iostream>
    #include <set>

    int main() {
        std::set<int> s;

        // 获取分配器
        std::allocator<int> alloc = s.get_allocator();

        // 使用分配器进行内存分配
        int* p = alloc.allocate(5);  // 分配5个int元素的内存

        // 显示分配的内存地址
        std::cout << "Allocated memory at: " << p << std::endl;

        // 记得释放分配的内存
        alloc.deallocate(p, 5);  // 释放之前分配的5个int内存

        return 0;
    }
    ```

4. 比较运算符：支持 ==, !=, <, <=, >, >=，C++20 起支持 <=>，但移除了!=, <, <=, >, >=。
