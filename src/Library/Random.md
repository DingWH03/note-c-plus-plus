# 随机数 (random)

C++11 引入的随机数库，把随机数生成拆分为**引擎**（engine）和**分布**（distribution）两部分，质量远高于 C 的 `rand()`。

这个拆分是有意设计的：引擎负责产生均匀分布的原始随机位，分布负责把这些位映射成你想要的分布形态。同一个引擎可以配不同的分布，同一个分布也可以配不同的引擎。

## 1. 引入

```c++
#include <random>
```

## 2. 核心概念

### 1. 为什么不用 rand()

`rand()` 有几个致命问题：

- **质量差**：很多实现的低位随机性很弱，`rand() % 2` 可能交替出现
- **范围固定**：通常是 `[0, RAND_MAX]`，`RAND_MAX` 可能只有 32767
- **取模有偏**：`rand() % 100` 在 `RAND_MAX` 不是 100 的倍数时，前面的数出现概率更高
- **不可控**：全局状态，多线程下不安全，也无法复现

`<random>` 解决了所有这些问题。

### 2. 引擎

引擎是**均匀随机位生成器**（Uniform Random Bit Generator, URBG），负责产生原始随机数。

| 引擎 | 说明 |
| :--- | :--- |
| `std::mt19937` | **推荐默认选择**，32 位梅森旋转算法 |
| `std::mt19937_64` | 64 位版本 |
| `std::minstd_rand` | 线性同余，快但周期短 |
| `std::ranlux24` / `ranlux48` | 质量更高，但更慢 |
| `std::knuth_b` | Knuth 的减法算法 |
| `std::random_device` | 系统提供的真随机源（可能较慢） |

```c++
std::mt19937 gen(42);        // 用固定种子，结果可复现
std::mt19937 gen2(std::random_device{}());   // 用真随机种子
```

**梅森旋转**（Mersenne Twister）是默认推荐——周期长达 \\( 2^{19937}-1 \\)，质量好，速度也不慢。

### 3. 分布

分布把引擎产生的均匀随机数转换成目标分布。

**均匀分布**：

| 分布 | 说明 |
| :--- | :--- |
| `std::uniform_int_distribution<T>` | 整数均匀分布，闭区间 `[a, b]` |
| `std::uniform_real_distribution<T>` | 浮点均匀分布，半开区间 `[a, b)` |

**其它分布**：

| 分布 | 说明 | 典型用途 |
| :--- | :--- | :--- |
| `std::normal_distribution` | 正态分布 | 模拟自然现象、噪声 |
| `std::bernoulli_distribution` | 伯努利分布（真假） | 概率事件 |
| `std::binomial_distribution` | 二项分布 | 成功次数 |
| `std::poisson_distribution` | 泊松分布 | 单位时间事件数 |
| `std::exponential_distribution` | 指数分布 | 事件间隔 |
| `std::discrete_distribution` | 离散加权分布 | 按权重选择 |

## 3. 用法

### 1. 基本用法

```c++
#include <iostream>
#include <random>

int main()
{
    // 1. 创建引擎并播种
    std::random_device rd;
    std::mt19937 gen(rd());

    // 2. 创建分布
    std::uniform_int_distribution<int> dist(1, 6);   // 骰子

    // 3. 生成随机数
    for (int i = 0; i < 10; ++i)
        std::cout << dist(gen) << ' ';
}
```

注意调用形式是 `dist(gen)`——把引擎传给分布，而不是调用引擎的方法。

### 2. 各种分布

```c++
std::mt19937 gen(std::random_device{}());

// 整数 [1, 100]
std::uniform_int_distribution<int> dice(1, 100);
int n = dice(gen);

// 浮点 [0.0, 1.0)
std::uniform_real_distribution<double> real(0.0, 1.0);
double d = real(gen);

// 正态分布，均值 0，标准差 1
std::normal_distribution<double> normal(0.0, 1.0);
double gauss = normal(gen);

// 伯努利分布，70% 概率为 true
std::bernoulli_distribution coin(0.7);
bool b = coin(gen);

// 加权选择：30%、50%、20%
std::discrete_distribution<int> weighted({30, 50, 20});
int choice = weighted(gen);   // 返回 0、1 或 2
```

### 3. 随机打乱容器

```c++
#include <algorithm>
#include <vector>

std::vector<int> v{1, 2, 3, 4, 5};
std::mt19937 gen(std::random_device{}());
std::shuffle(v.begin(), v.end(), gen);
```

### 4. 可复现的随机

固定种子可以让每次运行得到相同序列，这对测试和调试很重要：

```c++
std::mt19937 gen(12345);   // 固定种子
// 每次运行都产生相同的序列
```

反过来，需要不可预测时用 `random_device`：

```c++
std::random_device rd;
std::mt19937 gen(rd());
```

> `std::random_device` 在某些平台（如老版本 MinGW）上实现为确定性伪随机。如果需要真随机，要检查 `rd.entropy()` 是否非零。

## 4. 注意事项

**不要用 `%` 取模**。这会引入偏差：

```c++
// 错误：有偏
int n = gen() % 100;

// 正确：均匀
std::uniform_int_distribution<int> dist(0, 99);
int n = dist(gen);
```

**不要每次生成都新建引擎**。构造引擎有开销（梅森旋转要初始化 624 个字）：

```c++
// 错误：每次循环都构造
for (int i = 0; i < 1000; ++i) {
    std::mt19937 gen(std::random_device{}());
    int n = dist(gen);
}

// 正确：构造一次
std::mt19937 gen(std::random_device{}());
for (int i = 0; i < 1000; ++i)
    int n = dist(gen);
```

**`uniform_real_distribution` 是半开区间**。`[a, b)` 不包含 `b`，这与 `uniform_int_distribution` 的闭区间不同。

**引擎和分布都不是线程安全的**。多线程下每个线程应该有自己的引擎：

```c++
void worker()
{
    thread_local std::mt19937 gen(std::random_device{}());
    // 每个线程独立的引擎
}
```

**`random_device` 可能很慢**。它通常要读系统熵池，频繁调用会成为瓶颈。正确用法是只用它播种一次：

```c++
std::mt19937 gen(std::random_device{}());   // 只调用一次
```

**分布的参数不能改**。`uniform_int_distribution` 的区间在构造时确定，想换区间要新建分布对象（或者用 `param()` 设置新参数）。

**`mt19937` 的种子不能太简单**。用 `1`、`2` 这种小种子，前几个输出可能不够随机。用 `random_device` 或 `seed_seq` 更好。

## 5. 相关章节

- [时间库](./Chrono.md)：用时间做种子
- [Algorithms](../STL/Algorithms.md)：`std::shuffle` 与 `std::sample`
- [多线程与并发](../Advance/Concurrency.md)：线程局部存储
- [文件与流](./Stream.md)：随机数输出
