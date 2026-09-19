# 正则表达式 (regex)

C++11 引入的正则表达式库，支持匹配、搜索、替换和迭代遍历。

用之前先有个心理准备：**`<regex>` 的性能和编译期开销都不太理想**。它的接口设计依赖模板实例化，编译慢；运行时也不如专门的库快。如果正则只是简单模式（比如找子串、判断前缀），用 `std::string` 的成员函数更快也更清晰。

## 1. 引入

```c++
#include <regex>
```

## 2. 核心概念

正则库由三个部分组成：

| 组件 | 作用 |
| :--- | :--- |
| `std::regex` | 编译后的正则表达式对象 |
| `std::match_results`（`smatch`/`cmatch`） | 保存匹配结果 |
| 算法函数 | `regex_match`、`regex_search`、`regex_replace` |

### 三个匹配函数

| 函数 | 语义 | 典型用途 |
| :--- | :--- | :--- |
| `regex_match` | **整个字符串**必须完全匹配 | 校验格式（邮箱、日期） |
| `regex_search` | 字符串中**任意位置**匹配即可 | 查找、提取 |
| `regex_replace` | 替换所有匹配 | 文本处理 |

这个区别很关键。`regex_match` 要求从头到尾完全吻合，`regex_search` 只要找到一处就行：

```c++
std::regex re(R"(\d+)");
std::string s = "abc123def";

std::regex_match(s, re);    // false：整个串不全是数字
std::regex_search(s, re);   // true：中间有数字
```

### 字符类型

| 类型 | 字符类型 | 字符串类型 |
| :--- | :--- | :--- |
| `std::regex` | `char` | `std::string` |
| `std::wregex` | `wchar_t` | `std::wstring` |

对应的匹配结果类型：

| 类型 | 对应 |
| :--- | :--- |
| `std::smatch` | `std::string` 的匹配结果 |
| `std::cmatch` | `const char*` 的匹配结果 |
| `std::wsmatch` | `std::wstring` 的匹配结果 |

## 3. 用法

### 1. 校验格式

```c++
#include <iostream>
#include <regex>
#include <string>

bool isValidDate(const std::string& s)
{
    // 匹配 YYYY-MM-DD
    static const std::regex re(R"(\d{4}-\d{2}-\d{2})");
    return std::regex_match(s, re);
}

isValidDate("2026-09-19");   // true
isValidDate("2026-9-19");    // false：月份必须是两位
isValidDate("x2026-09-19");  // false：regex_match 要求完全匹配
```

用原始字符串 `R"(...)"` 可以避免反斜杠转义，正则里写 `\d` 而不用写 `\\d`。

### 2. 提取内容

用捕获组 `(...)` 提取子串：

```c++
std::string log = "2026-09-19 14:30:25 [INFO] 启动完成";
std::regex re(R"((\d{4})-(\d{2})-(\d{2}) (\d{2}):(\d{2}):(\d{2}))");

std::smatch m;
if (std::regex_search(log, m, re))
{
    std::cout << "完整匹配: " << m[0] << '\n';   // 2026-09-19 14:30:25
    std::cout << "年: " << m[1] << '\n';         // 2026
    std::cout << "月: " << m[2] << '\n';         // 09
    std::cout << "日: " << m[3] << '\n';         // 19
    std::cout << "时: " << m[4] << '\n';         // 14
}
```

`m[0]` 是完整匹配，`m[1]` 起是各个捕获组。`m.size()` 返回组数加一。

### 3. 遍历所有匹配

用 `sregex_iterator` 遍历：

```c++
std::string text = "apple 10, banana 20, cherry 30";
std::regex re(R"((\w+)\s+(\d+))");

for (auto it = std::sregex_iterator(text.begin(), text.end(), re);
     it != std::sregex_iterator(); ++it)
{
    const std::smatch& m = *it;
    std::cout << m[1] << " = " << m[2] << '\n';
}
// 输出：
// apple = 10
// banana = 20
// cherry = 30
```

### 4. 替换

```c++
std::string text = "2026-09-19";

// 把 YYYY-MM-DD 换成 DD/MM/YYYY
std::regex re(R"((\d{4})-(\d{2})-(\d{2}))");
std::string result = std::regex_replace(text, re, "$3/$2/$1");
// 结果：19/09/2026
```

替换字符串里用 `$1`、`$2` 引用捕获组，`$&` 表示整个匹配。

### 5. 正则语法速查

| 语法 | 含义 |
| :--- | :--- |
| `.` | 任意字符（默认不含换行） |
| `\d` / `\w` / `\s` | 数字 / 单词字符 / 空白 |
| `\D` / `\W` / `\S` | 上述的补集 |
| `[abc]` | 字符集合 |
| `[^abc]` | 排除集合 |
| `[a-z]` | 范围 |
| `*` / `+` / `?` | 0 次或多次 / 1 次或多次 / 0 或 1 次 |
| `{n}` / `{n,m}` | 恰好 n 次 / n 到 m 次 |
| `^` / `$` | 行首 / 行尾 |
| `\b` | 单词边界 |
| `(...)` | 捕获组 |
| `(?:...)` | 非捕获组 |
| `\|` | 或 |
| `\1` | 反向引用第 1 组 |

### 6. 语法选项

```c++
// 忽略大小写
std::regex re("hello", std::regex::icase);

// 多行模式：^ 和 $ 匹配每行
std::regex re2("^abc", std::regex::multiline);

// 扩展语法（默认）
std::regex re3("a+", std::regex::ECMAScript);
```

可用的语法标志：`ECMAScript`（默认）、`basic`、`extended`、`awk`、`grep`、`egrep`。

## 4. 注意事项

**`regex_match` 和 `regex_search` 语义不同**。前者要求完全匹配，后者只要部分匹配。校验格式用前者，提取内容用后者。

**正则对象应该复用**。构造 `std::regex` 需要编译正则表达式，开销很大：

```c++
// 错误：每次调用都重新编译
bool check(const std::string& s) {
    std::regex re(R"(\d+)");
    return std::regex_match(s, re);
}

// 正确：静态复用
bool check(const std::string& s) {
    static const std::regex re(R"(\d+)");
    return std::regex_match(s, re);
}
```

**`std::regex` 的性能一般**。它在很多实现里是基于回溯的，遇到复杂模式或恶意输入可能指数级退化（ReDoS）。性能敏感场景考虑 RE2、PCRE2 等库。

**编译期开销大**。`<regex>` 会引入大量模板实例化，一个简单的 `#include <regex>` 可能让编译时间增加不少。如果只是简单匹配，用字符串函数更划算。

**`std::regex` 不是线程安全的**（构造和修改时）。但构造完成后多个线程可以并发使用同一个 `const` 对象。

**注意 `\d` 在原始字符串中的写法**。用 `R"(\d+)"` 而不是 `"\\d+"`，前者更易读。

**异常处理**。正则语法错误会在构造时抛 `std::regex_error`：

```c++
try {
    std::regex re("([");   // 括号不匹配
} catch (const std::regex_error& e) {
    std::cerr << "正则错误: " << e.what() << '\n';
}
```

**简单场景用字符串函数**。判断前缀用 `starts_with`，查找子串用 `find`，这些都比正则快得多也清晰得多。

## 5. 相关章节

- [字符串](./String.md)：`std::string` 的查找与替换
- [格式化输出](./Format.md)：`std::format` 格式化
- [文件与流](./Stream.md)：从文件读取待处理文本
