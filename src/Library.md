# 标准库

C++ 标准库（Standard Library）是随语言一起发布的庞大组件集合，涵盖字符串、容器、算法、时间、随机数、文件系统、并发等各个方面。

STL（Standard Template Library）最初是一个独立的库，后来被纳入标准库。今天所说的 STL 通常指其中的**容器、迭代器、算法、函数对象**这几部分，而标准库的范围要更广。

| 组件 | 头文件 | 说明 |
| :--- | :--- | :--- |
| **STL** | 多个 | 容器、迭代器、算法、函数对象，见 [STL](./STL.md) |
| **字符串** | `<string>`、`<string_view>` | `std::string` 与字符串视图 |
| **输入输出** | `<iostream>`、`<fstream>`、`<sstream>` | 控制台、文件与内存字符串流 |
| **格式化** | `<format>`、`<print>` | C++20/23 的格式化输出 |
| **时间** | `<chrono>` | 时间点、时长与时钟 |
| **随机数** | `<random>` | 随机数引擎与分布 |
| **正则表达式** | `<regex>` | 正则匹配与替换 |
| **文件系统** | `<filesystem>` | 路径操作与目录遍历 |
| **数值** | `<cmath>`、`<complex>`、`<numbers>` | 数学函数、复数与数学常量 |
| **位操作** | `<bitset>`、`<bit>` | 位集合与位运算 |
| **并发** | `<thread>` 等 | 见 [多线程与并发](./Advance/Concurrency.md) |

## 本章内容

| 章节 | 说明 |
| :--- | :--- |
| [文件与流](./Library/Stream.md) | 流模型与类层次结构总览 |
| [标准输入输出流 (iostream)](./Library/Stream/Iostream.md) | `cin` / `cout` / `cerr` / `clog` 与格式化控制 |
| [文件流 (fstream)](./Library/Stream/Fstream.md) | 文件读写、打开模式、二进制与随机访问 |
| [字符串流 (sstream)](./Library/Stream/Sstream.md) | 内存字符串的解析与拼接 |
| [字符串 (string)](./Library/String.md) | `std::string` 接口、SSO 与 `std::string_view` |
| [时间库 (chrono)](./Library/Chrono.md) | 时长、时间点、时钟与 C++20 日历 |
| [随机数 (random)](./Library/Random.md) | 随机数引擎、分布与正确用法 |
| [正则表达式 (regex)](./Library/Regex.md) | 正则匹配、捕获组、遍历与替换 |
| [文件系统 (filesystem)](./Library/Filesystem.md) | 路径操作、目录遍历与文件属性 |
| [格式化输出 (format)](./Library/Format.md) | `std::format` 与 `std::print` |
