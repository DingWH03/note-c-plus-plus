# C++ 程序结构（Program Structure）

## 引言

C++ 程序的结构决定了它的组织方式、执行流程和可维护性。
从最小的“Hello, World!”程序开始，我们可以认识一个 C++ 程序通常包含的**头文件、命名空间、函数定义、语句块**等组成部分。

## 一个最小可运行的 C++ 程序

```cpp
#include <iostream>   // 头文件：引入输入输出库
using namespace std;  // 使用标准命名空间

int main() {          // 主函数：程序执行入口
    cout << "Hello, World!" << endl;  // 输出语句
    return 0;         // 返回值：表示程序是否成功结束
}
```

### 程序执行流程说明

| 行号 | 内容                                 | 说明                        |
| -- | ---------------------------------- | ------------------------- |
| 1  | `#include <iostream>`              | 预处理指令，引入标准输入输出库           |
| 2  | `using namespace std;`             | 告诉编译器默认使用 `std` 命名空间      |
| 3  | `int main() {`                     | 主函数是程序的入口点（返回类型必须是 `int`） |
| 4  | `cout << "Hello, World!" << endl;` | 使用 `cout` 输出字符串并换行        |
| 5  | `return 0;`                        | 返回 `0` 表示程序正常结束           |

## 程序的基本组成部分

一个完整的 C++ 程序通常由以下几个部分组成：

1. **预处理指令（Preprocessor Directives）**
   在编译前由预处理器执行，例如：

   ```cpp
   #include <iostream>
   #define PI 3.14159
   ```

2. **命名空间（Namespace）**
   命名空间用于避免名称冲突：

   ```cpp
   namespace myspace {
       int x = 10;
   }
   ```

3. **主函数 `main()`**
   所有可执行 C++ 程序都必须有且仅有一个 `main()` 函数。

4. **函数定义（Functions）**
   函数是代码逻辑的基本单元，示例：

   ```cpp
   int add(int a, int b) {
       return a + b;
   }
   ```

5. **变量与语句（Variables & Statements）**
   程序的逻辑通过语句执行，通过变量存储数据。

   ```cpp
   int sum = add(2, 3);
   cout << sum << endl;
   ```

## C++ 程序的执行入口

* 程序从 `main()` 函数开始执行。
* `main()` 的返回值会传递给操作系统。
* 常见的两种形式：

  ```cpp
  int main() {
      // 无参数版本
      return 0;
  }

  int main(int argc, char* argv[]) {
      // 带命令行参数的版本
      // argc 表示参数数量
      // argv 表示参数数组
      return 0;
  }
  ```

## 程序的语句块与作用域

语句块使用花括号 `{}` 表示，定义一个新的作用域（scope）：

```cpp
int x = 10;
{
    int x = 20;   // 内层作用域的 x 隐藏外层变量
    cout << x;    // 输出 20
}
cout << x;        // 输出 10
```

作用域是 C++ 的重要概念，影响变量的可见性与生命周期。

## 程序结构的逻辑层次（层级模型）

| 层级   | 名称              | 说明        |
| ---- | --------------- | --------- |
| 顶层   | 预处理部分           | 定义宏、引入头文件 |
| 全局层  | 命名空间、全局变量、函数声明  |           |
| 主函数层 | 程序入口、主流程        |           |
| 函数层  | 逻辑实现与局部变量       |           |
| 语句层  | 执行单元（表达式、控制语句等） |           |

这是一种从外到内的逻辑组织结构。良好的层级划分有助于程序的模块化。

## 程序文件结构（大型项目）

在真实项目中，一个 C++ 程序通常被拆分为多个文件：

```text
project/
├── main.cpp        // 程序入口
├── math/
│   ├── add.cpp
│   └── add.h
└── utils/
    ├── log.cpp
    └── log.h
```

通过 **头文件 (.h/.hpp)** 声明接口，**源文件 (.cpp)** 实现功能：

* add.h

    ```cpp
    #ifndef ADD_H
    #define ADD_H

    int add(int a, int b);

    #endif
    ```

* add.cpp

    ```cpp
    #include "add.h"
    int add(int a, int b) {
        return a + b;
    }
    ```

* main.cpp

    ```cpp
    #include <iostream>
    #include "add.h"

    int main() {
        std::cout << add(3, 4) << std::endl;
        return 0;
    }
    ```

> 大型工程中，通常直接或间接地使用`Cmake`或`make`进行编译和项目管理。
