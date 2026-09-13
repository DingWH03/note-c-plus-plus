# std::partition_copy

`std::partition_copy` 按谓词把输入范围的元素**分别复制**到两个输出范围：满足谓词的写入第一个，不满足的写入第二个。原范围不变。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class OutputIt1, class OutputIt2, class UnaryPred>
constexpr std::pair<OutputIt1, OutputIt2>
    partition_copy(InputIt first, InputIt last,
                   OutputIt1 d_first_true, OutputIt2 d_first_false, UnaryPred p);
```

- **迭代器要求**：输入 `InputIterator`，输出 `OutputIterator`。
- **复杂度**：恰好 \\( n \\) 次谓词调用和赋值。
- **返回值**：`std::pair`，分别指向两个输出范围中最后一个写入元素之后的位置。

`ranges::partition_copy` 返回 `ranges::partition_copy_result`，并支持投影。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5, 6};

    std::vector<int> evens, odds;
    std::partition_copy(v.begin(), v.end(),
                        std::back_inserter(evens),
                        std::back_inserter(odds),
                        [](int x) { return x % 2 == 0; });
    // evens = {2,4,6}, odds = {1,3,5}
    // v 不变

    // 也可以预先分配空间
    std::vector<int> e2(v.size()), o2(v.size());
    auto [endE, endO] = std::partition_copy(v.begin(), v.end(),
                                            e2.begin(), o2.begin(),
                                            [](int x) { return x % 2 == 0; });
    e2.erase(endE, e2.end());
    o2.erase(endO, o2.end());
}
```

### (2) 谓词与投影

`ranges::partition_copy` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 17}};

std::vector<Person> adults, minors;
std::ranges::partition_copy(people,
                            std::back_inserter(adults),
                            std::back_inserter(minors),
                            [](int age) { return age >= 18; },
                            &Person::age);
```

### (3) 执行策略

`partition_copy` 支持 C++17 执行策略。

## 4. 注意事项

- **两个输出范围都必须足够大**：不检查大小，用 `back_inserter` 最安全。
- **输出范围不能与输入重叠**。
- **保持相对顺序**：`partition_copy` 是稳定的，两组内部的相对顺序与原范围一致。
- **与 `partition` 的区别**：`partition` 原地分区，`partition_copy` 复制分区。

## 5. 相关算法

- [partition / stable_partition](./Partition.md)：原地分区
- [copy / copy_if / copy_n](../Modifying/Copy.md)：按条件复制
- [remove_copy](../Modifying/Remove_copy.md)：复制时省略元素
- [is_partitioned / partition_point](./Is_partitioned.md)：判断分区
