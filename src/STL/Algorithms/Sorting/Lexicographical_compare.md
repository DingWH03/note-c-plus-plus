# std::lexicographical_compare / lexicographical_compare_three_way

这两个算法按**字典序**比较两个范围：

- `lexicographical_compare`：返回布尔值，判断第一个范围是否字典序小于第二个
- `lexicographical_compare_three_way`：返回三路比较结果（C++20）

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt1, class InputIt2>
constexpr bool lexicographical_compare(InputIt1 first1, InputIt1 last1,
                                       InputIt2 first2, InputIt2 last2);

template<class InputIt1, class InputIt2, class Compare>
constexpr bool lexicographical_compare(InputIt1 first1, InputIt1 last1,
                                       InputIt2 first2, InputIt2 last2, Compare comp);

// C++20
template<class InputIt1, class InputIt2, class Cmp>
constexpr auto lexicographical_compare_three_way(InputIt1 first1, InputIt1 last1,
                                                 InputIt2 first2, InputIt2 last2, Cmp comp);
```

- **迭代器要求**：`InputIterator`。
- **复杂度**：至多 \\( 2\min(N_1, N_2) \\) 次比较。
- **返回值**：`lexicographical_compare` 返回 `bool`；`_three_way` 版本返回比较类别。

字典序规则：

1. 逐元素比较，遇到第一对不相等的元素，比较结果即为整体结果
2. 如果所有对应元素都相等，则**较短的范围较小**

这正是 `std::string`、`std::vector` 等容器 `operator<` 的语义。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{1, 2, 4};
    std::vector<int> c{1, 2};

    std::cout << std::boolalpha
              << std::lexicographical_compare(a.begin(), a.end(), b.begin(), b.end())
              << '\n'   // true（3 < 4）
              << std::lexicographical_compare(b.begin(), b.end(), a.begin(), a.end())
              << '\n'   // false
              << std::lexicographical_compare(c.begin(), c.end(), a.begin(), a.end())
              << '\n';  // true（c 是 a 的前缀且更短）

    // C++20 三路比较
    auto result = std::lexicographical_compare_three_way(
        a.begin(), a.end(), b.begin(), b.end(), std::compare_three_way{});
    std::cout << (result < 0 ? "a < b" : "a >= b") << '\n';   // a < b
}
```

### (2) 谓词与投影

自定义比较器，例如忽略大小写比较字符串：

```c++
std::string s1 = "Apple";
std::string s2 = "banana";

bool less = std::lexicographical_compare(
    s1.begin(), s1.end(), s2.begin(), s2.end(),
    [](char a, char b) { return std::tolower(a) < std::tolower(b); });
std::cout << std::boolalpha << less;   // true
```

### (3) 执行策略

`lexicographical_compare` **不支持**执行策略，因为需要短路返回。

## 4. 注意事项

- **长度不同时的规则**：如果一个是另一个的前缀，较短者较小。这与「先比较完所有元素再看长度」的直觉一致。
- **直接比较容器更简单**：`std::vector`、`std::string` 等都有 `operator<`，直接用 `a < b` 即可，内部就是字典序。
- **`_three_way` 返回比较类别**：需要包含 `<compare>`，返回 `std::strong_ordering` 等类型。
- **空范围**：空范围字典序小于任何非空范围，两个空范围相等。

## 5. 相关算法

- [equal](../NonModifying/Equal.md)：判断两个范围是否相同
- [mismatch](../NonModifying/Mismatch.md)：找第一个不匹配位置
- [is_permutation](./Permutation.md)：判断排列关系
- [sort / stable_sort](./Sort.md)：排序
