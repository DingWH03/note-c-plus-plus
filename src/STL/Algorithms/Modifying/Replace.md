# std::replace / std::replace_if

这两个算法把范围内满足条件的元素替换为另一个值：

- `replace`：把所有等于 `old_value` 的元素替换为 `new_value`
- `replace_if`：把所有使谓词返回 `true` 的元素替换为 `new_value`

## 1. 引入

```c++
#include <algorithm>
```

## 2. 原理

```c++
template<class ForwardIt, class T>
constexpr void replace(ForwardIt first, ForwardIt last,
                       const T& old_value, const T& new_value);

template<class ForwardIt, class UnaryPred, class T>
constexpr void replace_if(ForwardIt first, ForwardIt last, UnaryPred p, const T& new_value);
```

- **迭代器要求**：`ForwardIterator`。
- **复杂度**：恰好 \\( n \\) 次比较或谓词调用。
- **返回值**：`void`。

`ranges::replace` / `ranges::replace_if` 支持投影。

## 3. 用法

### (1) 基本用法

```c++
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> v{1, 2, 3, 2, 4};

    // 把所有 2 替换为 9
    std::replace(v.begin(), v.end(), 2, 9);
    // v = {1, 9, 3, 9, 4}

    // 把所有偶数替换为 0
    std::replace_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; }, 0);
    // v = {1, 0, 3, 0, 0}
}
```

### (2) 谓词与投影

`ranges::replace_if` 支持投影：

```c++
struct Person { std::string name; int age; };
std::vector<Person> people{{"Alice", 30}, {"Bob", 17}};

// 把未成年的年龄改为 0
std::ranges::replace_if(people, [](int age) { return age < 18; }, 0, &Person::age);
```

### (3) 执行策略

`replace` 和 `replace_if` 支持 C++17 执行策略：

```c++
#include <execution>

std::replace(std::execution::par, v.begin(), v.end(), 2, 9);
```

## 4. 注意事项

- **原地修改**：`replace` 直接修改原范围，不创建副本。需要保留原数据时用 `replace_copy`。
- **返回 `void`**：无法链式调用，但 `ranges::replace` 同样返回 `void`（返回的是迭代器）。
- **`replace` 用 `operator==`**：自定义类型需要提供 `operator==`。
- **谓词不应有副作用**：标准不保证谓词调用次数。

## 5. 相关算法

- [replace_copy / replace_copy_if](./Replace_copy.md)：复制时替换
- [transform](./Transform.md)：按函数变换所有元素
- [remove / remove_if](./Remove.md)：移除而非替换
- [fill / fill_n](./Fill.md)：无条件赋值
