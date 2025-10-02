# std::array

`std::array` 是一个封装了**固定大小**数组的容器。它将 C 风格数组的性能和内存布局与标准容器的优点结合起来，例如能够知道自己的大小、支持赋值和随机访问迭代器等。`std::array` 的大小是模板参数的一部分，在**编译时**固定，因此它不支持动态大小的调整（如插入或删除元素）。

## 1\. 引入

```c++
#include <array>
```

## 2\. 存储方式

`std::array` 是一个**固定大小**的顺序容器，其元素是存储在**连续内存**中的，在栈中存储。它的内存布局与 C 风格数组 `T[N]` 完全相同，并且不包含任何额外的开销（例如，不需要存储大小）。

- **固定大小：** 数组的大小 `N` 是一个模板参数，在编译时确定，不能在运行时改变。
- **连续存储：** 元素在内存中是连续存放的，这使得 `std::array` 支持高效的**随机访问**（通过索引 \\(O(1)\\) 时间复杂度）和迭代器操作。
- **无数据成员开销：** 数组本身不存储除了元素之外的任何数据（例如，不需要存储大小），因此它的空间效率与 C 风格数组相同。

由于大小固定，`std::array` 不支持诸如 `push_back`、`pop_back` 或 `insert`、`erase` 等会改变容器大小的操作。

## 3\. 方法

`std::array` 的许多操作都是隐式定义的，因为它是一个聚合类型（Aggregate Type），遵循 C 风格数组的初始化和复制规则。

### (1) 构造方法

`std::array` 是一个聚合类型，因此它使用**聚合初始化**（Aggregate Initialization）规则：

1. **默认构造**：

    ```c++
    std::array<int, 5> arr; // 包含 5 个 int 元素，默认初始化（对于基本类型，值可能是不确定的）
    ```

2. **列表初始化/聚合初始化**：

    ```c++
    std::array<int, 3> arr1 = {1, 2, 3}; // 包含 3 个元素，值为 1, 2, 3
    std::array<int, 5> arr2 = {1, 2};   // 包含 5 个元素，前两个为 1, 2，其余为 0 (零初始化)
    ```

3. **复制/移动构造**：

    ```c++
    std::array<int, 3> arr3 = {1, 2, 3};
    std::array<int, 3> arr4(arr3); // 复制构造
    ```

### (2) 大小函数

1. `size_t size() const`: 返回 `std::array` 中元素的个数（即模板参数 \\(N\\)）。

    ```c++
    std::array<int, 4> arr;
    std::cout << arr.size(); // 输出 4
    ```

2. `bool empty() const`: 检查 `std::array` 是否为空。由于 \\(N\\) 在编译时固定，对于 \\(N > 0\\) 的数组总是返回 `false`。

    ```c++
    std::array<int, 0> arr0;
    std::cout << arr0.empty(); // 输出 true
    std::array<int, 5> arr5;
    std::cout << arr5.empty(); // 输出 false
    ```

3. `size_t max_size() const`: 返回容器可能包含的最大元素数，与 `size()` 相同。

### (3) 元素访问

1. `reference at(size_type pos)`: 访问指定位置 `pos` 的元素，**带边界检查**。如果 `pos` 超出范围，则抛出 `std::out_of_range` 异常。

    ```c++
    std::array<int, 3> arr = {1, 2, 3};
    std::cout << arr.at(1); // 输出 2
    // arr.at(5); // 运行时抛出异常
    ```

2. `reference operator[](size_type pos)`: 访问指定位置 `pos` 的元素，**不带边界检查**。

    ```c++
    std::array<int, 3> arr = {1, 2, 3};
    std::cout << arr[1]; // 输出 2
    // arr[5]; // 未定义行为
    ```

3. `reference front()`: 访问**第一个**元素。

    ```c++
    std::array<int, 3> arr = {1, 2, 3};
    std::cout << arr.front(); // 输出 1
    ```

4. `reference back()`: 访问**最后一个**元素。

    ```c++
    std::array<int, 3> arr = {1, 2, 3};
    std::cout << arr.back(); // 输出 3
    ```

5. `T* data()`: 返回指向存储元素的首元素的**底层 C 风格数组**的指针。

    ```c++
    std::array<int, 3> arr = {1, 2, 3};
    int* p = arr.data();
    std::cout << p[0]; // 输出 1
    ```

6. `std::get<I>(array)`: 使用**元组式接口**访问第 \\(I\\) 个元素，**编译时**进行边界检查。

    ```c++
    std::array<int, 3> arr = {1, 2, 3};
    std::cout << std::get<1>(arr); // 输出 2
    ```

### (4) 迭代器函数

`std::array` 支持随机访问迭代器。

1. `iterator begin()` / `const_iterator cbegin()`: 返回指向**第一个**元素的迭代器。
2. `iterator end()` / `const_iterator cend()`: 返回指向**最后一个元素之后**位置的迭代器。
3. `reverse_iterator rbegin()` / `const_reverse_iterator crbegin()`: 返回指向**最后一个**元素的反向迭代器。
4. `reverse_iterator rend()` / `const_reverse_iterator crend()`: 返回指向**第一个元素之前**位置的反向迭代器。

### (5) 修改函数

`std::array` 不支持改变大小的操作（如 `push_back`, `pop_back`, `insert`, `erase`）。

1. `void fill(const T& value)`: 将所有元素的值设置为 `value`。

    ```c++
    std::array<int, 5> arr;
    arr.fill(10); // arr 变为 {10, 10, 10, 10, 10}
    ```

2. `void swap(array& other)`: **交换**当前 `std::array` 和另一个大小、类型相同的 `std::array` 的内容。

    ```c++
    std::array<int, 3> arr1 = {1, 2, 3};
    std::array<int, 3> arr2 = {4, 5, 6};
    arr1.swap(arr2); // arr1 变为 {4, 5, 6}，arr2 变为 {1, 2, 3}
    ```

### (6) 遍历函数

由于 `std::array` 提供了随机访问迭代器，因此可以使用多种方式遍历。

1. 使用基于范围的 `for` 循环（C++11 及以上）：

    ```c++
    std::array<int, 3> arr = {1, 2, 3};
    for (const int& num : arr) {
        std::cout << num << " "; // 输出: 1 2 3
    }
    ```

2. 使用传统的 `for` 循环（基于迭代器）：

    ```c++
    std::array<int, 3> arr = {1, 2, 3};
    for (auto it = arr.begin(); it != arr.end(); ++it) {
        std::cout << *it << " "; // 输出: 1 2 3
    }
    ```

3. 使用传统的 `for` 循环（基于索引）：

    ```c++
    std::array<int, 3> arr = {1, 2, 3};
    for (size_t i = 0; i < arr.size(); ++i) {
        std::cout << arr[i] << " "; // 输出: 1 2 3
    }
    ```
