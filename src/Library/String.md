# 字符串 (string)

`std::string` 是标准库的字符串类型，封装了动态字符数组，自动管理内存。相比 C 风格的 `char*`，它无需手动分配释放，也不会因为忘记 `\0` 而出错。

C++17 引入的 `std::string_view` 则提供了**不拥有数据**的字符串视图，用于只读场景，避免不必要的拷贝。

## 1. 引入

```c++
#include <string>       // std::string
#include <string_view>  // std::string_view
```

## 2. 核心概念

### 1. 构造与赋值

```c++
std::string s1;                    // 空字符串
std::string s2 = "hello";          // 从字面量
std::string s3("hello", 3);        // "hel"（前 3 个字符）
std::string s4(5, 'x');            // "xxxxx"
std::string s5(s2);                // 拷贝构造
std::string s6(std::move(s2));     // 移动构造

s1 = "world";                      // 赋值
```

### 2. 常用操作

| 操作 | 说明 |
| :--- | :--- |
| `size()` / `length()` | 字符数（两者等价） |
| `empty()` | 是否为空 |
| `clear()` | 清空 |
| `c_str()` | 返回 C 风格字符串（保证以 `\0` 结尾） |
| `data()` | 返回底层字符数组指针 |
| `substr(pos, n)` | 取子串 |
| `find(str)` | 查找子串，返回位置或 `npos` |
| `rfind(str)` | 从后往前查找 |
| `find_first_of(chars)` | 查找任意一个字符首次出现 |
| `replace(pos, n, str)` | 替换 |
| `insert(pos, str)` | 插入 |
| `erase(pos, n)` | 删除 |
| `append(str)` / `+=` | 追加 |
| `compare(str)` | 比较，返回负数/0/正数 |
| `starts_with(str)` | 是否以某串开头（C++20） |
| `ends_with(str)` | 是否以某串结尾（C++20） |
| `contains(str)` | 是否包含子串（C++23） |

```c++
std::string s = "Hello World";

std::cout << s.size() << '\n';              // 11
std::cout << s.substr(6, 5) << '\n';        // World
std::cout << s.find("World") << '\n';       // 6

if (s.find("xyz") == std::string::npos)
    std::cout << "未找到\n";

s.replace(0, 5, "Goodbye");                 // "Goodbye World"
s += "!";                                   // "Goodbye World!"
```

### 3. 遍历与转换

```c++
std::string s = "hello";

// 遍历
for (char c : s)
    std::cout << c;

// 转数字
int n = std::stoi("42");
double d = std::stod("3.14");

// 数字转字符串
std::string a = std::to_string(42);
std::string b = std::to_string(3.14);
```

`std::stoi` 等函数在转换失败时会抛出 `std::invalid_argument` 或 `std::out_of_range`。

### 4. 小字符串优化（SSO）

大多数实现采用**小字符串优化**：短字符串（通常 15~22 字节以内）直接存储在 `string` 对象内部的缓冲区中，**不进行堆分配**。

```c++
std::string short_str = "hi";        // 无堆分配
std::string long_str(1000, 'x');     // 堆分配
```

这意味着短字符串的拷贝成本很低，但 `sizeof(std::string)` 通常比想象中大（约 32 字节）。

## 3. 用法

### string_view

`std::string_view` 是对字符序列的**非拥有视图**，只保存指针和长度：

```c++
void process(std::string_view sv)   // 不拷贝
{
    std::cout << sv.size() << '\n';
}

std::string s = "hello";
process(s);              // 从 string 构造，无拷贝
process("world");        // 从字面量构造，无拷贝
process(s.substr(0, 3)); // 注意：substr 返回临时 string，此处有拷贝
```

**优势**：

- 传参时避免拷贝，尤其适合只读的字符串参数
- 可以接受 `std::string`、字符串字面量、`char*` 等多种来源
- 取子串是 \\(O(1)\\)（只调整指针和长度）

**局限**：

- **不拥有数据**，底层字符串销毁后视图会悬垂
- **不保证以 `\0` 结尾**，不能直接传给需要 C 字符串的接口
- **没有 `c_str()`**，需要转换时用 `std::string(sv)`

```c++
// 危险：返回指向临时对象的视图
std::string_view bad()
{
    std::string local = "temporary";
    return local;   // 错误：local 已销毁
}

// 安全：返回 string
std::string good()
{
    return "safe";
}
```

### 何时用哪个

| 场景 | 推荐 |
| :--- | :--- |
| 需要拥有数据、修改内容 | `std::string` |
| 只读参数、临时查看 | `std::string_view` |
| 需要以 `\0` 结尾传给 C API | `std::string`（`c_str()`） |
| 成员变量长期持有 | `std::string`（视图易悬垂） |
| 返回字符串 | `std::string`（视图可能悬垂） |

## 4. 注意事项

- **`npos` 是 `size_t` 的最大值**。用 `find` 判断时要写 `== std::string::npos`，不要写成 `-1`。
- **`c_str()` 的指针可能失效**。任何修改字符串的操作都可能触发重新分配，使之前取得的指针悬垂。
- **`string_view` 不拥有数据**。绝不要返回指向局部变量的 `string_view`，也不要把 `string_view` 作为长期成员。
- **`string_view` 不保证 `\0` 结尾**。传给 `printf` 等 C 函数会越界。
- **`s + "a"` 与 `"a" + s` 不同**。前者合法，后者中 `"a"` 是 `const char*`，指针加法是未定义行为。应写 `std::string("a") + s`。
- **`substr` 返回新字符串**。`string_view::substr` 返回视图（\\(O(1)\\)），`string::substr` 返回新字符串（\\(O(n)\\)），注意区分。
- **字符串拼接的性能**。循环中反复 `s += x` 可能多次重新分配，预先 `reserve()` 可避免。

## 5. 相关章节

- [字符类型](../Basis/Types/Character.md)：C 风格字符串与字符类型
- [文件与流](../Library/Stream.md)：字符串流 `std::stringstream`
- [格式化输出](../Library/Format.md)：`std::format` 格式化字符串
- [正则表达式](../Library/Regex.md)：字符串的正则匹配
