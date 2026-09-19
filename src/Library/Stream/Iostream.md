# 标准输入输出流（iostream）

C++ 的输入输出体系以 `<iostream>` 头文件为核心。它定义了程序与外部设备（例如键盘与显示器）交互的最基本机制，是所有流操作的基础。标准输入输出流提供了一组通用接口，用于在内存与外部设备之间以流的形式进行数据传输。

## 一、标准流对象

C++ 预定义了四个标准流对象，分别用于输入、输出与错误处理：

| 对象名    | 所属类       | 方向 | 描述                |
| ------ | --------- | -- | ----------------- |
| `cin`  | `istream` | 输入 | 从标准输入（键盘）读取数据     |
| `cout` | `ostream` | 输出 | 向标准输出（控制台）写入数据    |
| `cerr` | `ostream` | 输出 | 向标准错误输出写入数据（不带缓冲） |
| `clog` | `ostream` | 输出 | 向标准错误输出写入数据（带缓冲）  |

`cin` 和 `cout` 是最常用的两个流对象，它们对应于 C 语言中的 `stdin` 与 `stdout`。而 `cerr` 与 `clog` 则用于错误与日志输出：

* `cerr` 立即输出，不经过缓冲，适合错误提示；
* `clog` 使用缓冲区，适合记录日志或调试信息。

## 二、输入与输出运算符

C++ 使用运算符重载机制，使输入与输出操作更自然直观：

```cpp
int a;
std::cin >> a;       // 输入，将数据流入变量 a
std::cout << a << '\n'; // 输出，将数据流出到控制台
```

`>>` 是提取运算符（extraction operator），从输入流中提取数据；
`<<` 是插入运算符（insertion operator），向输出流中插入数据。

这两个运算符被广泛重载，可用于所有基本类型与标准容器（通过 `operator<<` 的重载），使得流式编程成为 C++ 的一大特色。例如：

```cpp
std::string name;
int age;
std::cin >> name >> age;
std::cout << "Name: " << name << ", Age: " << age << std::endl;
```

每个操作符都返回流对象自身的引用，从而允许**链式调用**。

## 三、流缓冲与刷新机制

所有标准输出流都带有缓冲区。输出内容首先被写入缓冲区，当缓冲区满、遇到换行符、调用 `flush` 或程序结束时，缓冲内容才会真正输出到设备。

常见的刷新方式有：

* `std::endl`：输出换行并刷新缓冲区；
* `std::flush`：仅刷新缓冲区；
* `std::ends`：输出一个空字符并刷新缓冲区。

```cpp
std::cout << "Hello, world!" << std::endl;  // 换行并刷新
```

这种延迟输出机制提高了效率，但在交互程序中需要注意及时刷新，否则可能出现输出滞后。

## 四、格式化输出

C++ 流提供了丰富的格式化控制机制，用于调整输出格式。最常见的控制包括：

1. **操纵符（Manipulators）**
   通过 `<iomanip>` 头文件可以使用 `setw`, `setfill`, `setprecision`, `fixed`, `scientific` 等控制输出格式：

   ```cpp
   #include <iomanip>
   double pi = 3.14159265;
   std::cout << std::fixed << std::setprecision(3) << pi;  // 输出 3.142
   ```

2. **流格式标志**
   可以通过 `setf()`、`unsetf()` 修改流的格式状态，例如控制对齐方式、进制表示、是否显示符号等：

   ```cpp
   std::cout.setf(std::ios::showpos);
   std::cout << 42;  // 输出 +42
   ```

## 五、`ios` 与 `ios_base`

`ios_base` 是所有流类的根基，提供全局的格式化与状态管理功能，如：

* 格式标志（`fmtflags`）
* I/O 状态（`iostate`）
* 用户自定义存储（`xalloc`, `iword`, `pword`）

而 `ios` 则在此基础上增加了缓冲区管理与错误处理机制，是 `istream` 与 `ostream` 的共同父类。

## 六、标准流的重定向

C++ 允许将标准输入输出流重定向到文件或其他流对象，实现灵活的数据通道。例如：

```cpp
std::ofstream fout("log.txt");
std::streambuf* backup = std::cout.rdbuf(fout.rdbuf());  // 重定向 cout 到文件
std::cout << "This will be written to file." << std::endl;
std::cout.rdbuf(backup); // 恢复
```

这种技术常用于日志系统或单元测试环境，能够让程序在不修改逻辑的情况下改变输出目标。

## 七、非格式化输入与其他高级输入方法

提取运算符 `>>`（格式化输入）在读取数据时会跳过开头的空白符，并且遇到空格、制表符或换行符时会停止，这不适用于读取包含空格的字符串或需要精确控制读取字节数的情况。C++ iostream 库提供了一系列**非格式化输入函数**来满足这些需求。

### 1\. 读取整行：`getline()`

`std::getline()` 是最常用的非格式化输入函数之一，用于读取一行文本，包括其中的空格。

| 函数签名 | 描述 |
| :--- | :--- |
| `std::getline(istream& is, std::string& str, char delim)` | 从输入流 `is` 中读取字符，直到遇到指定的分隔符 `delim`（默认为 `\n`），并将读取的内容存入 `str`。**分隔符会被读取，但不会存入 `str`。** |
| `std::getline(istream& is, std::string& str)` | 使用默认的分隔符 `\n` 读取一行。 |

```cpp
#include <iostream>
#include <string>

std::string fullName;
std::cout << "Enter your full name: ";
// 读取整行输入，直到遇到换行符
std::getline(std::cin, fullName);
std::cout << "Welcome, " << fullName << std::endl;
```

### 2\. 读取单个字符：`get()`

`get()` 函数用于从流中读取单个字符，且**不会跳过空白符**。

| 函数签名 | 描述 |
| :--- | :--- |
| `is.get(char& ch)` | 将流中的下一个字符存入 `ch`。 |
| `is.get()` | 返回流中的下一个字符（作为 `int` 类型），或返回 `EOF`（文件结束）。 |

```cpp
char c1, c2;
std::cin.get(c1); // 读取第一个字符（可能是空格）
std::cin.get(c2); // 读取第二个字符
std::cout << "c1: " << c1 << ", c2: " << c2 << std::endl;
```

### 3\. 忽略流中字符：`ignore()`

`ignore()` 函数常用于清除输入缓冲区中残余的字符，特别是在混合使用格式化输入（`>>`）和 `getline()` 时。

| 函数签名 | 描述 |
| :--- | :--- |
| `is.ignore(streamsize count, char delim)` | 从输入流中丢弃最多 `count` 个字符，直到遇到指定的分隔符 `delim`。**分隔符也会被丢弃**。 |

```cpp
int age;
char gender;

std::cin >> age;
// 假设用户输入 "25\n"
// `>> age` 读取了 25，但换行符 '\n' 仍留在缓冲区。

// 清除缓冲区中剩余的字符，直到遇到换行符
std::cin.ignore(10000, '\n');

std::cout << "Enter gender (M/F): ";
std::cin.get(gender);
// 现在 `get()` 可以正确读取新的输入，而不是残留在缓冲区的 '\n'
```

> **注意：** `10000` 是一个较大的数字，确保能够处理大多数行长度。

### 4\. 窥视下一个字符：`peek()`

`peek()` 函数用于查看流中下一个可用的字符，但**不会将其从流中移除**。

```cpp
// 假设流中下一个字符是 'H'
char nextChar = std::cin.peek();
// nextChar 是 'H'
// 'H' 仍然在输入流中，可供下次操作读取
```

### 5\. 高级操作：`read()` 与 `gcount()`

`read()` 是一个底层的非格式化输入函数，用于读取指定数量的原始字节数据，常用于处理二进制文件。

| 函数签名 | 描述 |
| :--- | :--- |
| `is.read(char* s, streamsize n)` | 从流中读取 `n` 个字节到内存地址 `s` 开始的缓冲区。 |

在调用 `read()` 或任何非格式化输入函数后，可以使用 `gcount()` 来获取**最近一次非格式化输入操作实际读取的字符数量**。

```cpp
char buffer[10];
// 尝试从流中读取 10 个字节
std::cin.read(buffer, 10);
// 报告实际读取的字节数（可能小于 10，例如遇到文件末尾）
std::streamsize actualRead = std::cin.gcount();
```
