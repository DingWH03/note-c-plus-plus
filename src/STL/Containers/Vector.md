# std::vector

向量（Vector）是一个封装了动态大小数组的顺序容器（Sequence Container）。跟任意其它类型容器一样，它能够存放各种类型的对象。可以简单的认为，向量是一个能够存放任意类型的动态数组。Vector支持快速随机访问。

## 1. 引入

```C++
#include <vector>
```

## 2. 存储方式

为了支持随机访问，vector将元素连续存储–每个元素紧挨着前一个元素存储。容器中元素是连续存储的，且容器的大小是可变的。

在容器中增加元素时。vector根据存储元素的大小，在内存上申请一个空间，用于存储数据，空间的大小通常会大于所存储元素的实际大小，并且预留出一部分预留的空间，以便再次增加数据时，可以不用重新开辟空间。

当容器再次增加新的元素后，首先判断预留的空间是否够用，如果够用直接在预留空间中存储。如果预留的空间不够，需要在内存中开辟一整块新的更大的空间，并将vector原来的存储的数据**拷贝**过来，存储到新的内存中，然后在新的内存中增加需要增加的元素，这样保证存储的空间是连续的。所开的空间会预留出一部分空间，以便后续增加数据。

当 vector 增长时，容量通常会按一定比例（通常是1.5或2）增长。这有助于减少频繁的重新分配，提升性能。

## 3. 方法

### (1)构造方法

1. `vector()`: 创建一个空vector，创建时也可以使用迭代器进行初始化

    ```c++
    std::vector<int> vec;
    std::vector<int> vec = {1, 2, 3, 4, 5};
    ```

2. `vector(size_t nSize)`: 创建一个vector,元素个数为nSize，默认构造函数会赋值与默认值

    ```c++
    std::vector<int> vec(10);
    ```

3. `vector(size_t nSize,const t& t)`: 创建一个vector，元素个数为nSize,且值均为t

    ```c++
    std::vector<int> vec(10, 5);
    ```

4. `vector(const vector&)`: 复制构造函数

    ```c++
    std::vector<int> vec1 = {1, 2, 3, 4, 5};
    std::vector<int> vec2(vec1);  // vec2 是 vec1 的一个复制版本
    ```

5. `vector(begin,end)`: 复制\[begin,end\)区间内另一个数组的元素到vector中

    ```c++
    std::array<int, 5> arr = {1, 2, 3, 4, 5}; // 创建一个 std::array 或 std::vector
    std::vector<int> vec(arr.begin(), arr.begin() + 3);  // 复制前3个元素
    ```

### (2)大小函数

1. `size_t size() const`: 返回vector中存放元素的实际数量（实际存储元素的个数）

2. `size_t capacity() const`: 返回vector在内存中，开辟空间的容量（最多能放所少个元素不需要重新扩容）

3. `size_t max_size() const`: 返回最大可允许的vector元素数量值

4. `bool empty() const`: 返回数组是否为空

5. `void shrink_to_fit()`: 调整数组大小(capacity)刚好适应当前的大小，节省内存

6. `void resize(size_t size)`: 修改 vector 的大小。如果新大小比当前大小大，则扩大数组容量，会导致控制帧。如果新大小比当前大小小，则不做处理

7. `void reserve(size_t n)`: 修改 vector 的capacity，为数组预留空间，不改变size。

### (3) 增加函数

1. `push_back(const T& value)`: 将元素 `value` 添加到 vector 的末尾。如果 vector 已满，`push_back` 会自动扩展容量。

   ```c++
   std::vector<int> vec = {1, 2, 3};
   vec.push_back(4);  // 向 vec 中添加元素 4，vec 变为 {1, 2, 3, 4}
   ```

2. `emplace_back(Args&&... args)`: 在 vector 的末尾就地构造一个元素，使用提供的参数直接构造该元素，而不是首先创建元素再添加。这样可以减少不必要的拷贝或移动操作。

   ```c++
   std::vector<std::pair<int, int>> vec;
   vec.emplace_back(1, 2);  // 在末尾构造一个 pair<int, int>，值为 {1, 2}
   ```

3. `insert(iterator pos, const T& value)`: 在指定位置 `pos` 插入一个元素。元素会被插入到 `pos` 之前，`pos` 之后的元素会被向后移动。

   ```c++
   std::vector<int> vec = {1, 2, 4, 5};
   vec.insert(vec.begin() + 2, 3);  // 在位置2插入3，vec 变为 {1, 2, 3, 4, 5}
   ```

4. `insert(iterator pos, size_t count, const T& value)`: 在指定位置 `pos` 插入 `count` 个元素，所有的元素值为 `value`。

   ```c++
   std::vector<int> vec = {1, 2, 4, 5};
   vec.insert(vec.begin() + 2, 2, 3);  // 在位置2插入两个 3，vec 变为 {1, 2, 3, 3, 4, 5}
   ```

5. `insert(iterator pos, InputIterator first, InputIterator last)`: 将 `[first, last)` 区间的元素插入到 `pos` 位置。

   ```c++
   std::vector<int> vec1 = {1, 2, 3};
   std::vector<int> vec2 = {4, 5};
   vec1.insert(vec1.begin() + 2, vec2.begin(), vec2.end());  // 在位置 2 插入 vec2 中的元素，vec1 变为 {1, 2, 4, 5, 3}
   ```

6. `emplace(iterator pos, Args&&... args)`: 在指定位置 `pos` 就地构造一个元素，使用提供的参数直接构造该元素。

   ```c++
   std::vector<std::pair<int, int>> vec;
   vec.emplace(vec.begin(), 1, 2);  // 在位置0处构造一个 pair<int, int>，值为 {1, 2}
   ```

### (4) 删除函数

1. `pop_back()`: 删除 vector 中的最后一个元素。该函数不会改变容器的容量，只是移除最后一个元素并缩小容器的大小。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4};
   vec.pop_back();  // 删除最后一个元素，vec 变为 {1, 2, 3}
   ```

2. `erase(iterator pos)`: 删除指定位置 `pos` 处的元素。删除后，后面的元素会向前移动。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4};
   vec.erase(vec.begin() + 2);  // 删除索引为2的元素，vec 变为 {1, 2, 4}
   ```

3. `erase(iterator first, iterator last)`: 删除 `[first, last)` 区间内的所有元素。此操作删除从 `first` 到 `last` 之间的元素（不包括 `last`）。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   vec.erase(vec.begin() + 1, vec.begin() + 4);  // 删除索引从 1 到 3 的元素，vec 变为 {1, 5}
   ```

4. `clear()`: 删除 vector 中的所有元素，容器变为空，但容器的容量不会立即改变，直到发生重新分配。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4};
   vec.clear();  // 删除所有元素，vec 变为空 { }
   ```

### (5)遍历函数

1. `reference at(int pos)`: 返回 `pos` 位置元素的引用。与 `operator[]` 类似，但会检查边界，如果访问无效的位置会抛出 `std::out_of_range` 异常。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   int& element = vec.at(2);  // 返回位置 2 处元素的引用，即值为 3
   element = 10;  // 修改元素为 10
   std::cout << vec[2] << std::endl;  // 输出: 10
   ```

2. `reference front()`: 返回 vector 的第一个元素的引用。如果 vector 为空，调用该方法会导致未定义行为。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   int& firstElement = vec.front();  // 返回第一个元素的引用，即值为 1
   firstElement = 20;  // 修改第一个元素为 20
   std::cout << vec.front() << std::endl;  // 输出: 20
   ```

3. `reference back()`: 返回 vector 的最后一个元素的引用。如果 vector 为空，调用该方法会导致未定义行为。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   int& lastElement = vec.back();  // 返回最后一个元素的引用，即值为 5
   lastElement = 50;  // 修改最后一个元素为 50
   std::cout << vec.back() << std::endl;  // 输出: 50
   ```

4. `iterator begin()`: 返回指向 vector 第一个元素的迭代器。这个迭代器指向 vector 的首元素。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   std::vector<int>::iterator it = vec.begin();  // 返回指向第一个元素的迭代器
   std::cout << *it << std::endl;  // 输出: 1
   ```

5. `iterator end()`: 返回指向 vector 最后一个元素之后位置的迭代器。这个迭代器指向 vector 的尾元素的下一个位置。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   std::vector<int>::iterator it = vec.end();  // 返回指向最后一个元素之后位置的迭代器
   --it;  // 移动到最后一个元素
   std::cout << *it << std::endl;  // 输出: 5
   ```

6. `reverse_iterator rbegin()`: 返回指向 vector 最后一个元素的反向迭代器。该迭代器可以用来从后往前遍历元素。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   std::vector<int>::reverse_iterator rit = vec.rbegin();  // 返回指向最后一个元素的反向迭代器
   std::cout << *rit << std::endl;  // 输出: 5
   ```

7. `reverse_iterator rend()`: 返回指向 vector 第一个元素之前位置的反向迭代器。该迭代器指向 vector 的第一个元素之前的位置。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   std::vector<int>::reverse_iterator rit = vec.rend();  // 返回指向第一个元素之前位置的反向迭代器
   ++rit;  // 移动到第一个元素
   std::cout << *rit << std::endl;  // 输出: 1
   ```

8. 使用基于范围的 `for` 循环（C++11 及以上）: 可以直接遍历 `std::vector` 中的每个元素，语法简洁。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   for (const int& num : vec) {
       std::cout << num << " ";  // 输出: 1 2 3 4 5
   }
   ```

9. 使用传统的 `for` 循环（基于索引）: 使用索引来遍历 `std::vector`，适合在需要访问元素索引的情况下使用。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4, 5};
   for (size_t i = 0; i < vec.size(); ++i) {
       std::cout << vec[i] << " ";  // 输出: 1 2 3 4 5
   }
   ```

### (6)其他函数

1. `swap(vector& other)`: 交换当前 vector 和另一个 vector 的内容。如果需要“删除”当前 vector 中的元素，可以通过交换将其与一个空的 vector 交换，从而达到清空的效果。

   ```c++
   std::vector<int> vec = {1, 2, 3, 4};
   std::vector<int> emptyVec;
   vec.swap(emptyVec);  // vec 变为空，emptyVec 变为 {1, 2, 3, 4}
   ```

2. `assign(size_t count, const T& value)`: 将 `count` 个 `value` 元素赋值给当前 vector，替换原有内容。

   ```c++
   std::vector<int> vec;
   vec.assign(5, 10);  // 将 vec 赋值为 {10, 10, 10, 10, 10}
   ```

3. `assign(InputIterator first, InputIterator last)`: 使用区间 `[first, last)` 的元素来填充当前 vector，替换原有内容。

   ```c++
   std::vector<int> vec1 = {1, 2, 3};
   std::vector<int> vec2;
   vec2.assign(vec1.begin(), vec1.end());  // vec2 变为 {1, 2, 3}
   ```
