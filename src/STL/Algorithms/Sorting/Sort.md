# std::sort / std::stable_sort

这两个算法对范围内的元素排序：

- `sort`：不保证相等元素的相对顺序
- `stable_sort`：保持相等元素的相对顺序

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class RandomIt>
constexpr void sort(RandomIt first, RandomIt last);

template<class RandomIt, class Compare>
constexpr void sort(RandomIt first, RandomIt last, Compare comp);

template<class RandomIt>
void stable_sort(RandomIt first, RandomIt last);
```

- **迭代器要求**：`RandomAccessIterator`。
- **复杂度**：
  - `sort`：\\( O(n \log n) \\) 次比较（典型实现为 **Introsort**：快排 + 堆排 + 插入排序的混合）
  - `stable_sort`：有足够额外内存时 \\( O(n \log n) \\)，否则 \\( O(n \log^2 n) \\)
- **返回值**：`void`。
- **比较器要求**：必须是**严格弱序**（strict weak ordering），否则行为未定义。

`sort` 不是稳定排序，如果需要稳定性必须用 `stable_sort`。`std::list` 和 `std::forward_list` 有成员函数 `sort`（基于归并排序，天然稳定）。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{3, 1, 4, 1, 5, 9, 2, 6};

    // 升序
    std::sort(v.begin(), v.end());

    // 降序
    std::sort(v.begin(), v.end(), std::greater<>{});

    // 自定义比较器
    std::sort(v.begin(), v.end(), [](int a, int b) { return a > b; });

    // stable_sort 保持相等元素的相对顺序
    std::vector<std::pair<int, std::string>> items{
        {2, "a"}, {1, "b"}, {2, "c"}, {1, "d"}
    };
    std::stable_sort(items.begin(), items.end(),
                     [](const auto& x, const auto& y) { return x.first < y.first; });
    // 结果为 {1,b}, {1,d}, {2,a}, {2,c}（同 key 保持原顺序）
}
```

### (2) 谓词与投影

`ranges::sort` 支持投影，可以直接按成员排序：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}, {"Carol", 35}};

// 按年龄升序
std::ranges::sort(people, {}, &Person::age);

// 按年龄降序
std::ranges::sort(people, std::greater<>{}, &Person::age);

// 多级排序：先按年龄，再按姓名
std::ranges::sort(people, [](const Person& a, const Person& b) {
    return std::tie(a.age, a.name) < std::tie(b.age, b.name);
});
```

### (3) 执行策略

`sort` 支持 C++17 执行策略；`stable_sort` 也支持，但并行版本的稳定性实现代价较高。

```c++
#include <execution>

std::sort(std::execution::par, v.begin(), v.end());
```

C++26 起 `ranges::sort` 也支持执行策略。

## 4. 注意事项

- **比较器必须是严格弱序**：使用 `<=` 或 `>=` 而非 `<` 或 `>` 是经典错误，会导致越界或死循环。
- **`sort` 不稳定**：相等元素的相对顺序可能改变。
- **迭代器失效**：排序会交换元素，所有指向元素的迭代器、指针、引用都会失效（元素本身没被销毁，但位置变了）。
- **`std::list` 用成员 `sort`**：`std::sort` 需要随机访问迭代器，不能用于 `list`。
- **`std::greater<>` 而非 `std::greater<int>`**：C++14 起的透明比较器更通用。
- **性能**：`sort` 通常比 `stable_sort` 快，且不需要额外内存。

## 5. 相关算法

- [partial_sort](./Partial_sort.md)：只排序前 \\( N \\) 个
- [nth_element](./Nth_element.md)：只定位第 \\( N \\) 个
- [is_sorted](./Is_sorted.md)：检查是否已排序
- [二分查找](./Binary_search.md)：在有序范围上查找
- [merge / inplace_merge](./Merge.md)：合并有序范围
