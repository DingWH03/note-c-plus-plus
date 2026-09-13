# std::find_end

`std::find_end` 在范围 `[first, last)` 中查找子序列 `[s_first, s_last)` **最后一次**出现的位置，返回该子序列起始处的迭代器。如果找不到，返回 `last`。

它与 `search` 的区别在于：`search` 找**第一次**出现，`find_end` 找**最后一次**出现。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt1, class ForwardIt2>
constexpr ForwardIt1 find_end(ForwardIt1 first, ForwardIt1 last,
                              ForwardIt2 s_first, ForwardIt2 s_last);

template<class ForwardIt1, class ForwardIt2, class BinaryPred>
constexpr ForwardIt1 find_end(ForwardIt1 first, ForwardIt1 last,
                              ForwardIt2 s_first, ForwardIt2 s_last, BinaryPred p);
```

- **迭代器要求**：`ForwardIterator`。
- **复杂度**：至多 \\( O(S \cdot N) \\) 次比较，其中 \\( S \\) 为子序列长度，\\( N \\) 为主范围长度。
- **空子序列**：如果 `[s_first, s_last)` 为空，返回 `last`。
- **返回值**：指向最后一次匹配的子序列首元素；无匹配时返回 `last`。

`ranges::find_end` 支持投影，返回 `ranges::subrange`。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 1, 2, 3, 4};
    std::vector<int> sub{1, 2, 3};

    // 查找子序列最后一次出现
    auto it = std::find_end(v.begin(), v.end(), sub.begin(), sub.end());
    if (it != v.end())
        std::cout << "起始索引: " << (it - v.begin()) << '\n';   // 3

    // 找不到的情况
    std::vector<int> sub2{9, 9};
    auto it2 = std::find_end(v.begin(), v.end(), sub2.begin(), sub2.end());
    std::cout << std::boolalpha << (it2 == v.end()) << '\n';   // true
}
```

### (2) 谓词与投影

可以传入二元谓词自定义相等判断：

```c++
// 忽略大小写地查找
std::string text = "Hello World hello";
std::string pat = "HELLO";
auto it = std::find_end(text.begin(), text.end(), pat.begin(), pat.end(),
                        [](char a, char b) {
                            return std::tolower(a) == std::tolower(b);
                        });
// 指向 "hello" 的起始位置
```

### (3) 执行策略

`find_end` **不支持**执行策略。它需要确定「最后一次」出现，无法有效并行化。

## 4. 注意事项

- **复杂度较高**：最坏情况是 \\( O(S \cdot N) \\)，在大文本中查找长子串时可能很慢。这类场景应考虑更高效的字符串搜索算法（如 KMP、Boyer-Moore）。
- **空子序列返回 `last`**：这一点容易忽略，使用前最好检查子序列非空。
- **与 `search` 的混淆**：`search` 找第一次，`find_end` 找最后一次，名字容易记反。
- **需要 forward 迭代器**：`input_iterator` 不可用。

## 5. 相关算法

- [search / search_n](./Search.md)：查找子序列的**第一次**出现
- [find_first_of](./Find_first_of.md)：查找一组元素中的任意一个
- [find / find_if](./Find.md)：查找单个元素
- [find_last](./Find_last.md)：查找最后一个满足条件的单个元素
