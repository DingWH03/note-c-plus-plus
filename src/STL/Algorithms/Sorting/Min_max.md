# std::min / max / minmax / min_element / max_element / minmax_element / clamp

这一组算法用于求最小值、最大值，以及把值限制在边界内。

| 算法 | 含义 | 版本 |
| :--- | :--- | :--- |
| `min` | 返回两个值中的较小者 | C++98 |
| `max` | 返回两个值中的较大者 | C++98 |
| `minmax` | 同时返回较小者和较大者 | C++11 |
| `min_element` | 返回范围中的最小元素 | C++98 |
| `max_element` | 返回范围中的最大元素 | C++98 |
| `minmax_element` | 同时返回最小和最大元素 | C++11 |
| `clamp` | 把值钳制在一对边界之间 | C++17 |

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class T>
constexpr const T& min(const T& a, const T& b);

template<class T>
constexpr const T& max(const T& a, const T& b);

template<class T>
constexpr std::pair<const T&, const T&> minmax(const T& a, const T& b);

template<class ForwardIt>
constexpr ForwardIt min_element(ForwardIt first, ForwardIt last);

template<class ForwardIt>
constexpr ForwardIt max_element(ForwardIt first, ForwardIt last);

template<class ForwardIt>
constexpr std::pair<ForwardIt, ForwardIt> minmax_element(ForwardIt first, ForwardIt last);

template<class T>
constexpr const T& clamp(const T& v, const T& lo, const T& hi);
```

- **迭代器要求**：`ForwardIterator`。
- **复杂度**：`min_element` / `max_element` 至多 \\( n-1 \\) 次比较；`minmax_element` 至多 \\( \lfloor 3(n-1)/2 \rfloor \\) 次比较（比分别调用两个算法更高效）。
- **返回值**：`min` / `max` / `clamp` 返回 `const T&`；`*_element` 返回迭代器；`minmax` 返回 `std::pair`。

**注意**：`min` / `max` 返回的是**引用**，如果参数是临时对象会导致悬垂引用。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    // 两个值
    std::cout << std::min(3, 5) << '\n';    // 3
    std::cout << std::max(3, 5) << '\n';    // 5

    auto [lo, hi] = std::minmax(3, 5);      // lo=3, hi=5

    // 范围
    std::vector<int> v{3, 1, 4, 1, 5, 9, 2, 6};

    auto minIt = std::min_element(v.begin(), v.end());
    auto maxIt = std::max_element(v.begin(), v.end());
    std::cout << "最小: " << *minIt << ", 最大: " << *maxIt << '\n';   // 1, 9

    auto [mn, mx] = std::minmax_element(v.begin(), v.end());
    std::cout << "最小: " << *mn << ", 最大: " << *mx << '\n';   // 1, 9

    // clamp：把值限制在 [0, 100]
    std::cout << std::clamp(150, 0, 100) << '\n';   // 100
    std::cout << std::clamp(-10, 0, 100) << '\n';   // 0
    std::cout << std::clamp(50, 0, 100) << '\n';    // 50
}
```

### (2) 谓词与投影

可以传入自定义比较器；`ranges::` 版本支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}, {"Carol", 35}};

// 找年龄最小的
auto it = std::ranges::min_element(people, {}, &Person::age);
std::cout << it->name;   // Bob

// 自定义比较器：按绝对值比较
std::vector<int> v{-5, 3, -1, 4};
auto it2 = std::min_element(v.begin(), v.end(),
                            [](int a, int b) { return std::abs(a) < std::abs(b); });
std::cout << *it2;   // -1
```

### (3) 执行策略

`min_element`、`max_element`、`minmax_element` 支持 C++17 执行策略（但并行版本可能因分割而失去「返回第一个最值」的保证）。

## 4. 注意事项

- **`min` / `max` 返回引用**：`auto x = std::max(1, 2);` 会得到悬垂引用（C++11 起返回 `const T&`）。应写成 `auto x = std::max<int>(1, 2);` 或先存变量。
- **`min` / `max` 对相等值的处理**：`min(a, b)` 在相等时返回 `a`，`max(a, b)` 在相等时返回 `a`（不是 `b`）。
- **`minmax_element` 更高效**：比分别调用 `min_element` 和 `max_element` 少约 25% 的比较。
- **`clamp` 的前提**：要求 `lo <= hi`，否则行为未定义。
- **`*_element` 返回第一个最值**：有多个相同最值时返回第一个。
- **空范围**：`min_element` / `max_element` 对空范围返回 `last`，解引用是未定义行为。

## 5. 相关算法

- [nth_element](./Nth_element.md)：求第 \\( N \\) 小
- [sort / stable_sort](./Sort.md)：完整排序
- [is_sorted](./Is_sorted.md)：检查是否已排序
- [accumulate](../Numeric/Accumulate.md)：求和
