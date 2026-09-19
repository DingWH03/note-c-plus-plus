# Ranges

C++20 引入的 **Ranges 库**（`<ranges>`）是对算法与迭代器库的扩展，它让操作序列的代码更**可组合**、更**不易出错**。

它的核心思想是：把「对序列的变换」抽象成一个个**视图（view）**，视图之间可以用管道符 `|` 串联成流水线，且整个过程是**惰性求值**的——只有在真正遍历时才执行计算。

关于 Ranges 版本的算法（`ranges::sort` 等），见 [Algorithms](./Algorithms.md)；关于迭代器分类与概念，见 [Iterators](./Iterators.md)。

## 一、为什么需要 Ranges

传统写法需要显式传递 `begin()` / `end()`，且中间结果往往要落到临时容器：

```c++
std::vector<int> v{1, 2, 3, 4, 5, 6};

// 传统写法：取出偶数并平方
std::vector<int> temp;
std::copy_if(v.begin(), v.end(), std::back_inserter(temp),
             [](int x) { return x % 2 == 0; });

std::vector<int> result;
std::transform(temp.begin(), temp.end(), std::back_inserter(result),
               [](int x) { return x * x; });
```

Ranges 写法把两步连成一条流水线，且**不产生中间容器**：

```c++
auto result = v | std::views::filter([](int x) { return x % 2 == 0; })
                | std::views::transform([](int x) { return x * x; });

for (int x : result)
    std::cout << x << ' ';   // 4 16 36
```

## 二、视图与惰性求值

**视图（view）** 是一个轻量的、不拥有数据的范围。它只记录「如何从底层序列计算元素」，而不实际存储结果。

```c++
std::vector<int> v{1, 2, 3, 4, 5};

auto view = v | std::views::transform([](int x) {
    std::cout << "计算 " << x << '\n';
    return x * x;
});

std::cout << "视图已创建\n";
// 此时没有任何输出——变换尚未执行

for (int x : view)
    ;   // 此时才逐个计算
```

输出顺序说明了一切：

```
视图已创建
计算 1
计算 2
...
```

**惰性求值带来两个好处**：

1. **不产生中间容器**，节省内存与拷贝开销
2. **可以表示无限序列**，如 `views::iota(0)` 配合 `views::take(10)`

代价是：每次遍历都会重新计算，如果同一视图被多次遍历，计算会重复执行。

## 三、管道语法

`|` 运算符把左操作数（范围）传给右操作数（视图适配器）：

```c++
// 以下两种写法等价
auto a = v | std::views::filter(pred);
auto b = std::views::filter(v, pred);
```

管道形式更符合阅读顺序，多个适配器串联时尤其清晰：

```c++
auto result = data
    | std::views::filter([](int x) { return x > 0; })
    | std::views::transform([](int x) { return x * 2; })
    | std::views::take(5);
```

## 四、常用视图适配器

### 1. 筛选与变换

| 适配器 | 作用 | 版本 |
| :--- | :--- | :--- |
| `views::filter(pred)` | 只保留满足谓词的元素 | C++20 |
| `views::transform(f)` | 对每个元素应用函数 | C++20 |
| `views::take(n)` | 取前 \\( n \\) 个元素 | C++20 |
| `views::take_while(pred)` | 取开头满足谓词的连续元素 | C++20 |
| `views::drop(n)` | 跳过前 \\( n \\) 个元素 | C++20 |
| `views::drop_while(pred)` | 跳过开头满足谓词的连续元素 | C++20 |

```c++
std::vector<int> v{1, 2, 3, 4, 5, 6, 7, 8};

// 取大于 3 的前 3 个
auto r = v | std::views::filter([](int x) { return x > 3; })
           | std::views::take(3);   // 4 5 6
```

### 2. 结构变换

| 适配器 | 作用 | 版本 |
| :--- | :--- | :--- |
| `views::reverse` | 反向遍历 | C++20 |
| `views::join` | 展平嵌套范围 | C++20 |
| `views::split(delim)` | 按分隔符切分 | C++20 |
| `views::keys` | 取 `pair` 的 first | C++20 |
| `views::values` | 取 `pair` 的 second | C++20 |
| `views::elements<N>` | 取 tuple-like 的第 N 个元素 | C++20 |

```c++
std::map<std::string, int> m{{"a", 1}, {"b", 2}};

for (const auto& key : m | std::views::keys)
    std::cout << key << ' ';   // a b
```

### 3. 生成器

| 适配器 | 作用 | 版本 |
| :--- | :--- | :--- |
| `views::iota(start)` | 从 `start` 开始的无限递增序列 | C++20 |
| `views::iota(start, end)` | 从 `start` 到 `end` 的序列 | C++20 |
| `views::single(x)` | 只含一个元素 | C++20 |
| `views::empty<T>` | 空序列 | C++20 |
| `views::repeat(x, n)` | 重复 \\( n \\) 次 | C++23 |

```c++
// 0 到 9
for (int i : std::views::iota(0, 10))
    std::cout << i << ' ';

// 无限序列配合 take
for (int i : std::views::iota(0) | std::views::take(5))
    std::cout << i << ' ';   // 0 1 2 3 4
```

### 4. 组合与切分（C++23）

| 适配器 | 作用 |
| :--- | :--- |
| `views::zip(a, b)` | 把多个范围按位置配对 |
| `views::enumerate` | 给每个元素附上索引 |
| `views::chunk(n)` | 每 \\( n \\) 个元素分为一组 |
| `views::slide(n)` | 滑动窗口 |
| `views::stride(n)` | 每隔 \\( n \\) 个元素取一个 |
| `views::adjacent<N>` | 相邻 \\( N \\) 个元素组成 tuple |
| `views::cartesian_product` | 笛卡尔积 |

```c++
std::vector<std::string> names{"Alice", "Bob"};
std::vector<int> ages{30, 25};

for (auto [name, age] : std::views::zip(names, ages))
    std::cout << name << ": " << age << '\n';

// 带索引遍历（C++23）
for (auto [i, x] : std::views::enumerate(names))
    std::cout << i << ": " << x << '\n';
```

> [!TIP]
> **`views::enumerate` 是 C++23 最实用的适配器之一**
>
> 在此之前，带索引遍历需要手动维护计数器，或使用 `views::iota` 配合 `views::zip`：
>
> ```c++
> // C++20 的替代写法
> for (auto [i, x] : std::views::zip(std::views::iota(0), names))
>     std::cout << i << ": " << x << '\n';
> ```

## 五、转换为容器

视图本身不拥有数据，需要具体容器时用 `std::ranges::to`（C++23）：

```c++
#include <ranges>
#include <vector>

std::vector<int> v{1, 2, 3, 4, 5, 6};

// 视图转 vector
auto result = v | std::views::filter([](int x) { return x % 2 == 0; })
                | std::ranges::to<std::vector>();

// 也可以指定容器类型
auto set = v | std::views::transform([](int x) { return x % 3; })
             | std::ranges::to<std::set>();
```

C++23 之前需要手动构造：

```c++
std::vector<int> result;
auto view = v | std::views::filter([](int x) { return x % 2 == 0; });
std::ranges::copy(view, std::back_inserter(result));
```

## 六、范围概念

Ranges 用 Concepts 描述范围的能力，与迭代器概念一一对应：

| 概念 | 含义 |
| :--- | :--- |
| `ranges::range` | 有 `begin` 和 `end` |
| `ranges::input_range` | 迭代器满足 `input_iterator` |
| `ranges::forward_range` | 迭代器满足 `forward_iterator` |
| `ranges::bidirectional_range` | 迭代器满足 `bidirectional_iterator` |
| `ranges::random_access_range` | 迭代器满足 `random_access_iterator` |
| `ranges::contiguous_range` | 迭代器满足 `contiguous_iterator` |
| `ranges::sized_range` | 能在常数时间内得到大小 |
| `ranges::view` | 满足视图要求（拷贝/移动/析构为常数时间） |

此外还有几个重要概念：

- **`ranges::borrowed_range`**：从该范围取得的迭代器可以安全地脱离范围使用
- **`ranges::common_range`**：`begin()` 和 `end()` 类型相同
- **`ranges::viewable_range`**：可以安全转换为视图的范围

## 七、注意事项

- **视图不拥有数据**。如果底层容器被销毁或修改，视图会悬垂。不要返回指向局部容器的视图。
- **惰性求值会重复计算**。同一视图遍历两次，变换会执行两次。需要缓存时先转成容器。
- **注意迭代器类别**。`views::reverse` 要求双向范围，`views::filter` 的结果不是连续范围，因此不能再传给要求连续迭代器的代码。
- **`views::filter` 的结果不能直接传给 `std::sort`**。因为 `filter_view` 不提供随机访问迭代器，需要先转成容器。
- **`std::views` 是 `std::ranges::views` 的别名**。两者等价，`std::views::filter` 更常用。
- **C++23 的适配器需要编译器支持**。`views::zip`、`views::enumerate` 等在 GCC 13+、Clang 17+ 才可用。
- **`views::join` 与 `views::split` 性能有差异**。`lazy_split` 比 `split` 更省内存，但某些操作受限。

## 八、相关章节

- [Algorithms](./Algorithms.md)：Ranges 版本的算法
- [Iterators](./Iterators.md)：迭代器分类与 C++20 概念
- [Containers](./Containers.md)：容器与范围的关系
- [Lambda 表达式](../Basis/Lambda.md)：视图中大量使用 lambda
