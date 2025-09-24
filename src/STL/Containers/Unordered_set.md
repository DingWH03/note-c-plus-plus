
# std::unordered\_set

`std::unordered_set` 是一个无序的关联容器，它与 `std::set` 的主要区别在于元素的存储顺序。`std::set` 按照元素的升序排列，而 `std::unordered_set` 不保持任何顺序，它依赖于哈希表实现，因此元素的存储位置是由哈希函数决定的。`std::unordered_set` 提供了高效的查找、插入和删除操作，底层通常使用哈希表，查找、插入和删除操作的平均时间复杂度为 O(1)，但最坏情况下为 O(n)。

`unordered_map`是一个模板类，其定义如下：

```cpp
std::unordered_set<Key, Hash = std::hash<Key>, Pred = std::equal_to<Key>, Alloc = std::allocator<Key>>
```

* Key 是存储在 `unordered_set` 中的元素类型。
* Hash 是一个函数或函数对象，用于生成元素的哈希值，默认为 `std::hash<Key>`。
* Pred 是一个二元谓词，用于比较两个元素是否相等，默认为 `std::equal_to<Key>`。
* Alloc 是分配器类型，用于管理内存分配，默认为 `std::allocator<Key>`。

## 1. 引入

```C++
#include <unordered_set>
```

## 2. 存储方式

`std::unordered_set` 使用哈希表（hash table）来存储元素。每个元素的存储位置由哈希函数计算出来，因此它不保证元素的顺序。哈希表允许非常高效的查找、插入和删除操作，通常时间复杂度为 O(1)，但在哈希冲突的情况下，最坏情况的时间复杂度可能退化为 O(n)。

* **插入**：插入一个元素时，哈希函数会计算该元素的哈希值，并将元素存储在哈希表中。
* **查找**：查找操作的时间复杂度为 O(1)，但是哈希冲突可能导致时间复杂度退化为 O(n)。
* **删除**：删除操作的时间复杂度通常为 O(1)，但最坏情况下可能退化为 O(n)。

`std::unordered_set` 与 `std::set` 最大的区别是，它使用哈希表来存储元素，因此没有顺序性，无法进行按顺序的遍历。

## 3. 方法

### (1)构造方法

1. `unordered_set()`: 创建一个空的 `std::unordered_set`，默认构造函数。

   ```cpp
   std::unordered_set<int> us;
   std::unordered_set<int> us = {1, 2, 3, 4, 5};
   ```

2. `unordered_set(const unordered_set&)`: 复制构造函数，创建一个与另一个 `std::unordered_set` 完全相同的副本。

   ```cpp
   std::unordered_set<int> us1 = {1, 2, 3, 4, 5};
   std::unordered_set<int> us2(us1);  // us2 是 us1 的一个复制版本
   ```

3. `unordered_set(begin, end)`: 使用另一个容器或迭代器区间中的元素初始化 `std::unordered_set`。

   ```cpp
   std::vector<int> vec = {1, 2, 3, 4, 5};
   std::unordered_set<int> us(vec.begin(), vec.end());  // 复制 vector 的元素到 unordered_set
   ```

#### (2) 大小函数

1. `size_t size() const`: 返回 `std::unordered_set` 中元素的个数。

   ```cpp
   std::unordered_set<int> us = {1, 2, 3, 4};
   std::cout << us.size();  // 输出 4
   ```

2. `bool empty() const`: 检查 `std::unordered_set` 是否为空，若为空返回 `true`，否则返回 `false`。

   ```cpp
   std::unordered_set<int> us;
   if (us.empty()) {
       std::cout << "Unordered Set is empty." << std::endl;
   }
   ```

3. `size_t max_size() const`: 返回最大可允许的 `unordered_set` 元素数量。

4. `void reserve(size_t n)`: 用于预先分配哈希表的桶数量。这可以有效减少动态扩容的开销，特别是在你知道将要插入大量元素时。

#### (3) 增加函数

1. `std::pair<iterator, bool> insert(const T& value)`: 插入元素到 `unordered_set` 中，返回一个 `pair`，其中 `first` 是指向插入元素所在位置的迭代器，`second` 是插入是否成功（如果存在重复元素，则插入失败）。

   ```cpp
   us.insert({1, 2, 3});                     // initializer_list
   us.insert(us.begin(), 42);               // 带 hint 的插入
   us.insert(vec.begin(), vec.end());      // 区间插入
   ```

2. `std::pair<iterator, bool> emplace(_Args&&... args)`: 就地构建并插入元素。

3. `iterator emplace_hint(const_iterator hint, _Args&&... args)`: 就地构建并插入元素，且指定插入位置，但是如果指定位置错误会发生重排序，最坏情况下可能会降低插入的效率。在哈希索引的前提下是无序的，因此该插入操作等效于`insert`。

#### (4) 删除元素

1. `void clear()`: 删除 `std::unordered_set` 中的所有元素，使容器变为空。

   ```cpp
   std::unordered_set<int> us = {1, 2, 3, 4};
   us.clear();  // 删除所有元素，us 变为空 {}
   ```

2. `void erase(iterator pos)`: 删除指定位置 `pos` 处的元素。

   ```cpp
   std::unordered_set<int> us = {1, 2, 3, 4};
   us.erase(std::next(us.begin(), 2));  // 删除索引为2的元素，us 变为 {1, 2, 4}
   ```

3. `size_t erase(const key_type& key)`: 删除指定键，返回删除个数。

4. `erase(first, last)`: 删除迭代器区间。

#### (5) 查找函数

1. `iterator find(const T& value)`: 查找某元素在 `unordered_set` 中的位置，返回一个指向该元素的迭代器，如果找不到会返回 `end()`。

   ```cpp
   std::unordered_set<int> us = {1, 2, 3, 4};
   auto it = us.find(1);  // 返回指向键1的迭代器
   if (it != us.end()) {
       std::cout << *it << std::endl;  // 输出: 1
   }
   ```

2. `std::pair<const_iterator, const_iterator> equal_range(const key_type& __x) const`: 返回一对迭代器，表示所有具有等于k的键的元素的范围。由于set包含唯一元素，因此下界将是元素本身，上限将指向键k之后的下一个元素。如果没有与键K匹配的元素，则根据容器的内部比较对象(key_comp)，返回的范围的长度为0，两个迭代器均指向大于k的第一个元素。如果键超过了set容器中的最大元素，它将返回一个指向set容器中最后一个元素的迭代器。

3. `size_t count(const T& value)`: 返回元素是否存在（0 或 1）。

4. `bool contains(const T& value) const`（C++20）：检查是否包含该元素。

#### (6) 遍历函数

1. `iterator begin()`: 返回指向 `std::unordered_set` 第一个元素的迭代器。

2. `iterator end()`: 返回指向 `std::unordered_set` 最后一个元素之后位置的迭代器。

3. 使用基于范围的 `for` 循环（C++11 及以上）: 直接遍历 `std::unordered_set` 中的每个元素。

   ```cpp
   std::unordered_set<int> us = {1, 2, 3};
   for (const int& num : us) {
       std::cout << num << " ";  // 输出: 1 2 3
   }
   ```

4. 使用传统的 `for` 循环（基于迭代器）: 使用迭代器来遍历 `std::unordered_set`。

   ```cpp
   std::unordered_set<int> us = {1, 2, 3};
   for (auto it = us.begin(); it != us.end(); ++it) {
       std::cout << *it << " ";  // 输出: 1 2 3
   }
   ```

### (7)哈希相关函数

1. `size_type max_bucket_count() const noexcept`: max_bucket_count() 返回 std::unordered_set 中可用的最大哈希桶数量。哈希表是由一组桶（bucket）构成的，每个桶存储一定数量的元素。当元素的数量增加时，std::unordered_set 可能会自动扩展桶的数量，以减少哈希冲突。max_bucket_count() 允许你查看哈希表最多能支持多少个桶。

2. `float max_load_factor() const noexcept`: max_load_factor() 返回或设置哈希表的最大负载因子。负载因子是哈希表中元素的数量与桶数量之比。它决定了哈希表何时扩展——当负载因子超过最大值时，std::unordered_set 会增加桶的数量，减少哈希冲突，从而提高性能。

    * 负载因子是指：负载因子 = 元素个数 / 桶的数量
    * 最大负载因子是哈希表在扩展之前允许的最大负载因子。负载因子越高，意味着桶的数量相对较少，这可能导致更多的哈希冲突，降低查找效率。

3. `void max_load_factor(float __z)`: 设置最大负载因子。

4. `size_t bucket_count() const`: 返回当前 unordered_set 的桶数量。哈希表的桶数量与容器中元素的数量以及负载因子相关，通常哈希表会在元素数量增加时自动扩展桶的数量。

5. `size_t bucket_size(size_t n) const`: 返回指定桶的元素数量。哈希表中的每个桶可能包含多个元素，特别是当哈希冲突发生时，多个元素可能会存储在同一个桶中。

6. `size_t bucket(const key_type& key) const`: 返回给定键所在的桶的索引。这个函数对于理解元素是如何分布在哈希表中的非常有用。

7. `float load_factor() const`: 返回当前哈希表的负载因子，即元素的数量与桶数量的比值。负载因子越高，哈希冲突的可能性越大，因此负载因子对于性能非常重要。

8. `void rehash(size_t n)`: 用来调整哈希表中的桶数量。它的作用是根据新的元素数量来调整桶的数量，避免哈希冲突过多。rehash() 会导致重新分配桶并重新哈希元素。n 是希望容器能够容纳的元素数量。

#### (8) 其他函数

1. `swap(unordered_set& other)`: 交换当前 `std::unordered_set` 和另一个 `std::unordered_set` 的内容。

2. `std::allocator<T> get_allocator() const`: 返回容器所使用的分配器（allocator）。

3. `void merge(unordered_set& other)`: `merge` 会将另一个 unordered_set 中不与当前容器重复的元素“移动”过来。

    ```c++
    std::unordered_set<int> s1 = {1, 3, 5};
    std::unordered_set<int> s2 = {2, 4, 6};
    s1.merge(s2);  // s1 合并 s2 后，变为 {1, 2, 3, 4, 5, 6}
    ```

4. `key_equal_type key_eq() const`： 返回一个比较器，用于判断两个键是否相等

    ```cpp
    std::unordered_set<Person, PersonHash> people;

    Person p1 = {"Alice", 30};
    Person p2 = {"Bob", 25};
    people.insert(p1);
    people.insert(p2);

    // 使用 key_eq 获取比较函数
    auto eq = people.key_eq();

    // 使用 key_eq 函数比较两个键
    std::cout << "Are p1 and p2 equal? " << std::boolalpha << eq(p1, p2) << std::endl;
    ```
