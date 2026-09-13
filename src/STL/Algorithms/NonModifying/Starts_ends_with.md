# std::ranges::starts_with / std::ranges::ends_with

这两个算法判断一个范围是否**以另一个范围开始或结束**，**只有 Ranges 版本**：

- `ranges::starts_with`：判断范围是否以给定范围作为前缀
- `ranges::ends_with`：判断范围是否以给定范围作为后缀

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<std::input_iterator I1, std::sentinel_for<I1> S1,
         std::input_iterator I2, std::sentinel_for<I2> S2,
         class Pred = ranges::equal_to, class Proj1 = std::identity, class Proj2 = std::identity>
requires (std::forward_iterator<I1> || std::sized_sentinel_for<S1, I1>) &&
         (std::forward_iterator<I2> || std::sized_sentinel_for<S2, I2>) &&
         std::indirectly_comparable<I1, I2, Pred, Proj1, Proj2>
constexpr bool starts_with(I1 first1, S1 last1, I2 first2, S2 last2,
                           Pred pred = {}, Proj1 proj1 = {}, Proj2 proj2 = {});
```

- **迭代器要求**：`InputIterator`（但需要能判断长度，或为 forward 迭代器）。
- **复杂度**：至多 `min(N1, N2)` 次比较。
- **返回值**：布尔值。
- **空前缀/后缀**：对空范围返回 `true`。

`ends_with` 的实现需要先求出两个范围的长度，因此对非 forward 迭代器会先计算距离。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    std::cout << std::boolalpha
              << std::ranges::starts_with(v, std::vector{1, 2}) << '\n'   // true
              << std::ranges::ends_with(v, std::vector{4, 5}) << '\n';    // true

    // 字符串场景
    std::string s = "hello world";
    std::cout << std::ranges::starts_with(s, std::string_view{"hello"}) << '\n';  // true
}
```

### (2) 谓词与投影

可以传入自定义比较谓词，例如忽略大小写：

```c++
std::string s = "Hello World";
std::string prefix = "HELLO";

bool ok = std::ranges::starts_with(s, prefix,
                                   [](char a, char b) {
                                       return std::tolower(a) == std::tolower(b);
                                   });   // true
```

### (3) 执行策略

**不支持**执行策略，因为需要短路返回。

## 4. 注意事项

- **只有 Ranges 版本**：C++23 才引入，没有 `std::starts_with`。
- **字符串有更直接的替代**：`std::string::starts_with` 和 `std::string::ends_with`（C++20）更简洁高效。
- **`ends_with` 的开销**：对非 forward 迭代器需要先求长度，可能多一次遍历。
- **空范围返回 `true`**：任何范围都以空范围开始和结束，这符合数学定义但需注意。

## 5. 相关算法

- [contains](./Contains.md)：判断是否包含元素或子范围
- [search / search_n](./Search.md)：查找子序列位置
- [equal](./Equal.md)：比较两个范围是否相同
- [mismatch](./Mismatch.md)：查找第一个不匹配位置
