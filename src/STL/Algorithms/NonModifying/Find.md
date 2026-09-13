# std::find / std::find_if / std::find_if_not

这三个算法在范围内查找**第一个**满足条件的元素，返回指向它的迭代器；如果找不到，返回 `last`。

- `find`：查找等于给定值的元素
- `find_if`：查找使谓词返回 `true` 的元素
- `find_if_not`：查找使谓词返回 `false` 的元素（C++11 起）

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class T>
constexpr InputIt find(InputIt first, InputIt last, const T& value);

template<class InputIt, class UnaryPred>
constexpr InputIt find_if(InputIt first, InputIt last, UnaryPred p);

template<class InputIt, class UnaryPred>
constexpr InputIt find_if_not(InputIt first, InputIt last, UnaryPred q);
```

- **迭代器要求**：`InputIterator`。
- **复杂度**：至多 \\( O(n) \\) 次比较或谓词调用。
- **返回值**：指向第一个匹配元素的迭代器；无匹配时返回 `last`。

由于是**线性查找**，对已排序范围应改用 `lower_bound` / `binary_search` 等二分算法（\\( O(\log n) \\)）。

`ranges::` 版本支持**投影**，返回类型仍是迭代器（`borrowed_iterator_t`），不是结构体。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{3, 1, 4, 1, 5, 9};

    auto it = std::find(v.begin(), v.end(), 4);
    if (it != v.end())
        std::cout << "找到: " << *it << '\n';   // 找到: 4

    // find_if：找第一个偶数
    auto it2 = std::find_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
    std::cout << *it2 << '\n';   // 4

    // find_if_not：找第一个非奇数（即偶数）
    auto it3 = std::find_if_not(v.begin(), v.end(), [](int x) { return x % 2 != 0; });
    std::cout << *it3 << '\n';   // 4

    // 未找到时返回 end()
    if (std::find(v.begin(), v.end(), 100) == v.end())
        std::cout << "未找到 100\n";
}
```

### (2) 谓词与投影

`find` 使用 `operator==` 比较；`find_if` / `find_if_not` 接受一元谓词。`ranges::` 版本支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}};

// 按成员查找
auto it = std::ranges::find(people, 25, &Person::age);
std::cout << it->name;   // Bob
```

### (3) 执行策略

`find`、`find_if`、`find_if_not` **不支持**执行策略。它们需要返回第一个匹配位置，短路语义与并行执行不兼容。

## 4. 注意事项

- **必须检查返回值**：解引用 `end()` 是未定义行为，使用前务必判断 `it != last`。
- **`find` 用 `operator==`**：对自定义类型需要提供 `operator==`，否则编译失败。
- **对已排序范围用二分**：`std::find` 是 \\( O(n) \\)，已排序时应改用 `std::lower_bound`（\\( O(\log n) \\)）。
- **C++20 的替代**：`std::ranges::contains` 更适合只判断「是否存在」的场景，无需处理迭代器。
- **`find_if_not` 的谓词含义**：它查找的是**使谓词为假**的元素，不要与 `find_if` 混淆。

## 5. 相关算法

- [find_last](./Find_last.md)：查找最后一个满足条件的元素
- [find_first_of](./Find_first_of.md)：查找一组元素中的任意一个
- [adjacent_find](./Adjacent_find.md)：查找相邻的重复元素
- [search](./Search.md)：查找子序列
- [count / count_if](./Count.md)：统计满足条件的元素数量
- [二分查找](../Sorting/Binary_search.md)：在有序范围上查找
