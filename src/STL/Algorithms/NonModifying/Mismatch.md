# std::mismatch

`std::mismatch` 比较两个范围，返回**第一对不相等**的元素位置。如果所有对应元素都相等，则返回两个范围各自的末尾。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
// C++14 起有四个迭代器版本（可比较不同长度的范围）
template<class InputIt1, class InputIt2>
constexpr std::pair<InputIt1, InputIt2>
    mismatch(InputIt1 first1, InputIt1 last1, InputIt2 first2);

template<class InputIt1, class InputIt2>
constexpr std::pair<InputIt1, InputIt2>
    mismatch(InputIt1 first1, InputIt1 last1, InputIt2 first2, InputIt2 last2);

template<class InputIt1, class InputIt2, class BinaryPred>
constexpr std::pair<InputIt1, InputIt2>
    mismatch(InputIt1 first1, InputIt1 last1, InputIt2 first2, InputIt2 last2, BinaryPred p);
```

- **迭代器要求**：`InputIterator`。
- **复杂度**：至多 `min(N1, N2)` 次比较。
- **返回值**：`std::pair`，包含两个范围中第一个不匹配位置的迭代器。

C++14 新增的双范围重载（带 `last2`）更安全，因为可以处理两个范围长度不同的情况。C++98 版本只接受第二个范围的首迭代器，需要调用者保证第二个范围足够长。

`ranges::mismatch` 返回 `ranges::mismatch_result`。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> a{1, 2, 3, 4, 5};
    std::vector<int> b{1, 2, 9, 4, 5};

    auto [it1, it2] = std::mismatch(a.begin(), a.end(), b.begin(), b.end());
    if (it1 != a.end())
        std::cout << "首个不同: " << *it1 << " vs " << *it2 << '\n';
        // 首个不同: 3 vs 9
}
```

### (2) 谓词与投影

可以传入二元谓词自定义「相等」：

```c++
std::string s1 = "HELLO";
std::string s2 = "hello";

// 忽略大小写比较
auto [i1, i2] = std::mismatch(s1.begin(), s1.end(), s2.begin(), s2.end(),
                              [](char a, char b) {
                                  return std::tolower(a) == std::tolower(b);
                              });
std::cout << std::boolalpha << (i1 == s1.end());   // true（完全相同）
```

### (3) 执行策略

`mismatch` **不支持**执行策略，因为需要返回第一个不匹配位置。

## 4. 注意事项

- **优先用双范围重载**：C++14 起的四迭代器版本能正确处理长度不同的范围，避免越界。
- **长度不同时的行为**：单范围版本（三参数）只比较 `min(N1, N2)` 个元素，若第一个范围更长则不会发现问题。
- **返回 `pair`**：C++17 起可以用结构化绑定 `auto [it1, it2] = ...` 简化代码。
- **与 `equal` 的区别**：`mismatch` 返回位置，`equal` 只返回布尔值。

## 5. 相关算法

- [equal](./Equal.md)：只判断两个范围是否相同
- [lexicographical_compare](../Sorting/Lexicographical_compare.md)：按字典序比较
- [search](./Search.md)：查找子序列
- [find / find_if](./Find.md)：在单个范围内查找
