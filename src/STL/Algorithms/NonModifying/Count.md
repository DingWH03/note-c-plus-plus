# std::count / std::count_if

这两个算法统计范围内满足条件的元素数量：

- `count`：统计等于给定值的元素个数
- `count_if`：统计使谓词返回 `true` 的元素个数

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class T>
constexpr typename iterator_traits<InputIt>::difference_type
    count(InputIt first, InputIt last, const T& value);

template<class InputIt, class UnaryPred>
constexpr typename iterator_traits<InputIt>::difference_type
    count_if(InputIt first, InputIt last, UnaryPred p);
```

- **迭代器要求**：`InputIterator`。
- **复杂度**：恰好 \\( O(n) \\) 次比较或谓词调用。
- **返回值**：`difference_type`（有符号整数类型），不是 `size_t`。
- **空范围**：返回 `0`。

`ranges::count` / `ranges::count_if` 支持投影。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 2, 4, 2, 5};

    // 统计值为 2 的元素个数
    auto n1 = std::count(v.begin(), v.end(), 2);
    std::cout << n1 << '\n';   // 3

    // 统计偶数个数
    auto n2 = std::count_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
    std::cout << n2 << '\n';   // 4
}
```

### (2) 谓词与投影

`ranges::count` 支持投影，可以按成员统计：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}, {"Carol", 30}};

// 统计年龄为 30 的人数
auto n = std::ranges::count(people, 30, &Person::age);   // 2
```

### (3) 执行策略

`count` 和 `count_if` **不支持**执行策略。如果需要并行统计，可以用 `transform_reduce` 把「是否满足条件」映射为 0/1 后求和。

```c++
// 并行统计偶数的替代写法
auto n = std::transform_reduce(std::execution::par,
                               v.begin(), v.end(), 0, std::plus<>{},
                               [](int x) { return x % 2 == 0 ? 1 : 0; });
```

## 4. 注意事项

- **返回类型是有符号的**：`difference_type` 而非 `size_t`，与 `size()` 比较时注意符号问题，可能触发编译器警告。
- **`count` 用 `operator==`**：自定义类型需要提供 `operator==`。
- **不要与 `count_if` 混淆**：`count` 接受值，`count_if` 接受谓词。
- **大数据量考虑并行**：如上所示，可用 `transform_reduce` 替代以获得并行加速。

## 5. 相关算法

- [all_of / any_of / none_of](./All_any_none_of.md)：判断是否全部/存在/没有满足条件
- [find / find_if](./Find.md)：查找第一个满足条件的元素
- [transform_reduce](../Numeric/Transform_reduce.md)：并行归约统计
- [equal](./Equal.md)：比较两个范围
