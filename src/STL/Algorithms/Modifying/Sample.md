# std::sample

C++17 引入的 `std::sample` 从序列中**随机选取 \\( N \\) 个元素**，写入目标范围。选取是无放回的，且每个元素被选中的概率相同。

## 1. 引入

```c++
#include <algorithm>
#include <random>
```

## 2. 原理

```c++
template<class PopulationIt, class SampleIt, class Distance, class URBG>
SampleIt sample(PopulationIt first, PopulationIt last, SampleIt out, Distance n, URBG&& g);
```

- **迭代器要求**：输入为 `InputIterator`，输出为 `OutputIterator`。
- **复杂度**：\\( O(n) \\) 次随机数生成（当输入为 forward 迭代器时）。
- **返回值**：指向目标范围中最后一个被写入元素之后的位置。

实现基于 **reservoir sampling（蓄水池抽样）** 的变体，因此即使输入是 `InputIterator`（不知道总长度）也能保证均匀性。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <random>
#include <vector>

int main()
{
    std::vector<int> population{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

    std::random_device rd;
    std::mt19937 g(rd());

    // 随机抽取 3 个元素
    std::vector<int> picked;
    std::sample(population.begin(), population.end(),
                std::back_inserter(picked), 3, g);
    // picked 包含 3 个随机元素，例如 {7, 2, 9}
}
```

### (2) 谓词与投影

`sample` **没有谓词或投影参数**。如果需要从满足条件的元素中抽样，应先过滤：

```c++
std::vector<int> evens;
std::copy_if(population.begin(), population.end(), std::back_inserter(evens),
             [](int x) { return x % 2 == 0; });

std::vector<int> picked;
std::sample(evens.begin(), evens.end(), std::back_inserter(picked), 2, g);
```

### (3) 执行策略

`sample` **不支持**执行策略。抽样必须按顺序进行。

## 4. 注意事项

- **`n` 不能超过总体大小**：如果 `n > size`，只会抽取 `size` 个元素（不会报错）。
- **无放回抽样**：每个元素最多被选中一次。需要放回抽样时应循环调用 `std::uniform_int_distribution`。
- **目标空间必须足够**：不检查目标范围大小。
- **生成器质量**：同样推荐 `std::mt19937` 配合 `std::random_device`。
- **`std::ranges::sample`**：C++20 起提供。

## 5. 相关算法

- [shuffle](./Shuffle.md)：随机打乱整个范围
- [copy / copy_if / copy_n](./Copy.md)：按条件复制
- [generate / generate_n](./Generate.md)：用随机数填充
- [random_shuffle](./Shuffle.md)：已移除的旧算法
