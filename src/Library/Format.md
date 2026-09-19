# 格式化输出 (format)

C++20 引入的格式化库，用 `std::format` 取代 `printf` 风格与流风格的输出；C++23 增加了 `std::print`。

它解决了两边的痛点：`printf` 类型不安全、不可扩展；`iostream` 冗长、状态易被污染、性能一般。`std::format` 借鉴了 Python 的 `str.format` 和 `{fmt}` 库的设计，兼顾安全性和可读性。

## 1. 引入

```c++
#include <format>   // C++20：std::format
#include <print>    // C++23：std::print
```

## 2. 核心概念

### 1. 为什么不用 printf 和 iostream

**`printf` 的问题**：

```c++
printf("%d\n", 3.14);   // 类型不匹配，未定义行为，编译器通常不报错
printf("%s\n", 42);     // 同上
```

格式串和参数类型不匹配是经典 bug，`printf` 无法在编译期检查（除非编译器开了 `-Wformat`，但只覆盖部分情况）。而且 `printf` 无法直接输出自定义类型。

**`iostream` 的问题**：

```c++
std::cout << "x = " << x << ", y = " << y << "\n";
```

冗长，格式和内容混在一起，而且 `std::hex`、`std::setprecision` 这类操纵符会**改变流的状态**，忘记恢复会影响后续输出。

**`std::format` 的写法**：

```c++
std::string s = std::format("x = {}, y = {}", x, y);
```

类型安全（编译期检查）、简洁、不改变任何全局状态。

### 2. 基本形式

```c++
std::format(格式串, 参数...)
```

格式串里用 `{}` 作为占位符，按顺序对应参数：

```c++
std::string s = std::format("{} + {} = {}", 1, 2, 3);
// "1 + 2 = 3"
```

也可以指定位置：

```c++
std::format("{1} {0}", "world", "hello");   // "hello world"
```

### 3. 格式说明符

完整语法是 `{[索引]:[填充][对齐][符号][#][0][宽度][.精度][类型]}`。

常用的部分：

| 说明符 | 含义 | 示例 |
| :--- | :--- | :--- |
| `{}` | 默认格式 | `std::format("{}", 42)` → `"42"` |
| `{:d}` | 十进制整数 | `std::format("{:d}", 42)` → `"42"` |
| `{:x}` / `{:X}` | 十六进制（小写/大写） | `std::format("{:x}", 255)` → `"ff"` |
| `{:o}` | 八进制 | `std::format("{:o}", 8)` → `"10"` |
| `{:b}` | 二进制 | `std::format("{:b}", 5)` → `"101"` |
| `{:f}` | 定点浮点 | `std::format("{:f}", 3.14)` → `"3.140000"` |
| `{:e}` | 科学计数法 | `std::format("{:e}", 314.0)` → `"3.140000e+02"` |
| `{:g}` | 自动选择 | `std::format("{:g}", 3.14)` → `"3.14"` |
| `{:c}` | 字符 | `std::format("{:c}", 65)` → `"A"` |
| `{:s}` | 字符串 | `std::format("{:s}", "hi")` → `"hi"` |
| `{:?}` | 调试格式（C++23） | 带引号的字符串 |

## 3. 用法

### 1. 宽度、对齐与填充

```c++
std::format("{:10}", "hi");      // "hi        "（左对齐，宽度 10）
std::format("{:>10}", "hi");     // "        hi"（右对齐）
std::format("{:^10}", "hi");     // "    hi    "（居中）
std::format("{:*^10}", "hi");    // "****hi****"（用 * 填充）
std::format("{:010}", 42);       // "0000000042"（补零）
```

数字默认右对齐，字符串默认左对齐。

### 2. 精度

```c++
std::format("{:.2f}", 3.14159);   // "3.14"
std::format("{:.3e}", 31415.9);   // "3.142e+04"
std::format("{:.5}", "hello world");   // "hello"（截断字符串）
```

### 3. 符号与进制前缀

```c++
std::format("{:+d}", 42);      // "+42"
std::format("{: d}", 42);      // " 42"
std::format("{:#x}", 255);     // "0xff"
std::format("{:#b}", 5);       // "0b101"
```

### 4. 输出到流或文件

`std::format` 返回字符串，可以直接输出：

```c++
std::cout << std::format("x = {}\n", x);

std::ofstream out("log.txt");
out << std::format("[{}] {}\n", level, message);
```

C++23 的 `std::print` 更直接：

```c++
std::print("x = {}\n", x);              // 输出到 stdout
std::println("x = {}", x);              // 自动加换行
std::print(stderr, "错误: {}\n", msg);  // 输出到 stderr
```

`print` 通常比 `cout` 快，因为它不需要与 C 的 `stdio` 同步。

### 5. 格式化自定义类型

需要特化 `std::formatter`：

```c++
struct Point
{
    int x, y;
};

template<>
struct std::formatter<Point>
{
    constexpr auto parse(std::format_parse_context& ctx)
    {
        return ctx.begin();   // 不支持额外格式说明符
    }

    auto format(const Point& p, std::format_context& ctx) const
    {
        return std::format_to(ctx.out(), "({}, {})", p.x, p.y);
    }
};

Point p{1, 2};
std::cout << std::format("{}", p);   // "(1, 2)"
```

如果要支持格式说明符（比如 `{:.2f}`），`parse` 需要解析它们并保存下来。

### 6. 格式化时间

`<chrono>` 类型可以直接格式化：

```c++
#include <chrono>
#include <format>

auto now = std::chrono::system_clock::now();
std::cout << std::format("{:%Y-%m-%d %H:%M:%S}", now);
```

### 7. 写入已有缓冲区

`std::format_to` 直接写入输出迭代器，避免中间字符串：

```c++
std::string buffer;
std::format_to(std::back_inserter(buffer), "x = {}", 42);

// 或者写入固定大小的缓冲区
char buf[100];
auto result = std::format_to_n(buf, sizeof(buf), "x = {}", 42);
// result.out 指向写入末尾
```

## 4. 注意事项

**编译期检查**。格式串是编译期常量时，`std::format` 会检查参数数量和类型：

```c++
std::format("{} {}", 1);        // 编译错误：参数太少
std::format("{:d}", "hello");   // 编译错误：字符串不能用 :d
```

这是相对 `printf` 的最大优势。但如果格式串是运行时字符串，检查就退化为运行时异常。

**需要编译器支持**。GCC 13+、Clang 17+、MSVC 19.29+ 才比较完整。老版本可以用 `{fmt}` 库，接口几乎一样。

**`std::format` 返回新字符串**。频繁调用会产生大量临时对象。性能敏感场景用 `std::format_to` 写入复用缓冲区。

**`std::print` 是 C++23**。只支持 C++20 时用 `std::format` 配合 `cout`。

**`{}` 与 `{{}}`**。要输出字面量花括号，需要写两个：

```c++
std::format("{{}}");   // "{}"
```

**不要混用 `printf` 和 `cout`**。如果程序里同时用了两者，输出顺序可能不符合预期（除非调用了 `std::ios::sync_with_stdio(true)`，但那样会拖慢 `cout`）。

**格式化宽字符**。`std::format` 处理 `char`，宽字符用 `std::format` 的宽字符版本（`std::wformat` 系列），或者先转成 UTF-8。

## 5. 相关章节

- [文件与流](./Stream.md)：`iostream` 与流状态
- [字符串](./String.md)：`std::string` 与 `std::string_view`
- [时间库](./Chrono.md)：格式化时间点
- [字符串流](./Stream/Sstream.md)：`sstream` 的替代方案
