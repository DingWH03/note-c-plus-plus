# std::ranges::find_last / find_last_if / find_last_if_not

这三个算法在范围内查找**最后一个**满足条件的元素，返回一个包含迭代器的结构体。它们**只有 Ranges 版本**，没有对应的 `std::` 经典版本。

- `find_last`：查找最后一个等于给定值的元素
- `find_last_if`：查找最后一个使谓词返回 `true` 的元素
- `find_last_if_not`：查找最后一个使谓词返回 `false` 的元素

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<std::forward_iterator I, std::sentinel_for<I> S, class T,
         class Proj = std::identity>
requires std::indirect_binary_predicate<ranges::equal_to,
             std::projected<I, Proj>, const T*>
constexpr ranges::subrange<I> find_last(I first, S last, const T& value, Proj proj = {});

template<ranges::forward_range R, class T, class Proj = std::identity>
requires std::indirect_binary_predicate<ranges::equal_to,
             std::projected<ranges::iterator_t<R>, Proj>, const T*>
constexpr ranges::borrowed_subrange_t<R> find_last(R&& r, const T& value, Proj proj = {});
```

- **迭代器要求**：`ForwardIterator`（因为需要保留「上一次找到的位置」）。
- **复杂度**：至多 \\( O(n) \\) 次比较或谓词调用。
- **返回值**：返回 `ranges::subrange<I>`，其 `begin()` 指向找到的元素，`end()` 指向原范围的末尾。未找到时返回**空子范围**。

返回 `subrange` 而非单个迭代器的好处是：可以直接用 `result.empty()` 判断是否找到，也可以直接用 `result` 作为范围参数传给其它算法。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{3, 1, 4, 1, 5, 9, 1};

    // 找最后一个 1
    auto result = std::ranges::find_last(v, 1);
    if (!result.empty())
        std::cout << *result.begin() << '\n';   // 输出: 1

    // 找最后一个偶数
    auto r2 = std::ranges::find_last_if(v, [](int x) { return x % 2 == 0; });
    std::cout << *r2.begin() << '\n';   // 输出: 4

    // 未找到时返回空子范围
    auto r3 = std::ranges::find_last(v, 100);
    std::cout << std::boolalpha << r3.empty() << '\n';   // true
}
```

### (2) 谓词与投影

支持投影，可以按成员查找：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}, {"Carol", 30}};

auto r = std::ranges::find_last(people, 30, &Person::age);
std::cout << r.begin()->name;   // 输出: Carol
```

### (3) 执行策略

**不支持**执行策略。查找最后一个匹配元素需要顺序扫描，无法有效并行化。

## 4. 注意事项

- **只有 Ranges 版本**：C++23 才引入，且没有 `std::find_last` 经典版本。老编译器上不可用。
- **返回 `subrange` 而非迭代器**：不要写成 `auto it = ranges::find_last(...)` 然后解引用 `it`，应该用 `result.begin()`。
- **未找到返回空范围**：用 `result.empty()` 或 `result.begin() == result.end()` 判断。
- **需要 forward 迭代器**：`input_iterator`（如 `istream_iterator`）不能用，因为无法回退。
- **性能考虑**：查找最后一个元素必须扫描整个范围，无法提前退出。如果频繁需要此操作，考虑反向存储或维护索引。

## 5. 相关算法

- [find / find_if](./Find.md)：查找第一个满足条件的元素
- [find_end](./Find_end.md)：查找最后一个子序列
- [search](./Search.md)：查找子序列的首次出现
- [contains](./Contains.md)：只判断是否存在
