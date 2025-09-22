# std::deque

`std::deque`（双端队列）是一个支持在两端快速插入和删除元素的顺序容器。与 `std::vector` 不同，`std::deque` 在内存中采用分段连续的存储方式，即它的元素分布在多个不连续的内存块中，这些内存块通过指针数组进行管理。由于这一内存布局，`std::deque` 可以在头尾两端高效地进行插入和删除操作，并且支持随机访问(可以使用`operator[]`)。尽管如此，它的随机访问性能较 `std::vector` 差一些，因为每个内存块并不连续。

`std::deque` 不需要像 `std::vector` 一样每次扩展时重新分配整个内存空间，而是通过扩展指向内存块的指针数组来增加容量。这使得它在动态变化的场景中，尤其是在需要频繁在两端插入和删除元素时，表现得尤为高效。

## 1. 引入

```C++
#include <deque>
```

## 2. 存储方式

std::deque 是双端队列，支持在两端进行高效的插入和删除。为了实现这一点，std::deque 使用的是一种分段连续的内存结构。它的内存布局由多个固定大小的内存块（即“段”）组成，每个块内的元素是连续存储的，但这些块在内存中并不连续。通过维护一个指向这些块的指针数组，std::deque 可以实现高效的两端插入和删除操作。

当 std::deque 容量不足时，它会动态扩展新的内存块，并在原有的内存块之间插入新的块。这样，由于每个块都是独立分配的，std::deque 既能保证高效的插入和删除操作，又能提供支持随机访问的能力（虽然随机访问性能不如 std::vector）。

与 std::vector 不同的是，std::deque 不需要一次性重新分配整个内存空间，它只是扩展指向内存块的指针数组。这样，std::deque 可以高效地在头尾进行扩展，并且仍然保持较好的访问性能。

### 3. 方法

### (1) 构造方法

1. `deque()`: 创建一个空的 `std::deque`。

   ```c++
   std::deque<int> deq;
   ```

2. `deque(size_t nSize)`: 创建一个包含 `nSize` 个默认构造元素的 `std::deque`。

   ```c++
   std::deque<int> deq(10);  // 创建一个包含10个默认构造元素的 deque
   ```

3. `deque(size_t nSize, const T& value)`: 创建一个包含 `nSize` 个元素，且每个元素的值为 `value` 的 `std::deque`。

   ```c++
   std::deque<int> deq(10, 5);  // 创建一个包含10个元素，值都为5的 deque
   ```

4. `deque(const deque&)`: 复制构造函数，创建一个与另一个 `std::deque` 完全相同的副本。

   ```c++
   std::deque<int> deq1 = {1, 2, 3, 4, 5};
   std::deque<int> deq2(deq1);  // deq2 是 deq1 的一个复制版本
   ```

5. `deque(begin, end)`: 使用另一个容器或迭代器区间中的元素初始化 `std::deque`。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   std::deque<int> deq(vec.begin(), vec.end());  // 复制 vector 的元素到 deque
   ```

### (2) 大小函数

1. `size_t size() const`: 返回 `std::deque` 中元素的个数。

   ```c++
   std::deque<int> deq = {1, 2, 3, 4};
   std::cout << deq.size();  // 输出 4
   ```

2. `bool empty() const`: 检查 `std::deque` 是否为空，若为空返回 `true`，否则返回 `false`。

   ```c++
   std::deque<int> deq;
   if (deq.empty()) {
       std::cout << "Deque is empty." << std::endl;
   }
   ```

3. `void resize(size_t size)`: 调整 `std::deque` 的大小。若新大小大于当前大小，`std::deque` 会插入默认值的元素；若小于当前大小，则会删除多余的元素。

   ```c++
   std::deque<int> deq = {1, 2, 3};
   deq.resize(5, 10);  // 增加两个元素，元素值为 10
   ```

4. `void shrink_to_fit()`: 调整队列预留内存刚好适应当前的大小，节省内存
5. `size_t max_size() const`: 返回最大可允许的vector元素数量值

### (3) 增加函数

1. `push_back(const T& value)`: 将元素 `value` 添加到 `std::deque` 的末尾。

   ```c++
   std::deque<int> deq = {1, 2, 3};
   deq.push_back(4);  // 向 deq 中添加元素 4，deq 变为 {1, 2, 3, 4}
   ```

2. `push_front(const T& value)`: 将元素 `value` 添加到 `std::deque` 的开头。

   ```c++
   std::deque<int> deq = {2, 3, 4};
   deq.push_front(1);  // 向 deq 中添加元素 1，deq 变为 {1, 2, 3, 4}
   ```

3. `emplace_back(Args&&... args)`: 在 `std::deque` 的末尾就地构造一个元素，使用提供的参数直接构造该元素。

   ```c++
   std::deque<std::pair<int, int>> deq;
   deq.emplace_back(1, 2);  // 在末尾构造一个 pair<int, int>，值为 {1, 2}
   ```

4. `emplace_front(Args&&... args)`: 在 `std::deque` 的前端就地构造一个元素。

   ```c++
   std::deque<std::pair<int, int>> deq;
   deq.emplace_front(1, 2);  // 在前端构造一个 pair<int, int>，值为 {1, 2}
   ```

5. `insert(iterator pos, const T& value)`: 在指定位置 `pos` 插入元素 `value`，插入操作会将 `pos` 后面的元素向后移动。

   ```c++
   std::deque<int> deq = {1, 2, 4, 5};
   deq.insert(deq.begin() + 2, 3);  // 在位置2插入3，deq 变为 {1, 2, 3, 4, 5}
   ```

6. `insert(iterator pos, size_t count, const T& value)`: 在指定位置 `pos` 插入 `count` 个元素，所有的元素值为 `value`。

   ```c++
   std::deque<int> deq = {1, 2, 4, 5};
   deq.insert(deq.begin() + 2, 2, 3);  // 在位置2插入两个3，deq 变为 {1, 2, 3, 3, 4, 5}
   ```

7. `insert(iterator pos, InputIterator first, InputIterator last)`: 将 `[first, last)` 区间的元素插入到指定位置 `pos`。

   ```c++
   std::deque<int> deq1 = {1, 2, 3};
   std::deque<int> deq2 = {4, 5};
   deq1.insert(deq1.begin() + 2, deq2.begin(), deq2.end());  // 在位置2插入 deq2 中的元素，deq1 变为 {1, 2, 4, 5, 3}
   ```

### (4) 删除函数

1. `pop_back()`: 删除 `std::deque` 中的最后一个元素。

   ```c++
   std::deque<int> deq = {1, 2, 3, 4};
   deq.pop_back();  // 删除最后一个元素，deq 变为 {1, 2, 3}
   ```

2. `pop_front()`: 删除 `std::deque` 中的第一个元素。

   ```c++
   std::deque<int> deq = {1, 2, 3, 4};
   deq.pop_front();  // 删除第一个元素，deq 变为 {2, 3, 4}
   ```

3. `erase(iterator pos)`: 删除指定位置 `pos` 处的元素，删除操作后，`pos` 后面的元素会向前移动。

   ```c++
   std::deque<int> deq = {1, 2, 3, 4};
   deq.erase(deq.begin() + 2);  // 删除索引为2的元素，deq 变为 {1, 2, 4}
   ```

4. `erase(iterator first, iterator last)`: 删除区间 `[first, last)` 内的所有元素。

   ```c++
   std::deque<int> deq = {1, 2, 3, 4, 5};
   deq.erase(deq.begin() + 1, deq.begin() + 4);  // 删除从索引1到3的元素，deq 变为 {1, 5}
   ```

5. `clear()`: 删除 `std::deque` 中的所有元素，使容器变为空。

   ```c++
   std::deque<int> deq = {1, 2, 3, 4};
   deq.clear();  // 删除所有元素，deq 变为空 {}
   ```

### (5) 遍历函数

1. `reference at(int pos)`: 同Vector, 返回 `pos` 位置元素的引用。与 `operator[]` 类似，但会检查边界，如果访问无效的位置会抛出 `std::out_of_range` 异常。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   int& element = vec.at(2);  // 返回位置 2 处元素的引用，即值为 3
   element = 10;  // 修改元素为 10
   std::cout << vec[2] << std::endl;  // 输出: 10
   ```

2. `iterator begin()`: 返回指向 `std::deque` 第一个元素的迭代器。

   ```c++
   std::deque<int> deq = {1, 2, 3};
   std::deque<int>::iterator it = deq.begin();  // 返回指向第一个元素的迭代器
   std::cout << *it << std::endl;  // 输出: 1
   ```

3. `iterator end()`: 返回指向 `std::deque` 最后一个元素之后位置的迭代器。

   ```c++
   std::deque<int> deq = {1, 2, 3};
   std::deque<int>::iterator it = deq.end();  // 返回指向最后一个元素之后位置的迭代器
   --it;  // 移动到最后一个元素
   std::cout << *it << std::endl;  // 输出: 3
   ```

4. `reverse_iterator rbegin()`: 返回指向 `std::deque` 最后一个元素的反向迭代器。

   ```c++
   std::deque<int> deq = {1, 2, 3};
   std::deque<int>::reverse_iterator rit = deq.rbegin();  // 返回指向最后一个元素的反向迭代器
   std::cout << *rit << std::endl;  // 输出: 3
   ```

5. `reverse_iterator rend()`: 返回指向 `std::deque` 第一个元素之前位置的反向迭代器。

   ```c++
   std::deque<int> deq = {1, 2, 3};
   std::deque<int>::reverse_iterator rit = deq.rend();  // 返回指向第一个元素之前位置的反向迭代器
   ++rit;  // 移动到第一个元素
   std::cout << *rit << std::endl;  // 输出: 1
   ```

6. 使用基于范围的 `for` 循环（C++11 及以上）: 直接遍历 `std::deque` 中的每个元素。

   ```c++
   std::deque<int> deq = {1, 2, 3};
   for (const int& num : deq) {
       std::cout << num << " ";  // 输出: 1 2 3
   }
   ```

7. 使用传统的 `for` 循环（基于迭代器）: 使用迭代器来遍历 `std::deque`。

   ```c++
   std::deque<int> deq = {1, 2, 3};
   for (auto it = deq.begin(); it != deq.end(); ++it) {
       std::cout << *it << " ";  // 输出: 1 2 3
   }
   ```

### (6) 其他函数

1. `swap(deque& other)`: 交换当前 `std::deque` 和另一个 `std::deque` 的内容。

   ```c++
   std::deque<int> deq1 = {1, 2, 3};
   std::deque<int> deq2 = {4, 5, 6};
   deq1.swap(deq2);  // deq1 变为 {4, 5, 6}，deq2 变为 {1, 2, 3}
   ```

2. `assign(size_t count, const T& value)`: 将 `count` 个 `value` 元素赋值给当前 `std::deque`，替换原有内容。

   ```c++
   std::deque<int> deq;
   deq.assign(5, 10);  // 将 deq 赋值为 {10, 10, 10, 10, 10}
   ```

3. `assign(InputIterator first, InputIterator last)`: 使用区间 `[first, last)` 的元素来填充当前 `std::deque`，替换原有内容。

   ```c++
   std::deque<int> deq1 = {1, 2, 3};
   std::deque<int> deq2;
   deq2.assign(deq1.begin(), deq1.end());  // deq2 变为 {1, 2, 3}
   ```

4. `std::allocator<T> get_allocator() const`: 返回容器所使用的分配器（allocator）。分配器负责容器内存的管理，包括内存的分配、释放等操作。在默认情况下，C++ 标准库容器使用 std::allocator 作为默认的分配器。通过调用 get_allocator()，你可以获取容器使用的分配器，并利用该分配器进行自定义内存操作，或者检查容器如何管理内存。

    ```c++
    #include <iostream>
    #include <vector>

    int main() {
        std::vector<int> vec;

        // 获取分配器
        std::allocator<int> alloc = vec.get_allocator();

        // 使用分配器进行内存分配
        int* p = alloc.allocate(5);  // 分配5个int元素的内存

        // 显示分配的内存地址
        std::cout << "Allocated memory at: " << p << std::endl;

        // 记得释放分配的内存
        alloc.deallocate(p, 5);  // 释放之前分配的5个int内存

        return 0;
    }
    ```
