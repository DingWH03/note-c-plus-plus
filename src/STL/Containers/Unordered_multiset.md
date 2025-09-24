# std::unordered\_multiset

`std::unordered_multiset` 是一个类似于 `std::unordered_set` 的容器，唯一的区别在于它允许元素重复。它同样使用哈希表实现，不保证元素的顺序，能够提供高效的查找、插入和删除操作。

为了支持重复键，`unordered_multiset`通过将每个元素存储为一个“桶”中的链表或其他结构。每个“桶”会存储多个相同的元素，而 `unordered_set` 只允许每个桶中最多存储一个元素。

`unordered_multiset` 的模板定义如下：

```cpp
std::unordered_multiset<Key, Hash = std::hash<Key>, Pred = std::equal_to<Key>, Alloc = std::allocator<Key>>
```

* Key：存储在 `unordered_multiset` 中的元素类型。
* Hash：用于计算元素哈希值的函数或函数对象，默认为 `std::hash<Key>`。
* Pred：用于比较两个元素是否相等的二元谓词，默认为 `std::equal_to<Key>`。
* Alloc：分配器类型，用于内存分配，默认为 `std::allocator<Key>`。

## 1. 引入

```cpp
#include <unordered_set>
```

## 2. 存储方式

`std::unordered_multiset` 使用哈希表（hash table）来存储元素。每个元素的存储位置由哈希函数计算出来，多个相同的元素会被存储在同一个桶中。

* **插入**：插入一个元素时，哈希函数计算该元素的哈希值，将元素存储在相应的桶中，允许同一元素的多个副本被插入。
* **查找**：查找操作的时间复杂度通常为 O(1)，但最坏情况下会退化为 O(n)，具体取决于哈希表的负载因子和哈希冲突。
* **删除**：删除操作通常是 O(1)，但最坏情况下会退化为 O(n)。

与 `unordered_set` 不同，`unordered_multiset` 允许在同一个哈希表的桶中存储多个相同的元素。

## 3. 方法

### (1) 构造方法

1. `unordered_multiset()`：创建一个空的 `std::unordered_multiset`，使用默认构造函数。

   ```cpp
   std::unordered_multiset<int> us;
   std::unordered_multiset<int> us = {1, 2, 3, 4, 5};
   ```

2. `unordered_multiset(const unordered_multiset&)`：复制构造函数，创建一个与另一个 `std::unordered_multiset` 完全相同的副本。

   ```cpp
   std::unordered_multiset<int> us1 = {1, 2, 3, 4, 5};
   std::unordered_multiset<int> us2(us1);  // us2 是 us1 的一个复制版本
   ```

3. `unordered_multiset(begin, end)`：使用另一个容器或迭代器区间中的元素初始化 `std::unordered_multiset`。

   ```cpp
   std::vector<int> vec = {1, 2, 3, 4, 5};
   std::unordered_multiset<int> us(vec.begin(), vec.end());  // 复制 vector 的元素到 unordered_multiset
   ```

### (2) 大小函数

1. `size_t size() const`：返回 `std::unordered_multiset` 中元素的个数。

   ```cpp
   std::unordered_multiset<int> us = {1, 2, 3, 4};
   std::cout << us.size();  // 输出 4
   ```

2. `bool empty() const`：检查 `std::unordered_multiset` 是否为空，若为空返回 `true`，否则返回 `false`。

   ```cpp
   std::unordered_multiset<int> us;
   if (us.empty()) {
       std::cout << "Unordered Multiset is empty." << std::endl;
   }
   ```

3. `size_t max_size() const`：返回最大可允许的 `unordered_multiset` 元素数量。

4. `void reserve(size_t n)`：用于预先分配哈希表的桶数量。这可以有效减少动态扩容的开销，特别是在你知道将要插入大量元素时。

### (3) 增加函数

1. `std::pair<iterator, bool> insert(const T& value)`：插入元素到 `unordered_multiset` 中，返回一个 `pair`，其中 `first` 是指向插入元素所在位置的迭代器，`second` 是插入是否成功（如果存在重复元素，则插入成功并增加该元素的数量）。

   ```cpp
   us.insert({1, 2, 3});  // 插入多个元素
   us.insert(us.begin(), 42);  // 带 hint 的插入
   us.insert(vec.begin(), vec.end());  // 区间插入
   ```

2. `std::pair<iterator, bool> emplace(_Args&&... args)`：就地构建并插入元素。

3. `iterator emplace_hint(const_iterator hint, _Args&&... args)`：就地构建并插入元素，并指定插入位置。与 `insert` 一样，最坏情况下会发生重排序。在哈希索引的前提下是无序的，因此该插入操作等效于`insert`。

### (4) 删除元素

1. `void clear()`：删除 `std::unordered_multiset` 中的所有元素，使容器变为空。

   ```cpp
   std::unordered_multiset<int> us = {1, 2, 3, 4};
   us.clear();  // 删除所有元素，us 变为空 {}
   ```

2. `void erase(iterator pos)`：删除指定位置 `pos` 处的元素。

   ```cpp
   std::unordered_multiset<int> us = {1, 2, 3, 4};
   us.erase(std::next(us.begin(), 2));  // 删除索引为2的元素，us 变为 {1, 2, 4}
   ```

3. `size_t erase(const key_type& key)`：删除指定键的元素并返回删除的数量，注意一个键可能有多个副本，因此返回的数量可能大于 1。

   ```cpp
   std::unordered_multiset<int> us = {1, 2, 2, 3, 4};
   us.erase(2);  // 删除所有值为 2 的元素，us 变为 {1, 3, 4}
   ```

4. `erase(first, last)`：删除指定范围的元素。

### (5) 查找函数

1. `iterator find(const T& value)`：查找某元素在 `unordered_multiset` 中的位置，返回一个指向该元素的迭代器。如果找不到会返回 `end()`。

   ```cpp
   std::unordered_multiset<int> us = {1, 2, 3, 4};
   auto it = us.find(1);  // 返回指向键1的迭代器
   if (it != us.end()) {
       std::cout << *it << std::endl;  // 输出: 1
   }
   ```

2. `std::pair<const_iterator, const_iterator> equal_range(const key_type& __x) const`：返回一个迭代器对，表示所有具有等于 `x` 的键的元素的范围。由于 `unordered_multiset` 允许重复元素，这个范围包含所有相同的键。

3. `size_t count(const T& value)`：返回指定元素在 `unordered_multiset` 中出现的次数。对于 `unordered_multiset`，返回的次数可能大于 1，因为允许重复元素。

4. `bool contains(const T& value) const`（C++20）：检查是否包含该元素。

### (6) 遍历函数

1. `iterator begin()`：返回指向 `std::unordered_multiset` 第一个元素的迭代器。

2. `iterator end()`：返回指向 `std::unordered_multiset` 最后一个元素之后位置的迭代器。

3. 使用基于范围的 `for` 循环（C++11 及以上）: 直接遍历 `std::unordered_multiset` 中的每个元素。

   ```cpp
   std::unordered_multiset<int> us = {1, 2, 2, 3};
   for (const int& num : us) {
       std::cout << num << " ";  // 输出: 1 2 2 3
   }
   ```

4. 使用传统的 `for` 循环（基于迭代器）: 使用迭代器来遍历 `std::unordered_multiset`。

   ```cpp
   std::unordered_multiset<int> us = {1, 2, 2, 3};
   for (auto it = us.begin(); it != us.end(); ++it) {
       std::cout << *it << " ";  // 输出: 1 2 2 3
   }
   ```

### (7) 哈希相关函数

1. `size_type max_bucket_count() const noexcept`：返回 `unordered_multiset` 中可用的最大哈希桶数量。

2. `float max_load_factor() const noexcept`：返回或设置哈希表的最大负载因子。

3. `void max_load_factor(float __z)`：设置最大负载因子。

4. `size_t bucket_count() const`：返回当前 `unordered_multiset` 的桶数量。

5. `size_t bucket_size(size_t n) const`：返回指定桶的元素数量。

6. `size_t bucket(const key_type& key) const`：返回给定键所在的桶的索引。

7. `float load_factor() const`：返回当前哈希表的负载因子。

8. `void rehash(size_t n)`：用来调整哈希表中的桶数量。

### (8) 其他函数

1. `swap(unordered_multiset& other)`：交换当前 `unordered_multiset` 和另一个 `unordered_multiset` 的内容。

2. `std::allocator<T> get_allocator() const`：返回容器所使用的分配器（allocator）。

3. `void merge(unordered_multiset& other)`：合并另一个 `unordered_multiset` 中不重复的元素。

4. `key_equal_type key_eq() const`：返回一个比较器，用于判断两个键是否相等。
