# std::unordered\_map

`std::unordered_map` 是 C++ 标准库中的一个无序关联容器，用于存储键值对（`key-value`）。与 `std::map` 不同的是，`unordered_map` 内部使用 **哈希表** 来组织元素，因此元素的顺序不固定，完全由哈希函数决定。
它能在平均 **O(1)** 时间内完成查找、插入和删除操作（最坏情况下退化为 O(n)）。

`unordered_map` 的定义如下：

```cpp
template<
    class Key,
    class T,
    class Hash = std::hash<Key>,
    class KeyEqual = std::equal_to<Key>,
    class Allocator = std::allocator<std::pair<const Key, T>>
> class unordered_map; // since C++11
```

```cpp
namespace pmr {
    template<
        class Key,
        class T,
        class Hash = std::hash<Key>,
        class KeyEqual = std::equal_to<Key>
    > using unordered_map =
          std::unordered_map<Key, T, Hash, KeyEqual,
              std::pmr::polymorphic_allocator<std::pair<const Key, T>>>;
} // since C++17
```

* Key：键的类型（必须支持哈希和比较相等操作）。
* T：映射值的类型。
* Hash：哈希函数对象，默认为 `std::hash<Key>`。
* Pred：相等比较函数对象，默认为 `std::equal_to<Key>`。
* Alloc：分配器类型，默认为 `std::allocator<std::pair<const Key, T>>`。

## 1. 引入

```cpp
#include <unordered_map>
```

## 2. 存储方式

* `std::map` 使用 **红黑树**，有序，查找/插入/删除是 O(log n)。
* `std::unordered_map` 使用 **哈希表**，无序，查找/插入/删除平均是 O(1)，但在哈希冲突严重时，最坏情况可能退化为 O(n)。

每个键值对通过哈希函数映射到某个 **桶（bucket）** 中，冲突时使用链表或开链方式解决。
键必须唯一，如果插入相同的键，新值会覆盖旧值。

## 3. 方法

### (1) 构造方法

1. `unordered_map()`: 默认构造函数

   ```cpp
   std::unordered_map<int, std::string> m;
   std::unordered_map<int, std::string> m = {{1, "one"}, {2, "two"}};
   ```

2. `unordered_map(const unordered_map&)`: 复制构造函数

   ```cpp
   std::unordered_map<int, std::string> m1 = {{1, "one"}, {2, "two"}};
   std::unordered_map<int, std::string> m2(m1);
   ```

3. `unordered_map(begin, end)`: 使用另一个容器或迭代器的区间中的元素初始化

   ```cpp
   std::vector<std::pair<int, std::string>> vec = {{1, "one"}, {2, "two"}};
   std::unordered_map<int, std::string> m(vec.begin(), vec.end());
   ```

4. `std::unordered_map<key, T, Compare>`: 使用自定义的比较器来进行键的排序

   ```cpp
   struct MyHash {
       size_t operator()(int x) const { return x % 10; }
   };
   std::unordered_map<int, std::string, MyHash> m;
   ```

### (2) 大小函数

1. `size_t size() const`： 返回元素个数
2. `bool empty() const`： 是否为空
3. `size_t max_size() const`： 返回最大可允许元素数量
4. `void reserve(size_type __n)`： 预分配桶的数量，减少扩容开销

### (3) 增加函数

1. `std::pair<iterator, bool> insert(const value_type& __x)`: 插入一个键值对，返回保存指向添加键值对位置的迭代器和是否添加成功的标志的pair

   ```cpp
   m.insert({1, "one"});
   m.insert(std::make_pair(2, "two"));
   ```

2. `std::pair<iterator, bool> emplace(Key&& key, T&& obj)`: 如果键 key 不存在，就直接插入一个新元素。

   ```cpp
   m.emplace(3, "three");
   ```

3. `std::pair<iterator, bool> try_emplace(const Key& key, Args&&... args)`: 只有在 key 不存在时，才插入新元素（避免额外的拷贝）。

   ```cpp
   m.try_emplace(1, "one");
   ```

4. `std::pair<iterator, bool> insert_or_assign(const Key& key, T&& obj)`

   ```cpp
   m.insert_or_assign(1, "uno");
   ```

5. `operator[](const Key& key)`: 如果 key 存在，返回对应的值；如果 key 不存在，插入一个新的键值对，值为默认构造的类型 T。

   ```cpp
   m[4] = "four";   // 插入
   m[1] = "uno";    // 更新
   ```

6. `at(const Key& key)`：如果 key 已存在，更新对应的值；如果 key 不存在，插入一个新的键值对。

   ```cpp
   std::cout << m.at(1);
   ```

### (4) 删除元素

1. `void clear()`：清空所有元素
2. `iterator erase(iterator pos)`：删除迭代器位置的元素
3. `iterator erase(const key_type& key)`：删除指定键，返回删除个数
4. `iterator erase(first, last)`：删除范围

### (5) 查找函数

1. `iterator find(const key_type& key)`: 查找某个键在 `unordered_map` 中的位置，返回一个指向该元素的迭代器，如果找不到会返回 `end()`。

   ```cpp
   std::unordered_map<int, std::string> m = {{1, "one"}, {2, "two"}};
   auto it = m.find(1);  // 返回指向键1的迭代器
   std::cout << it->second;  // 输出 "one"
   ```

2. `std::pair<const_iterator, const_iterator> equal_range(const key_type& key) const`: 返回一对迭代器，表示所有具有等于键 `key` 的元素的范围。在 `std::unordered_map` 中，由于键是唯一的，因此这个范围包含一个元素。

   ```cpp
   std::unordered_map<int, std::string> m = {{1, "one"}, {2, "two"}};
   auto range = m.equal_range(1);
   std::cout << range.first->second << std::endl;  // 输出 "one"
   ```

3. `size_t count(const key_type& key)`: 返回容器中是否包含指定键，`unordered_map` 中每个键最多出现一次，因此返回值要么是 `0`，要么是 `1`。

4. `iterator lower_bound(const key_type& key)`: 返回第一个不小于 `key` 的迭代器。

5. `iterator upper_bound(const key_type& key)`: 返回第一个大于 `key` 的迭代器。

6. `bool contains(const key_type& key) const`（C++20）：检查是否包含该键。

### (6) 遍历函数

1. `iterator begin()`: 返回指向 `std::unordered_map` 第一个元素的迭代器。

   ```cpp
   std::unordered_map<int, std::string> m = {{1, "one"}, {2, "two"}};
   auto it = m.begin();  // 返回指向第一个元素的迭代器
   std::cout << it->first << ": " << it->second << std::endl;  // 输出 1: one
   ```

2. `iterator end()`: 返回指向 `std::unordered_map` 最后一个元素之后位置的迭代器。

3. `reverse_iterator rbegin()`: 返回指向 `std::unordered_map` 最后一个元素的反向迭代器。

4. `reverse_iterator rend()`: 返回指向 `std::unordered_map` 第一个元素之前位置的反向迭代器。

5. 使用基于范围的 `for` 循环（C++11 及以上）: 直接遍历 `std::unordered_map` 中的每个元素。

   ```cpp
   std::unordered_map<int, std::string> m = {{1, "one"}, {2, "two"}};
   for (const auto& pair : m) {
       std::cout << pair.first << ": " << pair.second << " ";  // 输出: 1: one 2: two
   }
   ```

6. 使用传统的 `for` 循环（基于迭代器）: 使用迭代器来遍历 `std::unordered_map`。

   ```cpp
   std::unordered_map<int, std::string> m = {{1, "one"}, {2, "two"}};
   for (auto it = m.begin(); it != m.end(); ++it) {
       std::cout << it->first << ": " << it->second << " ";  // 输出: 1: one 2: two
   }
   ```

### (7) 哈希相关函数

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

### (8) 其他函数

1. `swap(unordered_map& other)`: 交换当前 `std::unordered_map` 和另一个 `std::unordered_map` 的内容。

2. `std::allocator<T> get_allocator() const`: 返回容器所使用的分配器（allocator）。

3. `void merge(unordered_map& other)`: `merge` 会将另一个 unordered_map 中不与当前容器重复的元素“移动”过来。

    ```c++
    std::unordered_map<int> s1 = {1, 3, 5};
    std::unordered_map<int> s2 = {2, 4, 6};
    s1.merge(s2);  // s1 合并 s2 后，变为 {1, 2, 3, 4, 5, 6}
    ```

4. `key_equal_type key_eq() const`： 返回一个比较器，用于判断两个键是否相等

    ```cpp
    std::unordered_map<Person, PersonHash> people;

    Person p1 = {"Alice", 30};
    Person p2 = {"Bob", 25};
    people.insert(p1);
    people.insert(p2);

    // 使用 key_eq 获取比较函数
    auto eq = people.key_eq();

    // 使用 key_eq 函数比较两个键
    std::cout << "Are p1 and p2 equal? " << std::boolalpha << eq(p1, p2) << std::endl;
    ```

5. `hasher hash_function() const`：返回当前使用的哈希函数对象
