# std::unique / std::unique_copy

这两个算法处理**连续重复**元素：

- `unique`：逻辑移除范围内连续重复的元素，只保留每组重复的第一个
- `unique_copy`：复制时省略连续重复的元素

**重要**：它们只处理**相邻**的重复元素。要移除所有重复，必须先排序。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt>
constexpr ForwardIt unique(ForwardIt first, ForwardIt last);

template<class ForwardIt, class BinaryPred>
constexpr ForwardIt unique(ForwardIt first, ForwardIt last, BinaryPred p);

template<class InputIt, class OutputIt>
constexpr OutputIt unique_copy(InputIt first, InputIt last, OutputIt d_first);
```

- **迭代器要求**：`unique` 需要 `ForwardIterator`；`unique_copy` 输入 `InputIterator`、输出 `OutputIterator`。
- **复杂度**：恰好 \\( n-1 \\) 次比较。
- **返回值**：`unique` 返回新的逻辑末尾；`unique_copy` 返回目标范围的末尾。

和 `remove` 一样，`unique` 只是逻辑移除，需要配合 `erase`：

```c++
v.erase(std::unique(v.begin(), v.end()), v.end());
```

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    // 只移除相邻重复
    std::vector<int> v{1, 1, 2, 2, 2, 3, 1, 1};
    v.erase(std::unique(v.begin(), v.end()), v.end());
    // v = {1,2,3,1}（注意最后的 1 保留了，因为它不与前面的 3 相邻）

    // 移除所有重复：先排序
    std::vector<int> v2{3, 1, 2, 1, 3, 2};
    std::sort(v2.begin(), v2.end());
    v2.erase(std::unique(v2.begin(), v2.end()), v2.end());
    // v2 = {1,2,3}

    // unique_copy
    std::vector<int> src{1, 1, 2, 3, 3, 3, 4};
    std::vector<int> dst;
    std::unique_copy(src.begin(), src.end(), std::back_inserter(dst));
    // dst = {1,2,3,4}
}
```

### (2) 谓词与投影

二元谓词自定义「相等」判断：

```c++
// 忽略大小写去除连续重复
std::string s = "aAbBcC";
s.erase(std::unique(s.begin(), s.end(),
                    [](char a, char b) {
                        return std::tolower(a) == std::tolower(b);
                    }),
        s.end());
// s = "aAbBcC"（因为 a 和 A 相邻，A 被移除；结果 "aAbBcC" -> "abc"? 实际为 "aAbBcC" 的相邻比较）
```

`ranges::unique` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"A", 30}, {"B", 30}, {"C", 25}};

people.erase(std::ranges::unique(people, {}, &Person::age).begin(), people.end());
// 相邻年龄相同的被去除，保留 {A, C}
```

### (3) 执行策略

`unique` 和 `unique_copy` 支持 C++17 执行策略。

## 4. 注意事项

- **只处理相邻重复**：这是最大的坑。要移除所有重复必须先排序。
- **不改变容器大小**：需要配合 `erase`。
- **`unique` 后元素顺序**：保留每组重复的第一个元素，顺序不变。
- **排序会打乱顺序**：如果顺序重要，不能用「排序 + unique」的方式去重，应改用哈希表或 `std::set`。
- **对 `list` 用成员函数**：`std::list::unique` 会真正删除元素。

## 5. 相关算法

- [remove / remove_if](./Remove.md)：按值或谓词移除
- [adjacent_find](../NonModifying/Adjacent_find.md)：查找相邻重复
- [sort / stable_sort](../Sorting/Sort.md)：排序后去重
- [set_operations](../Sorting/Set_operations.md)：有序范围上的集合运算
