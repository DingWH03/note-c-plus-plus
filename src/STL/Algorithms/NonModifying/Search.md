# std::search / std::search_n

这两个算法用于搜索子序列：

- `search`：在范围内查找子序列 `[s_first, s_last)` **第一次**出现的位置
- `search_n`：查找某个元素**连续出现 \\( N \\) 次**的第一次出现位置

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt1, class ForwardIt2>
constexpr ForwardIt1 search(ForwardIt1 first, ForwardIt1 last,
                            ForwardIt2 s_first, ForwardIt2 s_last);

template<class ForwardIt, class Size, class T>
constexpr ForwardIt search_n(ForwardIt first, ForwardIt last, Size count, const T& value);
```

- **迭代器要求**：`ForwardIterator`。
- **复杂度**：`search` 至多 \\( O(S \cdot N) \\) 次比较；`search_n` 至多 \\( O(N) \\) 次比较。
- **返回值**：指向匹配子序列的起始位置；未找到时返回 `last`。
- **空子序列**：`search` 对空子序列返回 `first`。

C++17 起 `search` 增加了接受 `Searcher` 策略的重载（如 `std::boyer_moore_searcher`），可以显著加速大文本搜索。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 1, 2, 3};
    std::vector<int> sub{1, 2, 3};

    // 查找子序列第一次出现
    auto it = std::search(v.begin(), v.end(), sub.begin(), sub.end());
    std::cout << "起始索引: " << (it - v.begin()) << '\n';   // 0

    // 查找连续 3 个 2
    std::vector<int> v2{1, 2, 2, 2, 3};
    auto it2 = std::search_n(v2.begin(), v2.end(), 3, 2);
    std::cout << "起始索引: " << (it2 - v2.begin()) << '\n';   // 1
}
```

### (2) 谓词与投影

```c++
// 忽略大小写查找子串
std::string text = "Hello World";
std::string pat = "WORLD";
auto it = std::search(text.begin(), text.end(), pat.begin(), pat.end(),
                      [](char a, char b) {
                          return std::tolower(a) == std::tolower(b);
                      });
std::cout << (it != text.end() ? "found" : "not found");   // found
```

### (3) 执行策略

`search` 和 `search_n` 支持 C++17 执行策略：

```c++
#include <execution>

auto it = std::search(std::execution::par, v.begin(), v.end(),
                      sub.begin(), sub.end());
```

但并行版本会失去短路特性，可能扫描整个范围，对小数据反而更慢。

## 4. 注意事项

- **`search` vs `find_end`**：`search` 找第一次，`find_end` 找最后一次。
- **大文本搜索用 Searcher**：C++17 的 `boyer_moore_searcher`、`boyer_moore_horspool_searcher` 可大幅提升性能。
- **`search_n` 的 `count` 必须为正**：如果 `count <= 0`，行为未定义。
- **空子序列**：`search` 对空子序列返回 `first`，容易忽略。
- **`std::string::find` 更直接**：对字符串搜索，直接用 `std::string::find` 通常更简洁。

## 5. 相关算法

- [find_end](./Find_end.md)：查找子序列的最后一次出现
- [find_first_of](./Find_first_of.md)：查找集合中的任意一个元素
- [contains](./Contains.md)：判断是否包含子范围
- [starts_with / ends_with](./Starts_ends_with.md)：判断前缀/后缀
- [find / find_if](./Find.md)：查找单个元素
