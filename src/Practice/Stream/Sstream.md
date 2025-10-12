# 字符串流（sstream）

在 C++ 中，**字符串流（String Streams）** 是一种特殊的内存流，它将字符串抽象为输入输出流，从而允许程序像操作文件或控制台那样读写字符串。字符串流的核心头文件为 `<sstream>`，主要用于格式化文本、解析数据或在内存中临时缓存输出内容。相比直接操作字符串，字符串流提供了统一、可扩展且安全的接口。

## 一、字符串流的分类

字符串流继承自标准流类体系，但其输入输出目标不是物理设备，而是内存中的字符串。C++ 提供三种主要类型的字符串流：

1. **`istringstream`**
   面向输入的字符串流，用于从字符串中提取数据。它将字符串当作数据源，实现类似于从文件读取的行为。

2. **`ostringstream`**
   面向输出的字符串流，用于向字符串写入数据。它提供了格式化输出的能力，将数据以文本形式存储在内存字符串中。

3. **`stringstream`**
   同时支持输入和输出，可以在同一个字符串流对象上进行读写操作。适合需要在内存中反复解析和构建字符串的场景。

这种设计与标准流对象保持一致，使得对内存字符串的操作与文件或控制台的操作具有统一的接口和方法。

## 二、基本用法

使用字符串流时，通常的步骤包括创建流对象、向流中写入或从流中读取数据，然后获取最终字符串或解析结果。示例：

```cpp
#include <sstream>
#include <string>
#include <iostream>

std::ostringstream oss;
oss << "Name: " << "Alice" << ", Age: " << 30;
std::string result = oss.str();
std::cout << result << std::endl;  // 输出 "Name: Alice, Age: 30"
```

```cpp
std::string data = "42 3.14 hello";
std::istringstream iss(data); // 从已有的字符串中建立字符流

int i;
double d;
std::string s;
iss >> i >> d >> s;  // 从字符串中提取整数、浮点数和字符串
```

通过这种方式，程序可以像处理标准流那样操作字符串，利用 `>>` 和 `<<` 运算符进行格式化输入输出。

## 三、流的格式化能力

字符串流继承自标准流类，因此支持所有流的格式化特性：

* 可以使用操纵符（`setw`, `setprecision`, `fixed` 等）控制输出格式。
* 可以通过 `setf()`、`unsetf()` 设置流的格式标志，实现对齐方式、数值进制或符号显示的控制。
* 可以链式调用输入输出操作，使字符串构建或解析更加简洁。

示例：

```cpp
#include <iomanip>
std::ostringstream oss2;
double pi = 3.14159265;
oss2 << std::fixed << std::setprecision(2) << pi;
std::cout << oss2.str();  // 输出 "3.14"
```

## 四、常见操作方法

字符串流提供了丰富的成员函数，用于在内存字符串上实现精细控制：

1. **获取内容**

   * `str()`：返回当前流中的字符串内容。
   * `str(const std::string&)`：设置或替换流的内容，为流重新赋值。

2. **清空或重置流**

   * `clear()`：重置流的状态，使其可重新进行读写操作。
   * `seekg()` / `seekp()`：调整读写位置，实现从特定位置开始读取或写入。

3. **读取与写入**

   * 使用 `>>` 从字符串中提取数据，使用 `<<` 向流中插入数据。
   * 可以使用 `getline()` 从字符串中逐行读取内容，与文件或控制台操作保持一致。

示例：

```cpp
std::stringstream ss("123 456");
int a, b;
ss >> a >> b;  // 提取两个整数
ss.str("");    // 清空流内容
ss.clear();    // 重置状态
ss << "New content";
```

## 五、应用实例

### 1. 解析数组

解析`"[2,-1,3,0,12]"`这样的字符串为数组：

```cpp
std::string s = "[2,-1,3,0,12]";
std::stringstream ss(s);
char ch;
std::vector<int> nums;
std::string temp;

// 读取第一个字符，应该是 '['
ss >> ch;
if (ch != '[') {
    std::cerr << "Invalid input format!" << std::endl;
    return 1;
}

while (ss >> ch) {
    if (ch == ',' || ch == ']') {
        if (!temp.empty()) {
            nums.push_back(std::stoi(temp)); // 转换字符串为整数
            temp.clear();
        }
        if (ch == ']') break; // 数组结束
    } else {
        temp += ch; // 累积数字字符，包括负号
    }
}

// 输出结果
std::cout << "Numbers: ";
for (int n : nums) std::cout << n << " ";
std::cout << std::endl;
```
