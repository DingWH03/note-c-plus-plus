# C++ 程序的编译与运行（Compilation and Execution）

## 引言

C++ 是一种 **编译型语言（compiled language）**。
这意味着在执行程序之前，必须先将源代码 (`.cpp`) 转换为计算机可以理解的机器指令（可执行文件）。

整个过程通常包括以下几个阶段：

```text
源代码 (.cpp)
   ↓
预处理（Preprocessing）
   ↓
编译（Compilation）
   ↓
汇编（Assembly）
   ↓
链接（Linking）
   ↓
可执行文件（Executable）
   ↓
运行（Execution）
```

## 编译过程的四个主要阶段

| 阶段                 | 作用                 | 示例命令                        | 输出文件     |
| ------------------ | ------------------ | --------------------------- | -------- |
| 预处理（Preprocessing） | 展开宏、包含头文件、删除注释     | `g++ -E main.cpp -o main.i` | `main.i` |
| 编译（Compilation）    | 将预处理后的代码转换为汇编代码    | `g++ -S main.i -o main.s`   | `main.s` |
| 汇编（Assembly）       | 将汇编代码转换为机器指令（目标文件） | `g++ -c main.s -o main.o`   | `main.o` |
| 链接（Linking）        | 将多个目标文件与库链接生成可执行文件 | `g++ main.o -o main`        | `main`   |

> 可以通过这些命令观察 C++ 源码在每个阶段的输出，理解“编译器到底做了什么”。

## 完整编译命令示例

最常用的编译命令只需一行：

```bash
g++ main.cpp -o main
```

* `g++`：GNU C++ 编译器（GCC 的 C++ 前端）
* `main.cpp`：源文件
* `-o main`：指定输出文件名（不写则默认为 `a.out`）

运行程序：

```bash
./main
```

输出：

```text
Hello, World!
```

## 多文件编译示例

大型项目通常由多个源文件组成：

```text
project/
├── main.cpp
├── add.cpp
└── add.h
```

### 编译方式一：一步完成

```bash
g++ main.cpp add.cpp -o main
```

### 编译方式二：分步编译

```bash
g++ -c main.cpp -o main.o
g++ -c add.cpp -o add.o
g++ main.o add.o -o main
```

> 分步编译的好处是：当部分文件修改时，只需重新编译变动的部分，而不是整个项目。

## 运行机制详解

C++ 程序编译完成后，会生成一个**可执行文件**（如 `main` 或 `main.exe`）。
程序的运行流程如下：

```text
操作系统加载可执行文件
   ↓
执行程序的入口函数 main()
   ↓
程序逻辑执行
   ↓
return 返回值传递给操作系统
```

返回值 `0` 通常表示正常退出，非零值表示错误或异常终止。

## 编译器与构建工具

| 工具                | 说明                      | 适用平台                           |
| ----------------- | ----------------------- | ------------------------------ |
| **g++**           | GNU 编译器套件中的 C++ 编译器     | Linux / macOS / Windows（MinGW） |
| **clang++**       | LLVM 项目的 C++ 编译器，语法检查友好 | 跨平台                            |
| **MSVC (cl.exe)** | 微软的 C++ 编译器             | Windows                        |
| **CMake**         | 构建系统生成工具，可跨平台生成编译脚本     | 所有平台                           |
| **Makefile**      | 编译自动化脚本，用于管理依赖          | Linux / macOS                  |

## 使用 IDE 进行编译运行

### 常见 IDE

| IDE           | 编译器                 | 特点            |
| ------------- | ------------------- | ------------- |
| Visual Studio | MSVC                | Windows 下功能最全 |
| CLion         | CMake + g++/clang++ | 跨平台支持强        |
| VS Code       | 可结合 `g++`、`clang++` | 轻量灵活          |
| Qt Creator    | qmake / CMake       | 适用于图形界面开发     |

### 在 IDE 中执行的隐含步骤

当你点击“运行（Run）”时，IDE 实际执行了以下操作：

1. 调用编译器生成目标文件；
2. 执行链接步骤生成可执行程序；
3. 启动该可执行文件并显示输出。

## 常见编译选项

| 选项           | 含义            | 示例                            |
| ------------ | ------------- | ----------------------------- |
| `-o <file>`  | 指定输出文件名       | `g++ main.cpp -o app`         |
| `-Wall`      | 启用所有警告信息      | `g++ -Wall main.cpp`          |
| `-g`         | 生成调试信息        | `g++ -g main.cpp`             |
| `-O2`        | 优化等级 2，提高执行效率 | `g++ -O2 main.cpp`            |
| `-std=c++17` | 指定 C++ 标准版本   | `g++ -std=c++17 main.cpp`     |
| `-I <dir>`   | 添加头文件搜索路径     | `g++ -I include main.cpp`     |
| `-L <dir>`   | 添加库文件路径       | `g++ -L lib -lmylib main.cpp` |
| `-D <macro>` | 定义宏           | `g++ -DDEBUG main.cpp`        |

## 扩展

* **交叉编译 (Cross Compilation)**：在一台机器上为另一架构（如 ARM、RISC-V）生成可执行程序。
  示例：

  ```bash
  riscv64-linux-gnu-g++ main.cpp -o main_rv
  ```

* **静态链接 vs 动态链接**：

  * 静态链接 (`.a`)：在编译时嵌入库文件。
  * 动态链接 (`.so` / `.dll`)：运行时动态加载库。
