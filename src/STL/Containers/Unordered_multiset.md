# std::unordered\_multiset

`std::unordered_multiset` 是一个基于哈希表实现的容器，用于存储多个相同的元素，并且不保证元素的顺序。与 `std::unordered_set` 类似，`std::unordered_multiset` 使用哈希表来存储元素，允许插入重复的元素。

## 1. 引入

```C++
#include <unordered_set>
```

## 2. 存储方式

`std::unordered_multiset` 使用哈希表来存储元素。每个元素的位置是通过哈希函数计算的，存储时不保证元素的顺序。与 `std::unordered_set` 不同，`std::unordered_multiset` 允许重复元素的插入，因此可以存储多个相同的元素。

* **插入**：插入一个新元素时，哈希函数会计算元素的哈希值，并将其存储在哈希表中。如果插入的元素已经存在，`std::unordered_multiset` 会允许插入重复元素。
* **查找**：查找操作的时间复杂度通常为 O(1)，但是在哈希冲突的情况下，可能退化为 O(n)。
* **删除**：删除操作的时间复杂度通常为 O(1)，但最坏情况下可能退化为 O(n)。

`std::unordered_multiset` 与 `std::unordered_set` 最大的区别是，它允许存储重复元素，适用于需要存储多个相同元素的场景。
