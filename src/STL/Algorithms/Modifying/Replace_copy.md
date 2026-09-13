# std::replace_copy / std::replace_copy_if

这两个算法**复制**一个范围，并在复制过程中把满足条件的元素替换为另一个值，原范围不被修改。

- `replace_copy`：复制时把等于 `old_value` 的元素替换为 `new_value`
- `replace_copy_if`：复制时把使谓词返回 `true` 的元素替换为 `new_value`

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class InputIt, class OutputIt, class T>
constexpr OutputIt replace_copy(InputIt first, InputIt last, OutputIt d_first,
                                const T& old_value, const T& new_value);

template<class InputIt, class OutputIt, class UnaryPred, class T>
constexpr OutputIt replace_copy_if(InputIt first, InputIt last, OutputIt d_first,
                                   UnaryPred p, const T& new_value);
```

- **迭代器要求**：输入 `InputIterator`，输出 `OutputIterator`。
- **复杂度**：恰好 \\( n \\) 次比较或谓词调用，以及 \\( n \\) 次赋值。
- **返回值**：指向目标范围中最后一个被写入元素之后的位置。

`ranges::replace_copy` / `ranges::replace_copy_if` 支持投影。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> src{1, 2, 3, 2, 4};

    // 复制时把 2 替换为 9，原范围不变
    std::vector<int> dst(src.size());
    std::replace_copy(src.begin(), src.end(), dst.begin(), 2, 9);
    // src = {1,2,3,2,4}（不变）
    // dst = {1,9,3,9,4}

    // 复制时把偶数替换为 0
    std::vector<int> dst2;
    std::replace_copy_if(src.begin(), src.end(), std::back_inserter(dst2),
                         [](int x) { return x % 2 == 0; }, 0);
    // dst2 = {1,0,3,0,0}
}
```

### (2) 谓词与投影

`ranges::replace_copy_if` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 17}};

std::vector<Person> masked;
std::ranges::replace_copy_if(people, std::back_inserter(masked),
                             [](int age) { return age < 18; }, 0, &Person::age);
```

### (3) 执行策略

`replace_copy` 和 `replace_copy_if` 支持 C++17 执行策略：

```c++
#include <execution>

std::replace_copy(std::execution::par, src.begin(), src.end(), dst.begin(), 2, 9);
```

## 4. 注意事项

- **目标空间必须足够**：不检查目标范围大小。
- **范围不能重叠**：目标范围与源范围重叠时行为未定义。
- **与 `replace` 的选择**：需要保留原数据时用 `replace_copy`，否则用 `replace` 更省内存。
- **`replace_copy` 用 `operator==`**：自定义类型需要提供 `operator==`。

## 5. 相关算法

- [replace / replace_if](./Replace.md)：原地替换
- [copy / copy_if / copy_n](./Copy.md)：只复制不替换
- [remove_copy](./Remove_copy.md)：复制时省略元素
- [transform](./Transform.md)：按函数变换所有元素
