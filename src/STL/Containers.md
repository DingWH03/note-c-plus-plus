# Containers

容器是用来存储数据的序列，它们提供了不同的存储方式和访问模式。

| 容器类别 | 容器名称 | 描述 |
|---------|--------|------|
| **序列容器 (Sequence Containers)** | **std::vector** | 动态数组，支持快速随机访问。 |
|                      | **std::deque**               | 双端队列，支持快速插入和删除。 |
|                      | **std::list**                | 链表，支持快速插入和删除，但不支持随机访问。 |
| **关联容器 (Associative Containers)** | **std::set**                 | 集合，不允许重复元素。 |
|                      | **std::multiset**            | 多重集合，允许多个元素具有相同的键。 |
|                      | **std::map**                 | 映射，每个键映射到一个值。 |
|                      | **std::multimap**            | 多重映射，存储键值对（pair），其中键是唯一的，但值可以重复，允许一个键映射到多个值。 |
| **无序容器 (Unordered Containers)**  | **std::unordered_set**       | 无序集合，哈希表实现，支持快速查找、插入和删除。  |
|                      | **std::unordered_multiset**  | 无序多重集合，哈希表实现，允许多个元素具有相同的键。 |
|                      | **std::unordered_map**       | 无序映射，哈希表实现，每个键映射到一个值。 |
|                      | **std::unordered_multimap**  | 无序多重映射，哈希表实现，允许一个键映射到多个值。 |
