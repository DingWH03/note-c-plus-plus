# 预处理器 (Preprocessor)

C++ 预处理器（Preprocessor）是 C++ 编译过程中的第一阶段。它负责处理源代码中所有以 `#` 开头的指令，在实际编译（生成目标代码）之前对源代码进行文本替换、文件包含、条件编译等操作。

## 一、预处理指令 (Directives)

预处理指令以 `#` 符号开头，并且占据一行。它们不是 C++ 语句，因此不需要以分号 (`;`) 结尾。

| 指令 | 目的 | 示例 |
| :--- | :--- | :--- |
| `#include` | 文件包含。将指定文件的内容插入到当前位置。 | `#include <iostream>` |
| `#define` | 宏定义。定义常量宏或函数宏。 | `#define PI 3.14159` |
| `#undef` | 取消宏定义。 | `#undef PI` |
| `#if` / `#elif` / `#else` / `#endif` | 条件编译。根据条件决定是否编译某段代码。 | `#if VERSION >= 2` |
| `#ifdef` / `#ifndef` | 条件编译。检查宏是否已被定义。 | `#ifndef DEBUG_MODE` |
| `#error` | 停止编译，并输出指定的错误信息。 | `#error "Platform not supported"` |
| `#warning` | 输出指定的警告信息，继续编译（非标准，但广泛支持）。 | `#warning "Legacy code detected"` |
| `#pragma` | **编译器特定**的指令，用于向编译器发出特殊指令。 | `#pragma once`, `#pragma warning` |

## 二、宏定义 (`#define`)

宏定义是预处理器最重要的功能之一，它执行的是**文本替换**。

1. 对象式宏 (Object-like Macros) - 定义常量

    ```cpp
    #define BUFFER_SIZE 1024
    // 预处理后：所有 BUFFER_SIZE 都会被替换为 1024
    ```

2. 函数式宏 (Function-like Macros) - 定义类似函数的替换

    使用时注意：必须在参数和整个宏体周围加上括号，以避免运算符优先级问题。

    ```cpp
    #define SQUARE(x) ( (x) * (x) )

    // 错误示例：#define SQUARE(x) x * x
    // SQUARE(1 + 2) -> 1 + 2 * 1 + 2 -> 5 (错误!)
    // 正确示例：SQUARE(1 + 2) -> ( (1 + 2) * (1 + 2) ) -> 9 (正确!)
    ```

    > 最佳实践: 除非必要，在 C++ 中应使用 `const` 或 `constexpr` 变量代替对象式宏，使用 **内联函数 (`inline`)** 或 **模板** 代替函数式宏，以获得类型检查和更好的可读性。

3. 宏的特殊操作符

    * `#` (字符串化运算符 / Stringizing Operator)：将宏参数转换为字符串字面值。

        ```cpp
        #define MESSAGE(x) #x
        // std::cout << MESSAGE(Hello World);  ->  输出: Hello World
        ```

    * `##` (连接运算符 / Token-Pasting Operator)：连接两个宏标记（token），形成一个新的标记。

        ```cpp
        #define GLUE(a, b) a ## b
        // int GLUE(my, Variable) = 10;  ->  int myVariable = 10;
        ```

## 三、文件包含 (`#include`)

用于将一个文件的内容插入到另一个文件中。

1. 尖括号 (`<>`)：用于包含**标准库头文件**。编译器会在标准系统路径下查找。

    ```cpp
    #include <iostream>
    ```

2. 双引号 (`""`)：用于包含**用户自定义头文件**。编译器首先在当前源文件目录查找，然后才在标准系统路径下查找（具体的查找路径取决于编译器设置）。

    ```cpp
    #include "my_header.h"
    ```

## 四、条件编译 (Conditional Compilation)

条件编译允许根据预处理器定义的条件，决定是否将一段代码发送给编译器。这对于平台移植、调试开关和头文件保护至关重要。

1. 头文件保护 (Header Guards) - 经典方式

    用于防止头文件被重复包含，导致重定义错误。

    ```cpp
    #ifndef MY_HEADER_H // IF Not DEFined
    #define MY_HEADER_H

    // 头文件内容

    #endif // MY_HEADER_H
    ```

2. 基于宏的条件

    ```cpp
    #define DEBUG_MODE // 开启调试模式

    // ...

    #ifdef DEBUG_MODE   // 如果 DEBUG_MODE 宏已定义，则编译以下代码
        std::cout << "Debugging output." << std::endl;
    #endif

    #ifndef RELEASE_MODE // 如果 RELEASE_MODE 宏未定义，则编译以下代码
        // ...
    #endif
    ```

3. 多分支条件

    ```cpp
    #define OS_LINUX 1 // 假设在 Linux 平台编译

    #if OS_LINUX
        #include <unistd.h>
    #elif OS_WINDOWS
        #include <windows.h>
    #else
        #error "Unsupported OS!"
    #endif
    ```

      * `#if` 后面要求表达式求值为非零（真）或零（假）。表达式中未定义的宏会被当作零处理。
      * `defined` 运算符：`#if defined(MACRO_NAME)` 与 `#ifdef MACRO_NAME` 效果相似，但 `defined` 可以用于更复杂的布尔表达式中，例如 `#if defined(A) && !defined(B)`。

## 五、编译器特定指令 (`#pragma`)

`#pragma` 指令是一种特殊的预处理指令，用于向编译器提供特定于编译器或平台的指示。由于它们是非标准的（不属于 C++ 语言规范），因此其功能和语法可能在不同编译器之间有所不同。

1. `#pragma once`

    * **目的：** 实现头文件保护，防止头文件被重复包含。
    * **优点：** 代码更简洁，不会污染全局宏命名空间，并且大多数现代编译器（GCC, Clang, MSVC 等）都支持且执行效率更高（编译器可以直接比较文件名或路径）。
    * **用法：** 放在头文件的最开始。

        ```cpp
        // my_header.h
        #pragma once

        struct MyData { /* ... */ };
        ```

    * **对比 `#ifndef`：**
    * `#ifndef` 是标准 C++ 的一部分，可移植性最高。
    * `#pragma once` 是事实上的行业标准，使用更方便，但技术上非标准。在新的代码中通常推荐使用它。

2. `#pragma warning` (主要用于 MSVC)

    * **目的：** 临时修改或控制编译器警告的行为。
    * **关键语法：**
    * `#pragma warning( disable : 警告编号 )`：禁用特定的警告。
    * `#pragma warning( error : 警告编号 )`：将特定的警告视为错误。
    * `#pragma warning( push )`：保存当前的警告状态到堆栈。
    * `#pragma warning( pop )`：恢复最近一次保存的警告状态。
    * **用途示例：** 临时禁用第三方库中产生的警告。

        ```cpp
        #pragma warning( push )         // 保存当前警告状态
        #pragma warning( disable : 4100 ) // 禁用 MSVC 警告 C4100 (未使用的形参)

        void unused_param_func(int x)
        {
            // ... code that intentionally doesn't use x
        }

        #pragma warning( pop )          // 恢复之前的警告状态
        ```

    > GCC/Clang 对应功能： 它们通常使用 `#pragma GCC diagnostic push`/`pop` 和 `#pragma GCC diagnostic ignored "-Wxxx"`。

3. 其它特定 `#pragma`

    * `#pragma pack(n)`：指定结构体成员的对齐字节数，常用于跨平台或与硬件交互。
    * `#pragma comment(...)`：向目标文件中插入注释信息或链接器指令（如链接特定的库文件）。
