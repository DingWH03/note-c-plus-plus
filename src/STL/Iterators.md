# Iterators

**迭代器（Iterator）** 是 STL 的核心组件之一。它是对指针的泛化，提供了一套统一的接口来遍历不同的数据结构——无论底层是连续数组、链表还是红黑树，算法都通过迭代器访问元素，无需关心容器的内部实现。

正因如此，STL 的算法才能独立于容器存在：`std::sort` 不关心你传的是 `vector` 还是 `deque`，它只要求迭代器满足随机访问的要求。

## 一、迭代器与指针的关系

迭代器的语义是**指针语义的推广**。指针本身就是一个合法的迭代器：

```c++
int arr[] = {1, 2, 3, 4, 5};

// 用指针作为迭代器
std::sort(arr, arr + 5);
auto it = std::find(arr, arr + 5, 3);
```

标准库保证：任何接受迭代器的函数模板，也能接受普通指针。

迭代器的基本操作包括：

| 操作 | 含义 |
| :--- | :--- |
| `*it` | 解引用，访问所指元素 |
| `++it` / `it++` | 移动到下一个元素 |
| `--it` / `it--` | 移动到上一个元素（部分迭代器支持） |
| `it1 == it2` | 判断是否指向同一位置 |
| `it + n` / `it - n` | 移动 n 个位置（部分迭代器支持） |
| `it1 - it2` | 计算距离（部分迭代器支持） |

## 二、迭代器分类

C++17 起共有**六类**迭代器，按能力从弱到强排列。每一类都包含前一类的能力：

| 分类 | 读写能力 | 可递增 | 可递减 | 随机访问 | 连续存储 | 典型例子 |
| :--- | :--- | :---: | :---: | :---: | :---: | :--- |
| **输入迭代器** (Input) | 只读，单遍 | ✓ | | | | `istream_iterator` |
| **输出迭代器** (Output) | 只写，单遍 | ✓ | | | | `ostream_iterator` |
| **前向迭代器** (Forward) | 读写，多遍 | ✓ | | | | `forward_list` |
| **双向迭代器** (Bidirectional) | 读写 | ✓ | ✓ | | | `list`、`set`、`map` |
| **随机访问迭代器** (Random Access) | 读写 | ✓ | ✓ | ✓ | | `deque` |
| **连续迭代器** (Contiguous) | 读写 | ✓ | ✓ | ✓ | ✓ | `vector`、`array`、`string` |

几点说明：

- **输入与输出迭代器**是单遍的（single-pass）：只能遍历一次，`it1 == it2` 不保证有意义。它们主要用于流操作。
- **前向迭代器**支持多遍遍历，可以保存副本后分别遍历。
- **连续迭代器**是 C++17 正式加入的分类（此前 `vector`、`string`、`array` 的迭代器在实际使用中被当作独立类别）。它保证元素在内存中连续，因此 `&*it` 得到的地址可以像数组指针一样使用。

> [!TIP]
> **为什么分类重要？**
>
> 每个算法都要求特定类别的迭代器。例如 `std::sort` 要求随机访问迭代器，因此**不能用于 `std::list`**（`list` 只提供双向迭代器）。这也是 `std::list` 自带成员函数 `sort` 的原因。
>
> 选错容器时，编译器报错通常就是「迭代器类别不满足要求」。

## 三、迭代器与容器对照

| 容器 | 迭代器类别 | 是否支持 `--it` | 是否支持 `it + n` |
| :--- | :--- | :---: | :---: |
| `vector` | 连续 | ✓ | ✓ |
| `array` | 连续 | ✓ | ✓ |
| `string` | 连续 | ✓ | ✓ |
| `deque` | 随机访问 | ✓ | ✓ |
| `list` | 双向 | ✓ | |
| `forward_list` | 前向 | | |
| `set` / `map` | 双向 | ✓ | |
| `unordered_set` / `unordered_map` | 前向 | | |

## 四、迭代器适配器

适配器在已有迭代器基础上改变行为，定义在 `<iterator>` 中。

### 1. 插入迭代器

插入迭代器把「赋值」变成「插入」，常用于向空容器写入：

| 适配器 | 创建函数 | 插入位置 | 适用容器 |
| :--- | :--- | :--- | :--- |
| `back_insert_iterator` | `back_inserter(c)` | 尾部 | `vector`、`deque`、`list`、`string` |
| `front_insert_iterator` | `front_inserter(c)` | 头部 | `deque`、`list`、`forward_list` |
| `insert_iterator` | `inserter(c, it)` | 指定位置之前 | 所有容器 |

```c++
std::vector<int> src{1, 2, 3};
std::vector<int> dst;

// 不用预先分配空间
std::copy(src.begin(), src.end(), std::back_inserter(dst));
// dst = {1,2,3}

std::list<int> lst;
std::copy(src.begin(), src.end(), std::front_inserter(lst));
// lst = {3,2,1}（每次插入头部，顺序反转）

std::vector<int> v{1, 5};
std::copy(src.begin(), src.end(), std::inserter(v, v.begin() + 1));
// v = {1,1,2,3,5}
```

### 2. 反向迭代器

`reverse_iterator` 把递增变成递减，用于从后往前遍历：

```c++
std::vector<int> v{1, 2, 3, 4, 5};

for (auto it = v.rbegin(); it != v.rend(); ++it)
    std::cout << *it << ' ';   // 输出: 5 4 3 2 1
```

`rbegin()` 对应最后一个元素，`rend()` 对应第一个元素之前的位置。注意 `*rbegin()` 是最后一个元素，而解引用 `rend()` 是未定义行为。

### 3. 移动迭代器

`move_iterator` 解引用时返回右值引用，使算法执行移动而非拷贝：

```c++
std::vector<std::string> src{"hello", "world"};
std::vector<std::string> dst;

std::copy(std::make_move_iterator(src.begin()),
          std::make_move_iterator(src.end()),
          std::back_inserter(dst));
// dst 获得元素，src 中的字符串被移空
```

### 4. 流迭代器

流迭代器把输入输出流接入算法：

```c++
#include <iterator>

// 从 cin 读取整数，直到输入失败
std::vector<int> v;
std::copy(std::istream_iterator<int>(std::cin),
          std::istream_iterator<int>(),
          std::back_inserter(v));

// 输出到 cout，用空格分隔
std::copy(v.begin(), v.end(),
          std::ostream_iterator<int>(std::cout, " "));
```

## 五、迭代器操作函数

`<iterator>` 提供了几个通用的迭代器操作，它们会根据迭代器类别自动选择高效实现：

| 函数 | 作用 | 复杂度 |
| :--- | :--- | :--- |
| `std::advance(it, n)` | 将 `it` 前进 `n` 步（`n` 可为负） | 随机访问为 \\(O(1)\\)，否则 \\(O(n)\\) |
| `std::next(it, n)` | 返回前进 `n` 步后的迭代器（默认 `n=1`） | 同上 |
| `std::prev(it, n)` | 返回后退 `n` 步后的迭代器（默认 `n=1`） | 双向以上 |
| `std::distance(first, last)` | 返回两个迭代器之间的距离 | 随机访问为 \\(O(1)\\)，否则 \\(O(n)\\) |

```c++
std::list<int> lst{1, 2, 3, 4, 5};

auto it = lst.begin();
std::advance(it, 3);                    // it 指向 4
auto it2 = std::next(lst.begin(), 2);   // it2 指向 3
auto n = std::distance(lst.begin(), it);  // 3
```

> [!TIP]
> **优先用 `std::next` 而非 `it + n`**
>
> `it + n` 只对随机访问迭代器有效，而 `std::next` 对所有前向以上迭代器都可用。在模板代码中应使用 `std::next`。

## 六、迭代器失效

迭代器失效是指容器修改后，原有的迭代器不再指向有效元素。这是使用 STL 时最容易出错的地方，规则因容器而异：

| 容器 | 插入 | 删除 |
| :--- | :--- | :--- |
| `vector` | 扩容时**全部失效**；否则插入点之后的失效 | 删除点及之后全部失效 |
| `deque` | 头尾插入仅迭代器失效；中间插入全部失效 | 头尾删除仅被删元素失效；中间删除全部失效 |
| `list` / `forward_list` | **不失效** | 仅被删元素失效 |
| `set` / `map` | **不失效** | 仅被删元素失效 |
| `unordered_*` | rehash 时全部失效 | 仅被删元素失效 |

详细规则见各容器章节，例如 [Vector · 迭代器失效](./Containers/Vector.md)。

## 七、C++20 的迭代器概念

C++20 用 Concepts 重新定义了迭代器体系，与 C++17 的「具名要求」相比，约束更精确、报错更清晰：

| C++17 分类 | C++20 概念 |
| :--- | :--- |
| `InputIterator` | `std::input_iterator` |
| `OutputIterator` | `std::output_iterator` |
| `ForwardIterator` | `std::forward_iterator` |
| `BidirectionalIterator` | `std::bidirectional_iterator` |
| `RandomAccessIterator` | `std::random_access_iterator` |
| `ContiguousIterator` | `std::contiguous_iterator` |

此外还有更基础的概念：`indirectly_readable`、`indirectly_writable`、`weakly_incrementable`、`sentinel_for`、`sized_sentinel_for` 等。

C++20 还引入了**哨兵（sentinel）**的概念：范围的起点和终点可以是**不同类型**，只要它们可以比较。这使 `[begin, sentinel)` 形式的范围更灵活。

关联类型也改用了新的别名模板：

```c++
std::iter_value_t<It>          // 迭代器的值类型
std::iter_reference_t<It>      // 解引用返回的引用类型
std::iter_difference_t<It>     // 距离类型
```

## 八、注意事项

- **不要把 `end()` 缓存起来跨修改使用**。容器修改后 `end()` 可能失效，应在循环条件中实时获取。
- **删除元素时用 `erase` 的返回值**。`erase` 返回下一个有效迭代器，直接 `++it` 是未定义行为。
- **反向迭代器的 `base()` 有偏移**。`rit.base()` 指向的是 `rit` 所指元素的**下一个**位置。
- **`std::distance` 对非随机访问迭代器是 \\(O(n)\\)**。在大链表上频繁调用会成为性能瓶颈。
- **`auto&&` 与代理迭代器**。`vector<bool>` 的迭代器返回代理对象而非 `bool&`，`for (auto& x : vec)` 无法编译，详见 [Vector](./Containers/Vector.md)。
- **警惕悬垂迭代器**。容器销毁或元素被删除后，指向它的迭代器立即失效。

## 九、相关章节

- [Containers](./Containers.md)：各容器的迭代器类别与失效规则
- [Algorithms](./Algorithms.md)：算法对迭代器的要求
- [Vector](./Containers/Vector.md)：迭代器失效的典型案例
- [List](./Containers/List.md)：双向迭代器与成员函数 `sort`
