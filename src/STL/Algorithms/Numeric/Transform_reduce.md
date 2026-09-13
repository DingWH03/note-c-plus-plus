# std::transform_reduce

C++17 引入的 `std::transform_reduce` 先对元素应用变换，再进行归约。它相当于 `transform` + `reduce` 的组合，支持并行。

## 1. 引入

```c++
#include <numeric>
```

## 2. 原理

```c++
// 一元版本：变换后归约
template<class InputIt, class T, class BinaryOp, class UnaryOp>
constexpr T transform_reduce(InputIt first, InputIt last, T init,
                             BinaryOp reduce, UnaryOp transform);

// 二元版本：两个范围先做变换，再归约（相当于广义内积）
template<class InputIt1, class InputIt2, class T>
constexpr T transform_reduce(InputIt1 first1, InputIt1 last1, InputIt2 first2, T init);
```

- **迭代器要求**：`InputIterator`（并行版本需要 `ForwardIterator`）。
- **复杂度**：\\( O(n) \\) 次变换和归约调用。
- **返回值**：归约结果。

二元版本等价于 `inner_product`，但允许乱序，因此可以并行。

## 3. 用法

### (1) 基本用法

```c++
#include <iostream>
#include <numeric>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4};

    // 先平方再求和：1+4+9+16 = 30
    int sumSq = std::transform_reduce(v.begin(), v.end(), 0,
                                      std::plus<>{},
                                      [](int x) { return x * x; });
    std::cout << sumSq << '\n';   // 30

    // 二元版本：内积
    std::vector<int> a{1, 2, 3}, b{4, 5, 6};
    int dot = std::transform_reduce(a.begin(), a.end(), b.begin(), 0);
    std::cout << dot << '\n';   // 32

    // 统计偶数个数（把条件映射为 0/1）
    int evens = std::transform_reduce(v.begin(), v.end(), 0, std::plus<>{},
                                      [](int x) { return x % 2 == 0 ? 1 : 0; });
    std::cout << evens << '\n';   // 2
}
```

### (2) 谓词与投影

`transform_reduce` 接受变换和归约两个可调用对象，没有投影参数。

### (3) 执行策略

支持 C++17 执行策略，这是它相对 `inner_product` 的主要优势：

```c++
#include <execution>

int sumSq = std::transform_reduce(std::execution::par,
                                  v.begin(), v.end(), 0,
                                  std::plus<>{},
                                  [](int x) { return x * x; });
```

## 4. 注意事项

- **归约操作必须满足交换律和结合律**（并行时）。
- **二元版本的第二个范围必须足够长**。
- **初始值类型决定返回类型**。
- **可替代 `count_if` 的并行版本**：如上面统计偶数的例子。
- **C++20 起 `ranges::` 版本**：目前还没有 `ranges::transform_reduce`。

## 5. 相关算法

- [reduce](./Reduce.md)：不带变换的归约
- [inner_product](./Inner_product.md)：顺序版本的内积
- [transform_scan](./Transform_scan.md)：先变换再扫描
- [transform](../Modifying/Transform.md)：只做变换
- [count / count_if](../NonModifying/Count.md)：统计数量
