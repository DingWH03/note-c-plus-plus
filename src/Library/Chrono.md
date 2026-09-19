# 时间库 (chrono)

C++11 引入的时间库，提供时长（duration）、时间点（time_point）与时钟（clock）三个核心概念，取代了 C 风格的 `<ctime>`。

它的设计有个重要特点：**类型安全**。秒和毫秒是不同的类型，编译器会阻止你把它们混用；时间点和时长也是不同的类型，不能相加两个时间点。这让很多时间相关的 bug 在编译期就被拦住了。

## 1. 引入

```c++
#include <chrono>

// C++20 起，字面量后缀在 std::chrono_literals 中
using namespace std::chrono_literals;
```

## 2. 核心概念

时间库由三个概念组成，它们的关系是：

```
时钟 (clock) ──产生──→ 时间点 (time_point)
                          │
                          │ 相减
                          ↓
                       时长 (duration)
```

### 1. duration：时长

`duration` 表示一段时间间隔，由**数值**和**单位**两部分组成：

```c++
template<class Rep, class Period = std::ratio<1>>
class duration;
```

- `Rep`：用什么类型存储数值（`int`、`long`、`double`）
- `Period`：单位，用 `std::ratio` 表示一个周期是多少秒

标准库预定义了一批常用单位：

| 类型 | 含义 | 与秒的比值 |
| :--- | :--- | :--- |
| `std::chrono::nanoseconds` | 纳秒 | 1/1'000'000'000 |
| `std::chrono::microseconds` | 微秒 | 1/1'000'000 |
| `std::chrono::milliseconds` | 毫秒 | 1/1000 |
| `std::chrono::seconds` | 秒 | 1 |
| `std::chrono::minutes` | 分 | 60 |
| `std::chrono::hours` | 小时 | 3600 |
| `std::chrono::days` | 天 | 86400（C++20） |
| `std::chrono::weeks` | 周 | 604800（C++20） |
| `std::chrono::months` | 月 | 约 30.44 天（C++20） |
| `std::chrono::years` | 年 | 约 365.24 天（C++20） |

```c++
std::chrono::seconds s(10);           // 10 秒
std::chrono::milliseconds ms(500);    // 500 毫秒

// C++14 起可以用字面量
auto s2 = 10s;
auto ms2 = 500ms;
auto us = 100us;
auto ns = 50ns;
```

### 2. time_point：时间点

时间点是「某个时钟的纪元加上一个时长」：

```c++
template<class Clock, class Duration = typename Clock::duration>
class time_point;
```

```c++
auto now = std::chrono::system_clock::now();   // 当前时间点
auto epoch = std::chrono::system_clock::time_point{};   // 纪元（1970-01-01）
```

两个时间点相减得到时长：

```c++
auto start = std::chrono::steady_clock::now();
doWork();
auto end = std::chrono::steady_clock::now();

auto elapsed = end - start;   // duration
```

### 3. clock：时钟

时钟提供「当前时间点」和「时长单位」：

| 时钟 | 特点 | 用途 |
| :--- | :--- | :--- |
| `system_clock` | 系统时间，**可被调整**（NTP、用户改时间） | 获取日历时间、与外部时间戳交互 |
| `steady_clock` | 单调递增，**不可调整** | **测量耗时** |
| `high_resolution_clock` | 通常是前两者的别名 | 取决于实现，不推荐直接用 |

**测耗时必须用 `steady_clock`**。`system_clock` 可能因为对时或用户操作而跳变，导致测出负数或异常大的值。

## 3. 用法

### 1. 测量耗时

这是最常见的场景：

```c++
#include <chrono>
#include <iostream>

int main()
{
    auto start = std::chrono::steady_clock::now();

    // ... 执行一些工作

    auto end = std::chrono::steady_clock::now();
    auto elapsed = std::chrono::duration_cast<std::chrono::milliseconds>(end - start);

    std::cout << "耗时 " << elapsed.count() << " ms\n";
}
```

`duration_cast` 用于在不同精度之间转换。**注意它会截断**，不是四舍五入：

```c++
std::chrono::milliseconds ms(1500);
auto s = std::chrono::duration_cast<std::chrono::seconds>(ms);
std::cout << s.count();   // 1，不是 2
```

C++17 起可以用 `std::chrono::round`、`floor`、`ceil` 精确控制：

```c++
auto s = std::chrono::round<std::chrono::seconds>(ms);   // 2
```

### 2. 隐式转换

低精度到高精度可以隐式转换，反之必须显式：

```c++
std::chrono::seconds s(1);
std::chrono::milliseconds ms = s;              // OK：隐式，1 秒 = 1000 毫秒

// std::chrono::seconds s2 = ms;               // 错误：可能丢失精度
auto s2 = std::chrono::duration_cast<std::chrono::seconds>(ms);   // 显式
```

这个规则是刻意的——它让「精度损失」这件事在代码里显式可见。

### 3. 时间运算

时长之间可以自由加减：

```c++
auto total = 1s + 500ms;      // 结果是 milliseconds(1500)
auto diff = 2s - 500ms;       // milliseconds(1500)

auto doubled = 2 * 100ms;     // milliseconds(200)
auto halved = 100ms / 2;      // milliseconds(50)

// 求余
auto rem = 1500ms % 1s;       // milliseconds(500)
```

时间点可以加减时长：

```c++
auto now = std::chrono::system_clock::now();
auto later = now + 1h;
auto earlier = now - 30min;
```

但**两个时间点不能相加**——这在语义上没有意义，编译器会阻止。

### 4. 与 C 风格时间互转

`system_clock` 可以与 `time_t` 互转：

```c++
// time_point → time_t
auto now = std::chrono::system_clock::now();
std::time_t t = std::chrono::system_clock::to_time_t(now);

// time_t → time_point
auto tp = std::chrono::system_clock::from_time_t(t);
```

C++20 起还可以直接与 `std::tm` 互转：

```c++
std::tm tm = std::chrono::system_clock::to_time_t(now);   // 旧写法需要 localtime
```

### 5. C++20 的日历功能

C++20 大幅扩展了时间库，加入了日历类型：

```c++
using namespace std::chrono;

// 年月日
year_month_day ymd{2026y, January/15d};
std::cout << int(ymd.year()) << "-" << unsigned(ymd.month()) << "-" << unsigned(ymd.day());

// 星期
weekday wd = 2026y/January/15d;
if (wd == Thursday) { /* ... */ }

// 时区
auto local = zoned_time{current_zone(), system_clock::now()};
std::cout << local;
```

还支持格式化输出：

```c++
std::cout << std::format("{:%Y-%m-%d %H:%M:%S}", std::chrono::system_clock::now());
```

## 4. 注意事项

**测耗时用 `steady_clock`，不要用 `system_clock`**。后者可能跳变，测出的结果不可信。

**`duration_cast` 会截断**。1500 毫秒转成秒是 1 而不是 2。需要四舍五入时用 `round`。

**`.count()` 返回的是裸数值**。它丢掉了单位信息，不要把它存成 `int` 变量再传出去。

```c++
// 不好：单位信息丢失
int ms = elapsed.count();

// 好：保留类型
auto ms = std::chrono::duration_cast<std::chrono::milliseconds>(elapsed);
```

**`high_resolution_clock` 不保证是 steady 的**。标准允许它别名到 `system_clock`，因此不适合测耗时。

**字面量后缀需要 `using namespace std::chrono_literals`**。否则 `1s`、`100ms` 这些无法使用。

**`ratio` 的精度**。`std::chrono::months` 和 `years` 是平均值（月约 30.44 天），不是精确的日历单位。做日期计算时要注意这一点。

**C++20 的日历功能需要编译器支持**。GCC 11+、Clang 17+ 才比较完整。

## 5. 相关章节

- [文件与流](./Stream.md)：流式输入输出
- [格式化输出](./Format.md)：`std::format` 格式化时间
- [随机数](./Random.md)：用时间做随机种子
- [多线程与并发](../Advance/Concurrency.md)：`sleep_for` 与 `sleep_until`
