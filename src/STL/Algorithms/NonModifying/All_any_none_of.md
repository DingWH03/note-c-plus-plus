# std::all_of / std::any_of / std::none_of

这三个算法用同一个一元谓词 `p` 检查范围内的元素，分别回答三个问题：

- `all_of`：是否**所有**元素都满足 `p`？
- `any_of`：是否**存在**元素满足 `p`？
- `none_of`：是否**没有**元素满足 `p`？

它们都是**短路求值**的，一旦结论确定就立即返回，不会继续扫描剩余元素。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class UnaryPred>
constexpr bool all_of(InputIt first, InputIt last, UnaryPred p);

template<class InputIt, class UnaryPred>
constexpr bool any_of(InputIt first, InputIt last, UnaryPred p);

template<class InputIt, class UnaryPred>
constexpr bool none_of(InputIt first, InputIt last, UnaryPred p);
```

- **迭代器要求**：`InputIterator`。
- **复杂度**：至多 \\( O(n) \\) 次谓词调用，实际调用次数取决于何时能得出结论。
- **空范围**：`all_of` 和 `none_of` 对空范围返回 `true`（空真命题），`any_of` 返回 `false`。

三者的逻辑关系如下：

| 算法 | 等价写法 | 空范围结果 |
| :--- | :--- | :--- |
| `all_of` | `!any_of(..., !p)` | `true` |
| `any_of` | `!none_of(...)` | `false` |
| `none_of` | `!any_of(...)` | `true` |

`ranges::` 版本额外支持**投影**，可以只对元素的某个成员进行判断。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{2, 4, 6, 8};

    bool allEven = std::all_of(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
    bool anyOdd  = std::any_of(v.begin(), v.end(), [](int x) { return x % 2 != 0; });
    bool noneNeg = std::none_of(v.begin(), v.end(), [](int x) { return x < 0; });

    std::cout << std::boolalpha
              << allEven << '\n'   // true
              << anyOdd  << '\n'   // false
              << noneNeg << '\n';  // true
}
```

### (2) 谓词与投影

`ranges::` 版本支持投影，可以直接对成员做判断：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}};

// 是否所有成年人都年满 18 岁
bool allAdult = std::ranges::all_of(people, [](int age) { return age >= 18; },
                                    &Person::age);   // true
```

### (3) 执行策略

这三个算法**不支持**执行策略。原因是它们需要短路求值，而并行执行无法保证「一旦得出结论就停止」，因此标准没有提供并行版本。

## 4. 注意事项

- **空范围的真假**：`all_of` 与 `none_of` 对空范围返回 `true`，这符合数学上的空真（vacuous truth），但容易与直觉不符，使用前最好先确认范围非空。
- **谓词不应有副作用**：标准不保证谓词被调用的次数，也不保证调用顺序。
- **`none_of` 不等于 `!all_of`**：`none_of(p)` 等价于 `!any_of(p)`，而不是 `!all_of(p)`。
- **短路特性**：如果谓词开销大，把最可能失败的判断放在前面可以提前结束。

## 5. 相关算法

- [find / find_if](./Find.md)：找出第一个满足条件的元素
- [count / count_if](./Count.md)：统计满足条件的元素数量
- [equal](./Equal.md)：比较两个范围是否相同
- [search](./Search.md)：查找子序列
