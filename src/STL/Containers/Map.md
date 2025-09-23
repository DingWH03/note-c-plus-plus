# std::map

`std::map` 是 C++ 标准库中的一个关联容器，它用于存储一组键值对（`key-value`），并根据键进行自动排序。每个键在 `std::map` 中是唯一的，因此不允许存储重复的键。`std::map` 底层通常使用红黑树结构实现，时间复杂度通常为 O(log n) 用于插入、查找和删除操作。

## 1. 引入

```cpp
#include <map>
```

## 2. 存储方式

`std::map` 是基于平衡二叉树（通常是红黑树）实现的容器。与 `std::set` 类似，`std::map` 也具有自动排序的特性，元素会根据键（key）进行升序排序（默认情况下）。与 `std::set` 的区别是，`std::map` 存储的是键值对，每个键对应一个值，且键是唯一的。

在 `std::map` 中，元素存储为节点，每个节点包含一个键值对和指向父节点、左子节点和右子节点的指针。通过这种方式，`std::map` 保证了所有键的排序特性，并支持高效的查找、插入和删除操作。

* **插入**：插入一个新元素的时间复杂度为 O(log n)，并且如果插入的键已经存在，新的值会覆盖原来的值。
* **查找**：在 `std::map` 中查找元素的时间复杂度为 O(log n)，这是因为它依赖于平衡二叉树的结构。
* **删除**：删除操作也有 O(log n) 的时间复杂度，因为删除节点时需要保持树的平衡。

`std::map` 不允许重复的键，因此每个键只能对应一个值。如果尝试插入一个已存在的键，插入操作会被忽略。

## 3. 方法

### (1) 构造方法

1. `map()`: 创建一个空的 `std::map`，默认构造函数。

   ```cpp
   // std::map<key_type, value_type> myMap;
   std::map<int, std::string> m;
   std::map<int, std::string> m = {{1, "one"}, {2, "two"}, {3, "three"}};
   ```

2. `map(const map&)`: 复制构造函数，创建一个与另一个 `std::map` 完全相同的副本。

   ```cpp
   std::map<int, std::string> m1 = {{1, "one"}, {2, "two"}};
   std::map<int, std::string> m2(m1);  // m2 是 m1 的一个复制版本
   ```

3. `map(begin, end)`: 使用另一个容器或迭代器区间中的元素初始化 `std::map`。

   ```cpp
   std::vector<std::pair<int, std::string>> vec = {{1, "one"}, {2, "two"}};
   std::map<int, std::string> m(vec.begin(), vec.end());  // 复制 vector 的元素到 map
   ```

4. `std::map<Key, T, Compare>`: 使用自定义的比较器来进行键的排序。

### (2) 大小函数

1. `size_t size() const`: 返回 `std::map` 中元素的个数。

   ```cpp
   std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
   std::cout << m.size();  // 输出 2
   ```

2. `bool empty() const`: 检查 `std::map` 是否为空，若为空返回 `true`，否则返回 `false`。

   ```cpp
   std::map<int, std::string> m;
   if (m.empty()) {
       std::cout << "Map is empty." << std::endl;
   }
   ```

3. `size_t max_size() const`: 返回最大可允许的 `map` 元素数量。

### (3) 增加函数

1. `std::pair<iterator, bool> insert(const value_type& value)`: 插入一个元素到 `std::map` 中，返回一个 `pair`，其中 `first` 是指向插入元素所在位置的迭代器，`second` 表示插入是否成功（如果键已存在，则插入失败）。

   ```cpp
   std::map<int, std::string> m;
   m.insert({1, "one"});
   m.insert({2, "two"});
   ```

2. `std::pair<iterator, bool> emplace(_Args&&... args)`: 就地构建并插入元素。

3. `iterator emplace_hint(const_iterator hint, _Args&&... args)`: 就地构建并插入元素，且指定插入位置。

4. `std::pair<iterator, bool> try_emplace(const key_type& key, Args&&... args)`: 是 C++17 引入的一种插入方式。它尝试插入一个元素，如果给定的键已经存在，则不会插入新元素，并且不会更改已有元素的值。它还允许通过参数传递构造函数的参数，避免不必要的拷贝或移动。

    ```cpp
    std::map<int, std::string> m;
    auto result = m.try_emplace(1, "one");
    if (result.second) {
        std::cout << "Element inserted: " << result.first->second << std::endl;
    } else {
        std::cout << "Element with key " << result.first->first << " already exists." << std::endl;
    }

    auto result2 = m.try_emplace(1, "uno");  // 不会插入新元素，因为键1已存在
    std::cout << "Element: " << result2.first->second << std::endl;  // 输出: "one"
    ```

5. `std::pair<iterator, bool> insert_or_assign(const key_type& key, const mapped_type& obj)`和`std::pair<iterator, bool> insert_or_assign(const key_type& key, mapped_type&& obj)`: 是 C++17 引入的另一种插入方式，它的作用是：如果键不存在，就插入该键值对；如果键已经存在，则更新其对应的值。

6. `operator[]` 插入方式(`mapped_type& operator[](const key_type& key)`或`mapped_type& operator[](key_type&& key);`): 最常见的插入方式。它会检查 std::map 中是否存在给定的键。如果键存在，它返回该键对应的值；如果键不存在，它会插入该键并且初始化对应的值为 mapped_type 的默认值（对于非内置类型，是通过默认构造函数进行初始化）。同样的，这样的方式还可以用来获取键对应的值。

    ```cpp
    std::map<int, std::string> m;
    m[1] = "one";  // 插入新元素
    std::cout << m[1] << std::endl;  // 输出: one

    m[1] = "uno";  // 更新已有的元素
    std::cout << m[1] << std::endl;  // 输出: uno

    m[2];  // 键2不存在，会插入键2，并将对应的值初始化为默认值 "" (空字符串)
    std::cout << m[2] << std::endl;  // 输出: (空字符串)
    ```

### (4) 删除元素

1. `void clear()`: 删除 `std::map` 中的所有元素，使容器变为空。

   ```cpp
   std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
   m.clear();  // 删除所有元素，m 变为空 {}
   ```

2. `void erase(iterator pos)`: 删除指定位置 `pos` 处的元素。

   ```cpp
   std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
   m.erase(m.begin());  // 删除第一个元素
   ```

3. `size_t erase(const key_type& key)`: 删除指定键，返回删除个数。

   ```cpp
   std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
   m.erase(1);  // 删除键为1的元素
   ```

4. `erase(first, last)`: 删除迭代器区间。

### (5) 查找函数

1. `iterator find(const key_type& key)`: 查找某个键在 `map` 中的位置，返回一个指向该元素的迭代器，如果找不到会返回 `end()`。

   ```cpp
   std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
   auto it = m.find(1);  // 返回指向键1的迭代器
   std::cout << it->second;  // 输出 "one"
   ```

2. `std::pair<const_iterator, const_iterator> equal_range(const key_type& key) const`: 返回一对迭代器，表示所有具有等于键 `key` 的元素的范围。在 `std::map` 中，由于键是唯一的，因此这个范围包含一个元素。

   ```cpp
   std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
   auto range = m.equal_range(1);
   std::cout << range.first->second << std::endl;  // 输出 "one"
   ```

3. `size_t count(const key_type& key)`: 返回容器中是否包含指定键，`map` 中每个键最多出现一次，因此返回值要么是 `0`，要么是 `1`。

4. `iterator lower_bound(const key_type& key)`: 返回第一个不小于 `key` 的迭代器。

5. `iterator upper_bound(const key_type& key)`: 返回第一个大于 `key` 的迭代器。

6. `bool contains(const key_type& key) const`（C++20）：检查是否包含该键。

### (6) 遍历函数

1. `iterator begin()`: 返回指向 `std::map` 第一个元素的迭代器。

   ```cpp
   std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
   auto it = m.begin();  // 返回指向第一个元素的迭代器
   std::cout << it->first << ": " << it->second << std::endl;  // 输出 1: one
   ```

2. `iterator end()`: 返回指向 `std::map` 最后一个元素之后位置的迭代器。

3. `reverse_iterator rbegin()`: 返回指向 `std::map` 最后一个元素的反向迭代器。

4. `reverse_iterator rend()`: 返回指向 `std::map` 第一个元素之前位置的反向迭代器。

5. 使用基于范围的 `for` 循环（C++11 及以上）: 直接遍历 `std::map` 中的每个元素。

   ```cpp
   std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
   for (const auto& pair : m) {
       std::cout << pair.first << ": " << pair.second << " ";  // 输出: 1: one 2: two
   }
   ```

6. 使用传统的 `for` 循环（基于迭代器）: 使用迭代器来遍历 `std::map`。

   ```cpp
   std::map<int, std::string> m = {{1, "one"}, {2, "two"}};
   for (auto it = m.begin(); it != m.end(); ++it) {
       std::cout << it->first << ": " << it->second << " ";  // 输出: 1: one 2: two
   }
   ```

### (7) 其他函数

1. `swap(map& other)`: 交换当前 `std::map` 和另一个 `std::map` 的内容。

2. `std::allocator<T> get_allocator() const`: 返回容器所使用的分配器（allocator）。

3. `std::map<Key, T, Compare> merge(map& other)`: 将另一个`map`容器合并过来，保持有序性。
