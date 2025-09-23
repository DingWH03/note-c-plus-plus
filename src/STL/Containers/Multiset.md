# std::multiset

`std::multiset` 是与 `std::set` 类似的一个容器，但与 `std::set` 不同，`std::multiset` 允许存储重复的元素。在 `std::multiset` 中，元素会按顺序排列，并且可以包含多个相同的元素。`std::multiset` 也提供了高效的查找、插入和删除操作，底层通常使用红黑树结构实现，时间复杂度通常为 O(log n)。

## 1. 引入

```C++
#include <set>
```

## 2. 存储方式

`std::multiset` 是基于平衡二叉树（通常是红黑树）实现的容器。与 `std::set` 相似，`std::multiset` 也具有自动排序的特性，元素会按升序（默认情况下）进行排列。但是，不同于 `std::set`，`std::multiset` 允许存储重复的元素。

在 `std::multiset` 中，元素存储在节点中，每个节点包含一个元素及其数量（即该元素出现的次数）。这种方式保证了所有元素的排序特性，同时允许插入多个相同的元素。

* **插入**：插入一个新元素的时间复杂度为 O(log n)，并且插入重复元素时，`std::multiset` 会将元素的数量增加，而不是替换现有的元素。
* **查找**：在 `std::multiset` 中查找元素的时间复杂度为 O(log n)，因为它依赖于平衡二叉树的结构。
* **删除**：删除操作也有 O(log n) 的时间复杂度，因为删除节点时需要保持树的平衡。

与 `std::set` 不同，`std::multiset` 允许插入重复元素，因此 `std::multiset` 可以包含多个相同的元素。std::multiset也不支持快速的随机访问。

## 3. 方法

### (1) 构造方法

1. `multiset()`: 创建一个空的 `std::multiset`，默认构造函数。

   ```c++
   std::multiset<int> ms;
   std::multiset<int> ms = {1, 2, 3, 4, 5};
   ```

2. `multiset(const multiset&)`: 复制构造函数，创建一个与另一个 `std::multiset` 完全相同的副本。

   ```c++
   std::multiset<int> ms1 = {1, 2, 3, 4, 5};
   std::multiset<int> ms2(ms1);  // ms2 是 ms1 的一个复制版本
   ```

3. `multiset(begin, end)`: 使用另一个容器或迭代器区间中的元素初始化 `std::multiset`。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   std::multiset<int> ms(vec.begin(), vec.end());  // 复制 vector 的元素到 multiset
   ```

4. `std::multiset<T&, std::greater<T&>>`: 按照降序创建 `multiset`。

### (2) 大小函数

1. `size_t size() const`: 返回 `std::multiset` 中元素的个数。

   ```c++
   std::multiset<int> ms = {1, 2, 3, 4};
   std::cout << ms.size();  // 输出 4
   ```

2. `bool empty() const`: 检查 `std::multiset` 是否为空，若为空返回 `true`，否则返回 `false`。

   ```c++
   std::multiset<int> ms;
   if (ms.empty()) {
       std::cout << "Multiset is empty." << std::endl;
   }
   ```

3. `size_t max_size() const`: 返回最大可允许的 `multiset` 元素数量。

### (3) 增加函数

1. `std::pair<iterator, bool> insert(const T& value)`: 插入元素到 `multiset` 中，返回一个 `pair`，其 `first` 是指向插入元素所在位置的迭代器，`second` 是插入是否成功（是否为重复元素）。

   ```c++
   ms.insert({1, 2, 3});                     // initializer_list
   ms.insert(ms.begin(), 42);               // 带 hint 的插入
   ms.insert(vec.begin(), vec.end());      // 区间插入
   ```

2. `std::pair<iterator, bool> emplace(_Args&&... __args)`: 就地构建并插入元素。

3. `iterator emplace_hint(const_iterator __pos, _Args&&... __args)`: 就地构建并插入元素，且指定插入位置。

### (4) 删除元素

1. `void clear()`: 删除 `std::multiset` 中的所有元素，使容器变为空。

   ```c++
   std::multiset<int> ms = {1, 2, 3, 4};
   ms.clear();  // 删除所有元素，ms 变为空 {}
   ```

2. `void erase(iterator pos)`: 删除指定位置 `pos` 处的元素。

   ```c++
   std::multiset<int> ms = {1, 2, 3, 4};
   ms.erase(std::next(ms.begin(), 2));  // 删除索引为2的元素，ms 变为 {1, 2, 4}
   ```

3. `size_t erase(const key_type& key)`: 删除指定键，返回删除个数。

4. `erase(first, last)`: 删除迭代器区间。

### (5) 查找函数

1. `iterator find(const T& value)`: 查找某元素在 `multiset` 中的位置，返回一个指向该元素的迭代器，如果找不到会返回 `end()`。

2. `std::pair<const_iterator, const_iterator> equal_range(const key_type& __x) const`: 返回一对迭代器，表示所有具有等于 `k` 的键的元素的范围。由于 `multiset` 允许重复元素，因此该范围可能包含多个相同的元素。如果没有与键K匹配的元素，则根据容器的内部比较对象(key_comp)，返回的范围的长度为0，两个迭代器均指向大于k的第一个元素。如果键超过了multiset容器中的最大元素，它将返回一个指向multiset容器中最后一个元素的迭代器。

   ```c++
       std::multiset<int> ms = {1, 2, 3, 3, 4};
       std::pair p = ms.equal_range(3);
       std::cout << *p.first << std::endl; // 输出 3
       std::cout << *p.second << std::endl;// 输出 4
   ```

3. `size_t count(const T& value)`: 返回元素出现的次数。
4. `iterator lower_bound(const T& value)`: 返回第一个不小于 `value` 的迭代器。
5. `iterator upper_bound(const T& value)`: 返回第一个大于 `value` 的迭代器。
6. `bool contains(const T& value) const`（C++20）：检查是否包含元素。

### (6) 遍历函数

1. `iterator begin()`: 返回指向 `std::multiset` 第一个元素的迭代器。

   ```c++
   std::multiset<int> ms = {1, 2, 3};
   std::multiset<int>::iterator it = ms.begin();  // 返回指向第一个元素的迭代器
   std::cout << *it << std::endl;  // 输出: 1
   ```

2. `iterator end()`: 返回指向 `std::multiset` 最后一个元素之后位置的迭代器。

3. `reverse_iterator rbegin()`: 返回指向 `std::multiset` 最后一个元素的反向迭代器。

4. `reverse_iterator rend()`: 返回指向 `std::multiset` 第一个元素之前位置的反向迭代器。

5. 使用基于范围的 `for` 循环（C++11 及以上）: 直接遍历 `std::multiset` 中的每个元素。

   ```c++
   std::multiset<int> ms = {1, 2, 3};
   for (const int& num : ms) {
       std::cout << num << " ";  // 输出: 1 2 3
   }
   ```

6. 使用传统的 `for` 循环（基于迭代器）: 使用迭代器来遍历 `std::multiset`。

   ```c++
   std::multiset<int> ms = {1, 2, 3};
   for (auto it = ms.begin(); it != ms.end(); ++it) {
       std::cout << *it << " ";  // 输出: 1 2 3
   }
   ```

### (7) 其他函数

1. `swap(multiset& other)`: 交换当前 `std::multiset` 和另一个 `std::multiset` 的内容。

2. `void merge(multiset& other)`: 将另一个 `multiset` 中的元素合并到当前 `multiset`，如果元素重复，则保留所有元素。

   ```c++
   std::multiset<int> ms1 = {1, 3, 5};
   std::multiset<int> ms2 = {2, 4, 6};
   ms1.merge(ms2);  // ms1 合并 ms2 后，变为 {1, 2, 3, 4, 5, 6}
   ```

3. `std::allocator<T> get_allocator() const`: 返回容器所使用的分配器（allocator）。
