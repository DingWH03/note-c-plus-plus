# 堆操作：make_heap / push_heap / pop_heap / sort_heap / is_heap

这一组算法在**随机访问范围**上维护**最大堆**（max heap）结构。堆是一个完全二叉树，用数组表示，满足父节点不小于子节点。

| 算法 | 含义 |
| :--- | :--- |
| `make_heap` | 把范围转换成堆 |
| `push_heap` | 把末尾元素加入堆 |
| `pop_heap` | 把堆顶（最大元素）移到末尾，并调整剩余部分为堆 |
| `sort_heap` | 把堆转换为升序序列 |
| `is_heap` | 判断范围是否为堆（C++11） |
| `is_heap_until` | 返回最大的堆前缀（C++11） |

`std::priority_queue` 正是基于这组算法实现的。

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class RandomIt>
constexpr void make_heap(RandomIt first, RandomIt last);

template<class RandomIt>
constexpr void push_heap(RandomIt first, RandomIt last);

template<class RandomIt>
constexpr void pop_heap(RandomIt first, RandomIt last);

template<class RandomIt>
constexpr void sort_heap(RandomIt first, RandomIt last);

template<class RandomIt>
constexpr bool is_heap(RandomIt first, RandomIt last);
```

- **迭代器要求**：`RandomAccessIterator`。
- **复杂度**：

| 算法 | 复杂度 |
| :--- | :--- |
| `make_heap` | 至多 \\( 3n \\) 次比较 |
| `push_heap` | 至多 \\( \log n \\) 次比较 |
| `pop_heap` | 至多 \\( 2\log n \\) 次比较 |
| `sort_heap` | 至多 \\( n \log n \\) 次比较 |
| `is_heap` | 至多 \\( n \\) 次比较 |

- **返回值**：除 `is_heap`（返回 `bool`）和 `is_heap_until`（返回迭代器）外，都返回 `void`。

**默认是最大堆**。传入 `std::greater<>` 可以得到**最小堆**。

堆的数组表示：对于索引 `i`，其父节点为 `(i-1)/2`，子节点为 `2i+1` 和 `2i+2`。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{3, 1, 4, 1, 5, 9, 2, 6};

    // 建堆
    std::make_heap(v.begin(), v.end());
    std::cout << "堆顶: " << v.front() << '\n';   // 9

    // 加入新元素：先 push_back，再 push_heap
    v.push_back(10);
    std::push_heap(v.begin(), v.end());
    std::cout << "新堆顶: " << v.front() << '\n';   // 10

    // 弹出堆顶：先 pop_heap，再 pop_back
    std::pop_heap(v.begin(), v.end());   // 堆顶移到末尾
    std::cout << "被弹出的最大值: " << v.back() << '\n';   // 10
    v.pop_back();

    // 堆排序：得到升序序列
    std::sort_heap(v.begin(), v.end());
    // v 变为升序

    // 判断是否为堆
    std::cout << std::boolalpha << std::is_heap(v.begin(), v.end()) << '\n';
}
```

### (2) 谓词与投影

传入 `std::greater<>` 得到最小堆：

```c++
std::vector<int> v{3, 1, 4, 1, 5};
std::make_heap(v.begin(), v.end(), std::greater<>{});
std::cout << v.front();   // 1（最小元素在堆顶）
```

`ranges::make_heap` 等支持投影。

### (3) 执行策略

堆操作**不支持**执行策略。堆的维护是顺序依赖的过程。

## 4. 注意事项

- **`push_heap` 前必须先 `push_back`**：`push_heap` 只调整堆结构，不添加元素。
- **`pop_heap` 后必须 `pop_back`**：`pop_heap` 把堆顶移到末尾，元素还在容器里。
- **`sort_heap` 后不再是堆**：排序后堆性质被破坏。
- **最大堆 vs 最小堆**：默认最大堆；用 `std::greater<>` 得到最小堆。
- **优先用 `priority_queue`**：除非需要直接操作底层容器，否则 `std::priority_queue` 更简洁安全。
- **`is_heap` 是 C++11 起**：C++98 没有。

## 5. 相关算法

- [sort / stable_sort](./Sort.md)：完整排序
- [partial_sort](./Partial_sort.md)：基于堆的部分排序
- [nth_element](./Nth_element.md)：快速选择
- [priority_queue](../../Containers/Priority_queue.md)：基于堆的容器适配器
