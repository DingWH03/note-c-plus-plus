# std::vector

`std::vector` 是一个封装了**动态大小数组**的顺序容器（Sequence Container）。与其它容器一样，它能够存放各种类型的对象，可以简单地认为它是一个能够存放任意类型的动态数组。

`std::vector` 支持**快速随机访问**，并且是 C++ 中最常用、最应该优先考虑的容器。在大多数场景下，如果你不确定该用哪个容器，`std::vector` 通常就是默认选择。

## 1. 引入

```c++
#include <vector>
```

## 2. 存储方式

为了支持随机访问，`std::vector` 将元素**连续存储**——每个元素紧挨着前一个元素存储。容器中元素是连续存储的，且容器的大小是可变的。

在容器中增加元素时，`std::vector` 会根据存储元素的大小，在内存上申请一块空间用于存储数据。空间的大小通常会大于所存储元素的实际大小，并且预留出一部分空间，以便再次增加数据时无需重新开辟空间。

当容器再次增加新的元素时，首先判断预留的空间是否够用：够用就直接在预留空间中存储；不够用则要在内存中开辟一整块新的更大的空间，把原来的数据拷贝（或移动）过去，再在新内存中加入新元素，这样才能保证存储空间始终连续。新开辟的空间同样会预留一部分，以便后续继续增加数据。当 `std::vector` 增长时，容量通常会按一定比例（通常是 1.5 或 2）增长，这有助于减少频繁的重新分配，提升性能。

需要注意的是，扩容意味着元素被搬到了新的内存地址，扩容之前保存的迭代器、指针和引用都会失效，继续使用属于未定义行为。如果提前知道元素数量，可以用 `reserve()` 预留空间，避免中途扩容。

```c++
std::vector<int> vec = {1, 2, 3};
int* p = &vec[0];
vec.push_back(4);   // 可能触发扩容，p 可能已经失效
// *p;              // 危险：未定义行为
```

上面说的"元素连续存储、可以取地址"，对 `std::vector<bool>` 并不成立。标准库对 `bool` 做了特化，把它实现成了一个按位存储的位容器，每个元素只占 1 个二进制位，8 个元素合起来才占 1 个字节，因此 `std::vector<bool>` 在省内存的同时也失去了普通 vector 的很多性质。这一点在后面几节会陆续提到。

## 3. 方法

### (1) 构造方法

1. `vector()`: 创建一个空 vector，创建时也可以使用初始化列表进行初始化

    ```c++
    std::vector<int> vec;
    std::vector<int> vec = {1, 2, 3, 4, 5};
    ```

2. `vector(size_t nSize)`: 创建一个包含 `nSize` 个元素的 vector，元素为值初始化（内置类型为 0）

    ```c++
    std::vector<int> vec(10);
    ```

3. `vector(size_t nSize, const T& value)`: 创建一个包含 `nSize` 个元素的 vector，且每个元素的值均为 `value`

    ```c++
    std::vector<int> vec(10, 5);
    ```

4. `vector(const vector&)`: 复制构造函数

    ```c++
    std::vector<int> vec1 = {1, 2, 3, 4, 5};
    std::vector<int> vec2(vec1);  // vec2 是 vec1 的一个复制版本
    ```

5. `vector(begin, end)`: 复制 \[begin, end\) 区间内另一个容器（或数组）的元素到 vector 中

    ```c++
    std::array<int, 5> arr = {1, 2, 3, 4, 5}; // 创建一个 std::array 或 std::vector
    std::vector<int> vec(arr.begin(), arr.begin() + 3);  // 复制前3个元素
    ```

### (2) 大小函数

1. `size_t size() const`: 返回 vector 中存放元素的实际数量（实际存储元素的个数）

2. `size_t capacity() const`: 返回 vector 在内存中开辟空间的容量（最多能放多少个元素而不需要重新扩容）

3. `size_t max_size() const`: 返回最大可允许的 vector 元素数量值

4. `bool empty() const`: 返回数组是否为空

5. `void shrink_to_fit()`: 调整数组容量（capacity）刚好适应当前的大小，节省内存

6. `void resize(size_t count)`: 修改 vector 的**大小（size）**。如果新大小比当前大小大，则用值初始化填充新增元素（可能触发扩容）；如果新大小比当前大小小，则删除多余的元素。

7. `void reserve(size_t n)`: 修改 vector 的 **capacity**，为数组预留空间，不改变 size。

这几个函数里，`size()`、`capacity()` 和 `reserve()` 是配合使用的重点。`size()` 是当前真正存了多少个元素，`capacity()` 是不扩容的前提下最多能放多少个，两者通常不相等，中间差的那部分就是预留空间。知道要装多少元素时，先 `reserve()` 一次性把容量要到够，可以避免边插入边扩容带来的反复拷贝。

对 `std::vector<bool>` 来说，这几个函数同样可用，但要注意 `capacity()` 的单位是**位**而不是字节，返回值表示的是能容纳多少个布尔值，而不是分配了多少内存。

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

除了末尾的 `push_back` 和 `emplace_back`，其余插入操作都需要移动插入点之后的元素，代价是 `O(n)`，所以能往末尾加就不要往中间插。

`std::vector<bool>` 的这些函数用法完全一样，但 `operator[]`、`front()`、`back()` 返回的都不是 `bool&`，而是一个代理对象（`std::vector<bool>::reference`），它内部记录了"哪一位"的信息，可以隐式转换成 `bool`，也可以赋值。用起来像引用，但它不是引用，所以 `&vec[0]` 是无法编译的：

```c++
std::vector<bool> vec = {true, false, true};
// bool* p = &vec[0];  // 错误：无法对 vector<bool> 的元素取地址
vec[0] = false;         // 可以，通过代理对象修改对应的位
bool value = vec[0];    // 可以，隐式转换为 bool
```

这一点在写模板代码时尤其容易出问题。下面这段代码对 `std::vector<int>` 没问题，但对 `std::vector<bool>` 直接编译不过，因为 `vec[i]` 返回的是右值代理对象，没法绑定到 `auto&`：

```c++
template <typename T>
void doubleAll(std::vector<T>& vec) {
    for (auto& x : vec) {  // vector<bool> 时无法绑定到 auto&
        x = x + x;
    }
}
```

另外，`auto a = vec[0];` 得到的也是代理对象而不是 `bool`。代理对象内部记录的是"哪一位"，如果它指向的 `std::vector<bool>` 被销毁或者重新分配了内存，再用它就是悬垂引用，属于未定义行为。想要真正的值，就显式写成 `bool b = vec[0];`。

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

删除操作里需要特别留意的是 `erase`。它删除的是 `[first, last)` 区间，删除位置及之后的迭代器、指针和引用都会失效，因此循环中删除元素时不能简单地 `++it`，而要接收 `erase` 返回的新迭代器：

```c++
std::vector<int> vec = {1, 2, 3, 4, 5, 6};
// 错误写法：it 在 erase 后失效，++it 是未定义行为
// for (auto it = vec.begin(); it != vec.end(); ++it) {
//     if (*it % 2 == 0) vec.erase(it);
// }

// 正确写法：用 erase 的返回值继续遍历
for (auto it = vec.begin(); it != vec.end(); ) {
    if (*it % 2 == 0) {
        it = vec.erase(it);  // erase 返回下一个有效迭代器
    } else {
        ++it;
    }
}
// vec 变为 {1, 3, 5}
```

`std::vector<bool>` 的删除操作和普通 vector 一样，迭代器失效规则也相同，同样要用 `erase` 的返回值继续遍历。

### (5) 遍历函数

1. `reference at(size_t pos)`: 返回 `pos` 位置元素的引用。与 `operator[]` 类似，但会检查边界，如果访问无效的位置会抛出 `std::out_of_range` 异常。

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

这里要再提一次 `std::vector<bool>`：它的迭代器不是指针，而是封装了位操作的类，因此不能当指针用，`&*it` 这类写法是非法的。前面提到的基于范围的 `for` 循环在它身上也会失效，因为 `for (auto& x : vec)` 绑定不上代理对象，只能写成 `for (bool x : vec)` 或 `for (auto&& x : vec)` 这样的形式。

### (6) 其他函数

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

4. `T* data()`: 返回指向底层数组中首元素的指针，便于与 C 风格接口交互。

   ```c++
   std::vector<int> vec = {1, 2, 3};
   int* p = vec.data();  // 等价于 &vec[0]
   std::cout << p[0] << std::endl;  // 输出: 1
   ```

`data()` 是普通 vector 与 C 接口打交道的主要方式，但 `std::vector<bool>` **没有** `data()` 成员，也没有 `bool*` 迭代器，所以 `std::memcpy(buffer, vec.data(), ...)` 这类写法以及任何要求 `bool*` 的 C 函数在它身上都用不了。`std::vector<bool>` 另外提供了一个特有的成员函数 `flip()`，用于翻转容器中所有的布尔值（相当于按位取反），普通 `std::vector` 没有这个函数。

```c++
std::vector<bool> vec;
vec.push_back(true);
vec.push_back(false);
vec.emplace_back(true);
vec[0] = false;                        // 通过代理对象修改
vec.flip();                            // 翻转所有位
std::cout << vec.size() << std::endl;  // 输出: 3
```

另外要注意，多个线程同时写同一个字节内的不同位（也就是不同的 `std::vector<bool>` 元素）会产生数据竞争，而 `std::vector<char>` 里相邻元素是独立的字节，不会有这个问题。还有一点，按位存储省的是内存，代价是每次读写都要做掩码和移位，访问单个元素比普通 `bool` 数组更贵，编译器也很难做向量化优化，如果代码是随机访问密集型的，`std::vector<bool>` 反而可能更慢。

总结一下，`std::vector<bool>` 适合的场景是只需要一个省内存的位集合，并且不需要取元素地址、不需要和 C 接口打交道。如果需要的是真正的 `bool` 数组语义，应该换成 `std::vector<char>`（每个元素占 1 字节，可以取地址），或者长度固定时用 `std::bitset`（按位存储，接口更丰富）。

```c++
std::vector<char> vec(10, 0);  // 每个元素占 1 字节，可以取地址
std::bitset<100> bits;         // 长度固定，按位存储
```
