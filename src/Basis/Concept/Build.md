# 编译与链接

C++ 程序从源代码到可执行文件要经过**预处理、编译、汇编、链接**四个阶段。理解这个过程，才能解释许多看似奇怪的现象：为什么头文件要加 include guard、为什么模板定义要放在头文件里、为什么会出现「未定义引用」错误。

## 一、编译流程

```
源文件 (.cpp)
    ↓ 预处理（Preprocessing）
预处理后的源文件 (.i)
    ↓ 编译（Compilation）
汇编代码 (.s)
    ↓ 汇编（Assembly）
目标文件 (.o / .obj)
    ↓ 链接（Linking）
可执行文件
```

| 阶段 | 工具 | 主要工作 |
| :--- | :--- | :--- |
| **预处理** | `cpp` | 展开 `#include`、替换宏、处理条件编译 |
| **编译** | `cc1plus` | 语法分析、语义检查、生成汇编代码 |
| **汇编** | `as` | 把汇编代码翻译成机器码 |
| **链接** | `ld` | 合并目标文件，解析符号引用，生成可执行文件 |

用 GCC 可以分步执行：

```bash
g++ -E main.cpp -o main.i      # 只做预处理
g++ -S main.i -o main.s        # 编译为汇编
g++ -c main.s -o main.o        # 汇编为目标文件
g++ main.o -o main             # 链接
```

日常使用的 `g++ main.cpp -o main` 会依次完成全部四步。

## 二、声明与定义

这是 C++ 中最基本也最重要的区分：

| | 声明 (Declaration) | 定义 (Definition) |
| :--- | :--- | :--- |
| 作用 | 告诉编译器「有这么个东西」 | 告诉编译器「它是什么」 |
| 是否分配内存 | 否 | 是（函数则是提供函数体） |
| 能否重复 | 可以多次 | 只能一次（ODR） |

```c++
// 声明
extern int global_var;        // 变量声明
int add(int a, int b);        // 函数声明

// 定义
int global_var = 42;          // 变量定义
int add(int a, int b) {       // 函数定义
    return a + b;
}
```

类定义、`inline` 函数、模板定义是例外——它们可以出现在多个翻译单元中。

## 三、单一定义规则（ODR）

**ODR（One Definition Rule）** 规定：

- 任何变量、函数、类类型、枚举在**整个程序中只能有一个定义**
- 同一个翻译单元内不能有重复定义
- 不同翻译单元中的定义必须**完全相同**（对于类、`inline` 函数、模板）

违反 ODR 属于**未定义行为**——编译器可能不报错，但链接时或运行时出现奇怪问题。

常见的 ODR 违规：

```c++
// header.h
int global_value = 42;   // 错误：这是定义，被多个 .cpp 包含会重复定义
```

正确做法是加 `extern` 声明，在一个 `.cpp` 中定义：

```c++
// header.h
extern int global_value;   // 声明

// one.cpp
int global_value = 42;     // 定义（只此一处）
```

## 四、头文件

### 1. 为什么要用头文件

C++ 采用**分离编译**模型：每个 `.cpp` 独立编译成目标文件。如果多个文件都要用同一个函数，就需要一种机制共享声明——这就是头文件。

```c++
// math_utils.h —— 声明
#pragma once
int add(int a, int b);
int multiply(int a, int b);

// math_utils.cpp —— 定义
#include "math_utils.h"
int add(int a, int b) { return a + b; }
int multiply(int a, int b) { return a * b; }

// main.cpp —— 使用
#include "math_utils.h"
int main() {
    return add(1, 2);
}
```

### 2. Include Guard

头文件可能被间接包含多次，导致重复定义。两种防护方式：

```c++
// 方式一：include guard（标准做法）
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

// 内容

#endif
```

```c++
// 方式二：#pragma once（非标准但广泛支持）
#pragma once

// 内容
```

`#pragma once` 更简洁，且能避免宏名冲突；include guard 是标准做法，可移植性更好。两者都可以用，但**不要混用**。

### 3. 头文件里应该放什么

| 可以放 | 不应放 |
| :--- | :--- |
| 函数声明 | 函数的普通定义（会重复定义） |
| 类定义 | 非 `inline` 的全局变量定义 |
| `inline` 函数定义 | — |
| 模板定义 | — |
| `constexpr` 变量 | — |
| 宏定义、类型别名 | — |

**模板和 `inline` 函数必须放在头文件**，因为编译器在实例化时需要看到完整定义。

## 五、链接

链接器的工作是**解析符号引用**：把每个目标文件中「引用了但未定义」的符号，与其它目标文件中「定义了」的符号对应起来。

### 1. 静态库与动态库

| | 静态库 | 动态库 |
| :--- | :--- | :--- |
| Linux 后缀 | `.a` | `.so` |
| Windows 后缀 | `.lib` | `.dll` |
| 链接时机 | 编译期，代码被复制进可执行文件 | 运行期，动态加载 |
| 可执行文件大小 | 较大 | 较小 |
| 更新库 | 需重新编译程序 | 替换库文件即可 |
| 部署 | 简单，单文件 | 需附带库文件 |

```bash
# 创建静态库
g++ -c lib.cpp -o lib.o
ar rcs libmylib.a lib.o
g++ main.cpp -L. -lmylib -o main

# 创建动态库
g++ -fPIC -c lib.cpp -o lib.o
g++ -shared lib.o -o libmylib.so
g++ main.cpp -L. -lmylib -o main
```

### 2. 常见链接错误

**「未定义引用」（undefined reference）**：声明了但没定义。

```
undefined reference to `add(int, int)'
```

原因通常是：忘记链接某个源文件或库、函数签名不匹配、模板定义没放在头文件。

**「重复定义」（multiple definition）**：同一个符号在多个目标文件中都有定义。

```
multiple definition of `global_value'
```

原因通常是：在头文件中定义了变量或非 `inline` 函数。

## 六、inline 与 static 的链接语义

这两个关键字都影响链接行为，但方式不同。

### 1. inline

`inline` 最初用于建议编译器内联展开，现代 C++ 中它的主要作用是**允许在多个翻译单元中重复定义**：

```c++
// header.h
inline int square(int x) { return x * x; }   // 多个 .cpp 包含也没问题
```

编译器会保证最终只保留一份。因此 **`inline` 函数可以放在头文件中**。

C++17 起还支持 `inline` 变量：

```c++
// header.h
inline int global_counter = 0;   // 头文件中定义变量，C++17 起可行
```

### 2. static

`static` 在 C++ 中有多重含义，取决于使用位置：

| 位置 | 含义 |
| :--- | :--- |
| 全局变量/函数前 | **内部链接**：只在当前翻译单元可见 |
| 局部变量前 | 静态存储期：函数退出后仍存在 |
| 类成员前 | 属于类而非对象 |

**内部链接**意味着同名符号在不同 `.cpp` 中互不冲突：

```c++
// a.cpp
static int counter = 0;   // 只在本文件可见

// b.cpp
static int counter = 0;   // 另一个独立的变量，不冲突
```

> [!WARNING]
> **头文件中的 `static` 是陷阱**
>
> 在头文件中定义 `static` 函数或变量，每个包含它的 `.cpp` 都会得到**独立的一份副本**：
>
> ```c++
> // header.h
> static int counter = 0;   // 每个 .cpp 各有一份！
> ```
>
> 这会导致内存浪费，且修改一个副本不会影响其它。应该用 `inline`（C++17）或 `extern` 声明。

## 七、条件编译

预处理指令可以根据条件选择性地编译代码：

```c++
// 平台判断
#ifdef _WIN32
    #include <windows.h>
#elif defined(__linux__)
    #include <unistd.h>
#endif

// 调试开关
#ifdef DEBUG
    #define LOG(msg) std::cout << msg << '\n'
#else
    #define LOG(msg)
#endif

// 版本判断
#if __cplusplus >= 202002L
    // C++20 及以上
#endif
```

标准库提供了 `__cplusplus` 宏表示语言版本（C++17 为 `201703L`，C++20 为 `202002L`）。

## 八、注意事项

- **头文件必须加 include guard**。否则重复包含会导致重复定义。
- **变量定义不要放在头文件**。应使用 `extern` 声明 + 单处定义，或 C++17 的 `inline` 变量。
- **模板和 `inline` 函数必须放头文件**。编译器实例化时需要看到完整定义。
- **`static` 在头文件中会复制**。每个翻译单元各一份，通常不是想要的效果。
- **`c_str()` 和 `data()` 的指针会失效**。容器修改后指针可能悬垂。
- **ODR 违规是未定义行为**。编译器可能不报错，问题在链接期或运行期才暴露。
- **`#pragma once` 非标准**。虽然广泛支持，但严格可移植的代码应用 include guard。
- **编译单元是独立编译的**。修改一个 `.cpp` 只需重新编译它，这是增量编译的基础。

## 九、相关章节

- [编译运行](./Compile_Run.md)：编译器与基本编译命令
- [预处理器](./Preprocessor.md)：宏定义与预处理指令
- [命名空间](./Namespace.md)：符号的组织与查找
- [模板](../Template.md)：为什么模板定义要放在头文件
