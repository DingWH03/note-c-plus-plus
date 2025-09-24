# std::unordered\_multimap

`std::unordered_multimap` 是 C++ 标准库中的一个无序关联容器，存储一组键值对（`key-value`）。与 `std::unordered_map` 类似，`std::unordered_multimap` 也使用 **哈希表** 来组织元素，但不同的是，它允许 **键重复**。
由于其底层实现为哈希表，元素顺序并不固定，完全由哈希函数决定。平均查找、插入、删除的时间复杂度为 O(1)，最坏情况下会退化为 O(n)。

`std::unordered_multimap` 的定义如下：

```cpp
template<
    class Key,
    class T,
    class Hash = std::hash<Key>,
    class KeyEqual = std::equal_to<Key>,
    class Allocator = std::allocator<std::pair<const Key, T>>
> class unordered_multimap; // since C++11
```

```cpp
namespace pmr {
    template<
        class Key,
        class T,
        class Hash = std::hash<Key>,
        class Pred = std::equal_to<Key>
    > using unordered_multimap =
          std::unordered_multimap<Key, T, Hash, Pred,
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

* `std::unordered_multimap` 底层使用 **哈希表** 来存储元素，每个元素（键值对）会根据其 **键** 被哈希到一个桶中。
* **键的唯一性**：与 `std::unordered_map` 不同，`std::unordered_multimap` 允许 **多个相同的键** 存在，每个键对应多个值。相同键的元素会存储在同一个桶中。
* **插入与查找**：所有操作（查找、插入、删除）平均时间复杂度为 **O(1)**，但在哈希冲突严重时，最坏情况可能退化为 **O(n)**。

## 3. 方法

### (1) 构造方法

1. `unordered_multimap()`: 默认构造函数

   ```cpp
   std::unordered_multimap<int, std::string> m;
   std::unordered_multimap<int, std::string> m = {{1, "one"}, {2, "two"}};
   ```

2. `unordered_multimap(const unordered_multimap&)`: 复制构造函数

   ```cpp
   std::unordered_multimap<int, std::string> m1 = {{1, "one"}, {2, "two"}};
   std::unordered_multimap<int, std::string> m2(m1);
   ```

3. `unordered_multimap(begin, end)`: 使用另一个容器或迭代器的区间中的元素初始化

   ```cpp
   std::vector<std::pair<int, std::string>> vec = {{1, "one"}, {2, "two"}};
   std::unordered_multimap<int, std::string> m(vec.begin(), vec.end());
   ```

4. `std::unordered_multimap<key, T, Hash>`: 使用自定义的哈希函数

   ```cpp
   struct MyHash {
       size_t operator()(int x) const { return x % 10; }
   };
   std::unordered_multimap<int, std::string, MyHash> m;
   ```

### (2) 大小函数

1. `size_t size() const`: 返回元素个数
2. `bool empty() const`: 是否为空
3. `size_t max_size() const`: 返回最大可允许元素数量
4. `void reserve(size_type __n)`: 预分配桶的数量，减少扩容开销

### (3) 插入函数

1. `std::pair<iterator, bool> insert(const value_type& __x)`: 插入一个键值对，返回保存指向添加键值对位置的迭代器和是否添加成功的标志的pair

   ```cpp
   m.insert({1, "one"});
   m.insert(std::make_pair(2, "two"));
   ```

2. `std::pair<iterator, bool> emplace(Key&& key, T&& obj)`: 如果键 `key` 不存在，就直接插入一个新元素。

   ```cpp
   m.emplace(3, "three");
   ```

3. `std::pair<iterator, bool> try_emplace(const Key& key, Args&&... args)`: 只有在 `key` 不存在时，才插入新元素（避免额外的拷贝）。

   ```cpp
   m.try_emplace(1, "one");
   ```

4. `std::pair<iterator, bool> insert_or_assign(const Key& key, T&& obj)`: 如果 `key` 存在，更新对应的值；如果 `key` 不存在，插入一个新的键值对。

   ```cpp
   m.insert_or_assign(1, "uno");
   ```

5. `operator[](const Key& key)`: 如果 `key` 存在，返回对应的值；如果 `key` 不存在，插入一个新的键值对，值为默认构造的类型 T。

   ```cpp
   m[4] = "four";   // 插入
   m[1] = "uno";    // 更新
   ```

6. `at(const Key& key)`: 如果 `key` 已存在，返回对应的值；如果 `key` 不存在，抛出 `std::out_of_range`。

   ```cpp
   std::cout << m.at(1);  // 输出值
   ```

### (4) 删除元素

1. `void clear()`: 清空所有元素

2. `iterator erase(iterator pos)`: 删除迭代器位置的元素

3. `size_t erase(const key_type& key)`: 删除指定键的所有元素，返回删除的数量

   ```cpp
   m.erase(1);  // 删除键为1的所有元素
   ```

4. `iterator erase(first, last)`: 删除指定迭代器区间的元素

### (5) 查找函数

1. `iterator find(const key_type& key)`: 查找指定键的第一个匹配元素，返回一个指向该元素的迭代器，如果找不到返回 `end()`。

   ```cpp
   auto it = m.find(2);
   if (it != m.end()) {
       std::cout << it->second << std::endl;
   }
   ```

2. `std::pair<const_iterator, const_iterator> equal_range(const key_type& key) const`: 返回一对迭代器，表示所有具有等于键 `key` 的元素的范围。

   ```cpp
   auto range = m.equal_range(2);
   for (auto it = range.first; it != range.second; ++it) {
       std::cout << it->second << std::endl;
   }
   ```

3. `size_t count(const key_type& key)`: 返回容器中指定键的元素数量（对于 `unordered_multimap`，返回值可能大于 1）。

   ```cpp
   size_t n = m.count(2);  // 返回键2的数量
   ```

4. `bool contains(const key_type& key) const`（C++20）：检查是否包含指定键。

   ```cpp
   if (m.contains(2)) {
       std::cout << "Key 2 exists!" << std::endl;
   }
   ```

### (6) 遍历函数

1. `iterator begin()`: 返回指向容器第一个元素的迭代器

   ```cpp
   auto it = m.begin();
   std::cout << it->first << " -> " << it->second << std::endl;
   ```

2. `iterator end()`: 返回指向容器最后一个元素之后位置的迭代器

3. `reverse_iterator rbegin()`: 返回指向容器最后一个元素的反向迭代器

4. `reverse_iterator rend()`: 返回指向容器第一个元素之前位置的反向迭代器

5. 使用基于范围的 `for` 循环：

   ```cpp
   for (const auto& [key, value] : m) {
       std::cout << key << " -> " << value << std::endl;
   }
   ```

6. 使用传统的迭代器遍历：

   ```cpp
   for (auto it = m.begin(); it != m.end(); ++it) {
       std::cout << it->first << " -> " << it->second << std::endl;
   }
   ```

### (7) 哈希相关函数

1. `size_t bucket_count() const`: 返回桶的数量
2. `size_t bucket_size(size_t n) const`: 返回指定桶的元素数量
3. `size_t bucket(const key_type& key) const`: 返回给定键所落入的桶的索引
4. `float load_factor() const`: 返回当前负载因子（元素数 / 桶数）
5. `void rehash(size_t n)`: 强制调整桶的数量，避免哈希冲突
6. `void reserve(size_t n)`: 确保容器至少可以容纳 `n` 个元素

### (8) 其他函数

1. `swap(unordered_multimap& other)`: 交换当前 `unordered_multimap` 和另一个容器的内容
2. `std::allocator<T> get_allocator() const`: 返回容器的分配器
3. `void merge(unordered_multimap& other)`: 将另一个 `unordered_multimap` 中不重复的元素“移动”过来（C++17）
4. `hasher hash_function() const`: 返回当前使用的哈希函数
5. `key_equal_type key_eq() const`: 返回用于判断键是否相等的谓词
