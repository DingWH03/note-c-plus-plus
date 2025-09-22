
# std::unordered\_multimap

`std::unordered_multimap` 是一个基于哈希表实现的容器，用于存储键值对，并允许一个键对应多个值。它不保证元素的顺序，适用于需要存储多个相同键值对的场景。`std::unordered_multimap` 提供了高效的查找、插入和删除操作，通常为 O(1)，但无法保证元素的顺序。

## 1. 引入

```C++
#include <unordered_map>
```

## 2. 存储方式

`std::unordered_multimap` 使用哈希表来存储键值对。每个元素由一对键值（key-value）组成，存储位置通过哈希函数计算得出。与 `std::unordered_map` 不同，`std::unordered_multimap` 允许多个相同的键出现，因此可以存储多个具有相同键的值。

* **插入**：插入一个新的键值对时，哈希函数会计算键的哈希值，并将元素存储在对应的桶中。如果键重复，`std::unordered_multimap` 会允许插入相同键的多个值。
* **查找**：查找操作的时间复杂度通常为 O(1)，但在哈希冲突的情况下，复杂度可能退化为 O(n)。
* **删除**：删除操作的时间复杂度通常为 O(1)，但是也有可能退化为 O(n)。

`std::unordered_multimap` 与 `std::unordered_map` 最大的区别是，它允许多个相同的键存在。
