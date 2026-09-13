# std::copy / std::copy_if / std::copy_n

这三个算法把元素复制到目标位置：

- `copy`：复制整个范围
- `copy_if`：只复制满足谓词的元素（C++11）
- `copy_n`：复制指定数量的元素（C++11）

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class OutputIt>
constexpr OutputIt copy(InputIt first, InputIt last, OutputIt d_first);

template<class InputIt, class OutputIt, class UnaryPred>
constexpr OutputIt copy_if(InputIt first, InputIt last, OutputIt d_first, UnaryPred pred);

template<class InputIt, class Size, class OutputIt>
constexpr OutputIt copy_n(InputIt first, Size count, OutputIt result);
```

- **迭代器要求**：输入为 `InputIterator`，输出为 `OutputIterator`。
- **复杂度**：`copy` 恰好 \\( n \\) 次赋值；`copy_if` 恰好 \\( n \\) 次谓词调用，至多 \\( n \\) 次赋值。
- **返回值**：指向目标范围中最后一个被复制元素之后的位置。

`copy` 的经典实现会对 `TriviallyCopyable` 类型且迭代器为 `contiguous_iterator` 的情况调用 `std::memmove`，因此对 `int`、`char` 等类型非常高效。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> src{1, 2, 3, 4, 5};

    // 复制到已分配空间的容器
    std::vector<int> dst(src.size());
    std::copy(src.begin(), src.end(), dst.begin());

    // 复制到空容器：用 back_inserter
    std::vector<int> dst2;
    std::copy(src.begin(), src.end(), std::back_inserter(dst2));

    // copy_if：只复制偶数
    std::vector<int> evens;
    std::copy_if(src.begin(), src.end(), std::back_inserter(evens),
                 [](int x) { return x % 2 == 0; });   // evens = {2, 4}

    // copy_n：复制前 3 个
    std::vector<int> first3(3);
    std::copy_n(src.begin(), 3, first3.begin());   // first3 = {1, 2, 3}
}
```

### (2) 谓词与投影

`copy_if` 接受一元谓词；`ranges::copy_if` 还支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 17}, {"Carol", 25}};

std::vector<Person> adults;
std::ranges::copy_if(people, std::back_inserter(adults),
                     [](int age) { return age >= 18; }, &Person::age);
// adults = {Alice, Carol}
```

### (3) 执行策略

`copy`、`copy_if`、`copy_n` 都支持 C++17 执行策略：

```c++
#include <execution>

std::copy(std::execution::par, src.begin(), src.end(), dst.begin());
```

并行版本会失去短路特性，`copy_if` 在并行下仍保持稳定（保留相对顺序）。

## 4. 注意事项

- **目标空间必须足够**：`copy` 不检查目标范围大小，写越界是未定义行为。用 `back_inserter` 可以自动扩容。
- **范围不能重叠**：如果目标范围与源范围重叠，应改用 `copy_backward`（向右复制）或 `std::copy` 配合临时缓冲。
- **`copy_if` 是稳定的**：保持被复制元素的相对顺序。
- **`copy_n` 的 `count` 非负**：`count < 0` 时行为未定义。
- **优先用 `ranges::copy`**：返回 `in_out_result`，同时给出输入和输出范围的末尾，便于链式操作。

## 5. 相关算法

- [copy_backward](./Copy_backward.md)：从后往前复制，用于重叠范围
- [move / move_backward](./Move.md)：移动而非复制
- [remove_copy](./Remove_copy.md)：复制时省略元素
- [replace_copy](./Replace_copy.md)：复制时替换元素
- [uninitialized_copy](../Memory/Uninitialized_copy.md)：复制到未初始化内存
