# std::equal

`std::equal` 判断两个范围内的元素是否**逐一相等**（或满足给定二元谓词），返回布尔值。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
// C++14 起有四个迭代器版本
template<class InputIt1, class InputIt2>
constexpr bool equal(InputIt1 first1, InputIt1 last1, InputIt2 first2);

template<class InputIt1, class InputIt2>
constexpr bool equal(InputIt1 first1, InputIt1 last1, InputIt2 first2, InputIt2 last2);

template<class InputIt1, class InputIt2, class BinaryPred>
constexpr bool equal(InputIt1 first1, InputIt1 last1, InputIt2 first2, InputIt2 last2,
                     BinaryPred p);
```

- **迭代器要求**：`InputIterator`。
- **复杂度**：至多 `min(N1, N2)` 次比较。
- **返回值**：全部相等返回 `true`，否则 `false`。
- **空范围**：两个空范围比较返回 `true`。

**重要**：C++14 之前的三参数版本只比较 `N1` 个元素，如果第二个范围更短会导致越界（未定义行为）；如果第二个范围更长，则多余的尾部元素会被忽略。C++14 起的四参数版本才会真正比较长度。

`ranges::equal` 支持投影，并且可以接受不同长度的范围。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> a{1, 2, 3};
    std::vector<int> b{1, 2, 3};
    std::vector<int> c{1, 2, 3, 4};

    std::cout << std::boolalpha
              << std::equal(a.begin(), a.end(), b.begin(), b.end()) << '\n'   // true
              << std::equal(a.begin(), a.end(), c.begin(), c.end()) << '\n';  // false
}
```

### (2) 谓词与投影

```c++
std::string s1 = "HELLO";
std::string s2 = "hello";

// 忽略大小写比较
bool same = std::equal(s1.begin(), s1.end(), s2.begin(), s2.end(),
                       [](char a, char b) {
                           return std::tolower(a) == std::tolower(b);
                       });
std::cout << std::boolalpha << same;   // true
```

`ranges::equal` 还支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> p1{{"Alice", 30}}, p2{{"Bob", 30}};

bool sameAge = std::ranges::equal(p1, p2, {}, &Person::age, &Person::age);   // true
```

### (3) 执行策略

`std::equal` **不支持**执行策略。但 `ranges::equal` 在 C++26 的并行范围算法中也不支持——它需要短路返回，无法有效并行化。

## 4. 注意事项

- **务必使用四参数版本**：三参数版本不检查第二个范围的长度，是经典的越界陷阱。
- **`operator==` 要求**：`equal` 使用 `operator==` 比较元素。
- **浮点数比较**：直接比较浮点数通常不合适，应传入自定义谓词做误差容忍比较。
- **与 `mismatch` 的选择**：需要知道「哪里不同」时用 `mismatch`，只需要「是否相同」时用 `equal`。
- **性能**：`equal` 会短路返回，遇到第一个不相等就停止。

## 5. 相关算法

- [mismatch](./Mismatch.md)：返回第一个不匹配的位置
- [lexicographical_compare](../Sorting/Lexicographical_compare.md)：按字典序比较
- [is_permutation](../Sorting/Permutation.md)：判断是否为排列关系
- [search](./Search.md)：查找子序列
