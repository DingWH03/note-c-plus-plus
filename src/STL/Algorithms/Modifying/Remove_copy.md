# std::remove_copy / std::remove_copy_if

这两个算法**复制**一个范围，并在复制过程中**省略**满足条件的元素，原范围不被修改。

- `remove_copy`：省略所有等于给定值的元素
- `remove_copy_if`：省略所有使谓词返回 `true` 的元素

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class OutputIt, class T>
constexpr OutputIt remove_copy(InputIt first, InputIt last, OutputIt d_first, const T& value);

template<class InputIt, class OutputIt, class UnaryPred>
constexpr OutputIt remove_copy_if(InputIt first, InputIt last, OutputIt d_first, UnaryPred p);
```

- **迭代器要求**：输入 `InputIterator`，输出 `OutputIterator`。
- **复杂度**：恰好 \\( n \\) 次比较或谓词调用，至多 \\( n \\) 次赋值。
- **返回值**：指向目标范围中最后一个被写入元素之后的位置。

`ranges::remove_copy` / `ranges::remove_copy_if` 支持投影。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> src{1, 2, 3, 2, 4, 2, 5};

    // 复制时省略所有 2
    std::vector<int> dst;
    std::remove_copy(src.begin(), src.end(), std::back_inserter(dst), 2);
    // src 不变，dst = {1,3,4,5}

    // 复制时省略偶数
    std::vector<int> odds;
    std::remove_copy_if(src.begin(), src.end(), std::back_inserter(odds),
                        [](int x) { return x % 2 == 0; });
    // odds = {1,3,5}
}
```

### (2) 谓词与投影

`ranges::remove_copy_if` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 17}};

std::vector<Person> adults;
std::ranges::remove_copy_if(people, std::back_inserter(adults),
                            [](int age) { return age < 18; }, &Person::age);
// adults = {Alice}
```

### (3) 执行策略

`remove_copy` 和 `remove_copy_if` 支持 C++17 执行策略：

```c++
#include <execution>

std::remove_copy(std::execution::par, src.begin(), src.end(), dst.begin(), 2);
```

## 4. 注意事项

- **目标空间必须足够**：不检查目标范围大小，用 `back_inserter` 最安全。
- **范围不能重叠**。
- **与 `remove` 的区别**：`remove` 原地逻辑移除，需要配合 `erase`；`remove_copy` 生成新序列，原范围不动。
- **`remove_copy` 用 `operator==`**。

## 5. 相关算法

- [remove / remove_if](./Remove.md)：原地逻辑移除
- [copy / copy_if / copy_n](./Copy.md)：复制时不省略
- [replace_copy](./Replace_copy.md)：复制时替换
- [unique_copy](./Unique.md)：复制时去除连续重复
