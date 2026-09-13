# std::generate / std::generate_n

这两个算法用**连续调用函数对象**的返回值填充范围：

- `generate`：填充整个范围
- `generate_n`：填充前 \\( N \\) 个元素

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt, class Generator>
constexpr void generate(ForwardIt first, ForwardIt last, Generator g);

template<class OutputIt, class Size, class Generator>
constexpr OutputIt generate_n(OutputIt first, Size count, Generator g);
```

- **迭代器要求**：`generate` 需要 `ForwardIterator`；`generate_n` 需要 `OutputIterator`。
- **复杂度**：恰好 \\( n \\) 次函数调用和赋值。
- **返回值**：`generate` 返回 `void`；`generate_n` 返回最后一个被赋值元素之后的位置。

函数对象 `g` 按值传递，每次调用后其状态（如果有）会被保留。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    // 用递增计数器填充
    int n = 0;
    std::vector<int> v(5);
    std::generate(v.begin(), v.end(), [&n] { return n++; });
    // v = {0,1,2,3,4}

    // 用随机数填充
    std::mt19937 gen(std::random_device{}());
    std::uniform_int_distribution<int> dist(1, 100);
    std::vector<int> r(5);
    std::generate(r.begin(), r.end(), [&] { return dist(gen); });

    // generate_n 配合 back_inserter
    std::vector<int> v2;
    std::generate_n(std::back_inserter(v2), 3, [&n] { return n++; });
    // v2 = {5,6,7}
}
```

### (2) 谓词与投影

`generate` 接受的是**生成器**（无参可调用对象），没有谓词或投影参数。

### (3) 执行策略

`generate` 和 `generate_n` 支持 C++17 执行策略：

```c++
#include <execution>

std::generate(std::execution::par, v.begin(), v.end(), [&] { return dist(gen); });
```

**警告**：并行执行时生成器的调用顺序不确定，如果生成器依赖共享状态（如上例的 `n++`），会产生数据竞争。并行版本要求生成器是线程安全的。

## 4. 注意事项

- **生成器按值传递**：需要修改外部变量时用引用捕获。
- **并行下的状态问题**：依赖内部状态的生成器（如计数器）在并行下不安全。
- **`generate_n` 的 `count` 非负**。
- **`iota` 更适合递增序列**：如果只是要 0,1,2,...，用 `std::iota` 更简洁。
- **C++26 的 `ranges::generate_random`**：专门用于用随机数生成器填充范围，比 `generate` 配合 lambda 更高效。

## 5. 相关算法

- [fill / fill_n](./Fill.md)：用固定值填充
- [iota](../Numeric/Iota.md)：用递增序列填充
- [for_each](../NonModifying/For_each.md)：遍历但不赋值
- [transform](./Transform.md)：基于已有元素变换
