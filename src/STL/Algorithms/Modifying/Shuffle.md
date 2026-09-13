# std::shuffle

`std::shuffle` 使用给定的随机数生成器**随机重新排列**范围内的元素，产生均匀分布的随机排列。

## 1. 引入

```c++
#include <algorithm>
#include <random>
```

## 2. 原理

```c++
template<class RandomIt, class URBG>
void shuffle(RandomIt first, RandomIt last, URBG&& g);
```

- **迭代器要求**：`RandomAccessIterator`。
- **复杂度**：恰好 \\( n-1 \\) 次交换。
- **返回值**：`void`。

实现基于 **Fisher-Yates 洗牌算法**：从后往前遍历，对每个位置 `i`，随机选择 `[0, i]` 中的一个位置与之交换。

`std::shuffle` 需要显式传入随机数生成器，这比 C++11 前已废弃的 `std::random_shuffle`（用 `rand()`）更可控、质量更高。`random_shuffle` 已在 C++17 中被移除。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <random>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    // 用随机设备初始化生成器
    std::random_device rd;
    std::mt19937 g(rd());

    std::shuffle(v.begin(), v.end(), g);
    // v 被随机打乱，例如 {3,1,5,2,4}
}
```

### (2) 谓词与投影

`shuffle` **没有谓词或投影参数**。

### (3) 执行策略

`shuffle` **不支持**执行策略。随机排列必须按顺序生成，无法并行化。

## 4. 注意事项

- **生成器质量很重要**：不要用 `rand()` 或 `std::default_random_engine` 的默认种子，推荐 `std::mt19937` 配合 `std::random_device`。
- **需要随机访问迭代器**：`std::list` 的迭代器不可用，需要先拷贝到 `vector`。
- **`random_shuffle` 已移除**：C++17 起不再可用，应改用 `shuffle`。
- **可复现性**：用固定种子初始化生成器可以得到可复现的洗牌结果，便于测试。
- **`std::ranges::shuffle`**：C++20 起提供，支持范围参数。

## 5. 相关算法

- [sample](./Sample.md)：随机抽取 \\( N \\) 个元素
- [next_permutation / prev_permutation](../Sorting/Permutation.md)：按字典序生成排列
- [sort / stable_sort](../Sorting/Sort.md)：排序
- [reverse / reverse_copy](./Reverse.md)：反转顺序
