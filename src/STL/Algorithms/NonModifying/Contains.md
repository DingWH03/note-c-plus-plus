# std::ranges::contains / std::ranges::contains_subrange

这两个算法用于判断范围是否包含给定内容，**只有 Ranges 版本**：

- `ranges::contains`：判断范围是否包含给定元素
- `ranges::contains_subrange`：判断范围是否包含给定子范围

它们是 `find(...) != end()` 和 `search(...) != end()` 的语义化封装，代码更清晰。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<std::input_iterator I, std::sentinel_for<I> S, class T, class Proj = std::identity>
requires std::indirect_binary_predicate<ranges::equal_to,
             std::projected<I, Proj>, const T*>
constexpr bool contains(I first, S last, const T& value, Proj proj = {});

template<ranges::input_range R, class T, class Proj = std::identity>
requires std::indirect_binary_predicate<ranges::equal_to,
             std::projected<ranges::iterator_t<R>, Proj>, const T*>
constexpr bool contains(R&& r, const T& value, Proj proj = {});
```

- **迭代器要求**：`InputIterator`。
- **复杂度**：`contains` 至多 \\( O(n) \\) 次比较；`contains_subrange` 至多 \\( O(S \cdot N) \\)。
- **返回值**：布尔值。
- **空子范围**：`contains_subrange` 对空子范围返回 `true`。

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
              << std::ranges::contains(v, 3) << '\n'    // true
              << std::ranges::contains(v, 9) << '\n';   // false

    std::vector<int> sub{3, 4};
    std::cout << std::ranges::contains_subrange(v, sub) << '\n';   // true
}
```

### (2) 谓词与投影

支持投影，可以按成员判断：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}};

bool has30 = std::ranges::contains(people, 30, &Person::age);   // true
```

### (3) 执行策略

**不支持**执行策略，因为需要短路返回。

## 4. 注意事项

- **只有 Ranges 版本**：C++23 才引入，且必须用 `std::ranges::contains`，没有 `std::contains`。
- **对有序范围仍用二分**：`contains` 是线性查找，有序范围应改用 `std::ranges::binary_search`。
- **`contains` 与 `find` 的选择**：只需要「是否存在」时用 `contains`，需要「位置」时用 `find`。
- **关联容器也有 `contains`**：C++20 起 `std::set`、`std::map` 等关联容器有成员函数 `contains`，性能是 \\( O(\log n) \\) 或平均 \\( O(1) \\)，比 `ranges::contains` 更快。

## 5. 相关算法

- [find / find_if](./Find.md)：查找元素位置
- [search / search_n](./Search.md)：查找子序列位置
- [starts_with / ends_with](./Starts_ends_with.md)：判断前缀/后缀
- [count / count_if](./Count.md)：统计元素数量
