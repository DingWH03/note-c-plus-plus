# std::transform

`std::transform` 对输入范围内的元素应用函数，并把结果写入目标范围。它支持一元和二元两种形式：

- **一元**：`transform(first, last, d_first, unary_op)`，把 `op(*it)` 写入目标
- **二元**：`transform(first1, last1, first2, d_first, binary_op)`，把 `op(*it1, *it2)` 写入目标

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class OutputIt, class UnaryOp>
constexpr OutputIt transform(InputIt first, InputIt last, OutputIt d_first, UnaryOp op);

template<class InputIt1, class InputIt2, class OutputIt, class BinaryOp>
constexpr OutputIt transform(InputIt1 first1, InputIt1 last1, InputIt2 first2,
                             OutputIt d_first, BinaryOp op);
```

- **迭代器要求**：输入为 `InputIterator`，输出为 `OutputIterator`。
- **复杂度**：恰好 \\( n \\) 次函数调用。
- **返回值**：指向目标范围中最后一个被写入元素之后的位置。

`transform` 允许**原地变换**（目标范围等于源范围），这是它比 `copy` 更灵活的地方。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 4, 5};

    // 一元：每个元素乘 2
    std::vector<int> out(v.size());
    std::transform(v.begin(), v.end(), out.begin(),
                   [](int x) { return x * 2; });   // out = {2,4,6,8,10}

    // 原地变换
    std::transform(v.begin(), v.end(), v.begin(),
                   [](int x) { return x * x; });   // v = {1,4,9,16,25}

    // 二元：两个范围逐元素相加
    std::vector<int> a{1, 2, 3}, b{10, 20, 30};
    std::vector<int> sum(3);
    std::transform(a.begin(), a.end(), b.begin(), sum.begin(),
                   std::plus<>{});   // sum = {11,22,33}
}
```

### (2) 谓词与投影

`transform` 接受一元或二元操作，`ranges::transform` 额外支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 25}};

std::vector<int> ages;
std::ranges::transform(people, std::back_inserter(ages), &Person::age);
// ages = {30, 25}
```

配合 `std::toupper` 做大小写转换：

```c++
std::string s = "hello";
std::ranges::transform(s, s.begin(),
                       [](char c) { return std::toupper(c); });   // "HELLO"
```

### (3) 执行策略

`transform` 支持 C++17 执行策略：

```c++
#include <execution>

std::transform(std::execution::par, v.begin(), v.end(), out.begin(),
               [](int x) { return x * 2; });
```

注意：并行执行时函数对象必须是**线程安全**的，不能有数据竞争。

## 4. 注意事项

- **原地变换是允许的**：目标可以等于源，这与 `copy` 不同。但二元版本中如果目标与第二个输入范围重叠，行为未定义。
- **目标空间必须足够**：不检查目标范围大小。
- **不要用 `std::transform` 做副作用**：虽然可以，但语义上应该用 `for_each`。
- **`std::transform` 与 `std::ranges::transform` 的返回类型**：后者返回 `in_out_result`，可以链式调用。
- **`std::toupper` 的坑**：直接传 `std::toupper` 给 `transform` 可能因负值字符导致未定义行为，应包一层 lambda 并转换类型。

## 5. 相关算法

- [for_each](../NonModifying/For_each.md)：只遍历不求值
- [copy / copy_if / copy_n](./Copy.md)：复制时不做变换
- [replace / replace_if](./Replace.md)：按条件替换
- [transform_reduce](../Numeric/Transform_reduce.md)：变换后归约
