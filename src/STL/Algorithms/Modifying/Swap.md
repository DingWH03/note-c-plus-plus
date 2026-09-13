# std::swap / std::swap_ranges / std::iter_swap

这三个算法用于交换内容：

- `swap`：交换两个对象的值
- `swap_ranges`：交换两个等长范围的元素
- `iter_swap`：交换两个迭代器所指向的元素

## 1. 引入

```c++
#include <utility>    // std::swap, std::iter_swap（C++11 起）
#include <algorithm>  // std::swap_ranges；C++11 前 swap 也在此
```

## 2. 原理

```c++
// <utility>
template<class T>
constexpr void swap(T& a, T& b) noexcept(/* see below */);

template<class ForwardIt1, class ForwardIt2>
constexpr void iter_swap(ForwardIt1 a, ForwardIt2 b);

// <algorithm>
template<class ForwardIt1, class ForwardIt2>
constexpr ForwardIt2 swap_ranges(ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2);
```

- **迭代器要求**：`swap_ranges` 和 `iter_swap` 需要 `ForwardIterator`。
- **复杂度**：`swap` 为常数；`swap_ranges` 恰好 \\( n \\) 次交换。
- **返回值**：`swap_ranges` 返回第二个范围中最后一个被交换元素之后的位置。

`std::swap` 的经典实现是三次移动：

```c++
template<class T>
void swap(T& a, T& b) {
    T tmp = std::move(a);
    a = std::move(b);
    b = std::move(tmp);
}
```

但标准库对标准容器提供了特化版本（如 `std::vector::swap`），只交换内部指针，是 \\( O(1) \\) 的。

**注意**：C++11 起 `std::swap` 的声明从 `<algorithm>` 移到了 `<utility>`。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <utility>
#include <vector>

int main()
{
    // 交换两个对象
    int a = 1, b = 2;
    std::swap(a, b);   // a=2, b=1

    // 交换两个范围的元素
    std::vector<int> v1{1, 2, 3};
    std::vector<int> v2{4, 5, 6};
    std::swap_ranges(v1.begin(), v1.end(), v2.begin());
    // v1 = {4,5,6}, v2 = {1,2,3}

    // 交换两个迭代器指向的元素
    std::iter_swap(v1.begin(), v1.begin() + 2);
    // v1 = {6,5,4}
}
```

### (2) 谓词与投影

这三个算法都没有谓词或投影参数。

### (3) 执行策略

`swap_ranges` 支持 C++17 执行策略：

```c++
#include <execution>

std::swap_ranges(std::execution::par, v1.begin(), v1.end(), v2.begin());
```

`std::swap` 和 `std::iter_swap` 是单次操作，不需要执行策略。

## 4. 注意事项

- **头文件位置**：C++11 起 `std::swap` 在 `<utility>`，不是 `<algorithm>`。
- **对容器用成员 `swap`**：`vec1.swap(vec2)` 或 `std::swap(vec1, vec2)` 都是 \\( O(1) \\)，只交换内部指针，迭代器保持有效（跟随元素转移）。
- **`iter_swap` 支持 ADL**：`ranges::iter_swap` 会对 `iter_swap` 做 ADL 查找，而 `std::iter_swap` 不会。
- **自定义类型的 swap**：应在同命名空间提供 `swap` 重载，以便 ADL 找到更高效的实现。
- **`swap_ranges` 要求等长**：两个范围必须长度相同，不检查第二个范围是否足够长。

## 5. 相关算法

- [copy / copy_if / copy_n](./Copy.md)：复制而非交换
- [move / move_backward](./Move.md)：移动而非交换
- [reverse](./Reverse.md)：反转顺序
- [rotate](./Rotate.md)：旋转顺序
