# 命名规则 (Naming Conventions)

命名规则是代码风格的重要组成部分，它能显著提高代码的可读性、可维护性和团队协作效率。在 C++ 社区中，存在多种主流命名规范（如 Google Style、LLVM Style）。

## 核心原则

1. **一致性 (Consistency)：**
   在一个项目或代码库中，一旦选定了一种命名风格，就必须始终如一地使用它，避免混用多种风格导致的混乱。

2. **描述性 (Descriptive)：**
   名称应清晰地描述其所代表的实体的用途或含义，便于其他开发者理解代码。

3. **避免歧义 (Avoid Ambiguity)：**
   避免使用容易引起误解的缩写或模糊单词，例如 `tmp`、`data` 等，必要时增加上下文信息。

## 常用命名风格

| 风格名称                 | 描述                               | 示例                |
| :------------------- | :------------------------------- | :---------------- |
| **PascalCase** / 大驼峰 | 每个单词首字母大写，不使用分隔符。通常用于类型名称。       | `MyClassName`     |
| **camelCase** / 小驼峰  | 第一个单词首字母小写，其余单词首字母大写。通常用于变量和函数。  | `calculateSum`    |
| **snake_case** / 下划线 | 所有字母小写，单词之间用下划线 `_` 分隔。          | `current_balance` |
| **UPPER_CASE** / 全大写 | 所有字母大写，单词之间用下划线 `_` 分隔。通常用于常量或宏。 | `MAX_SIZE`        |

## 命名实体规范

| 实体类型                       | 推荐风格                    | 示例                                          | 备注                                      |
| :------------------------- | :---------------------- | :------------------------------------------ | :-------------------------------------- |
| **类 (Class)**              | PascalCase              | `Customer`, `FileManager`                   | 类名通常是**名词**。                            |
| **结构体 (Struct)**           | PascalCase              | `Point2D`, `StudentInfo`                    | 与类一致，都是用户自定义类型。                         |
| **函数/方法 (Function)**       | camelCase               | `calculateArea()`, `getFileSize()`          | 通常是**动词或动宾短语**，表示行为。                    |
| **局部变量 (Local Variable)**  | camelCase               | `age`, `totalCount`, `isValid`              | 保持简洁、描述清晰。                              |
| **成员变量 (Member Variable)** | camelCase + 后缀或前缀       | `m_value` 或 `value_`                        | 用于区分成员变量和局部变量，提高可读性。                    |
| **常量 (Constants)**         | UPPER_CASE              | `MAX_ITEMS`, `PI_VALUE`                     | 用于 `#define` 或 `const/constexpr` 编译期常量。 |
| **枚举 (Enums/Enum Class)**  | PascalCase              | `ColorType`                                 | 枚举类型使用 PascalCase。                      |
| **枚举值 (Enum Values)**      | PascalCase 或 UPPER_CASE | `Red`, `Green` 或 `COLOR_RED`, `COLOR_GREEN` | 建议加类型前缀避免冲突。                            |
| **命名空间 (Namespace)**       | snake_case 或 全小写        | `my_project`, `network`                     | 命名空间通常是项目名或模块名。                         |
| **宏 (Macros)**             | UPPER_CASE              | `#define DEBUG_MODE`                        | 宏是全局预处理符号，必须与变量/常量区分。                   |

## 命名限制与最佳实践

### 1. C++ 保留标识符

C++ 标准保留了某些名称供编译器和标准库使用，用户代码**禁止使用**：

#### 1.1 全局范围（Global Scope）保留

* **包含两个连续下划线 `__` 的名称**
  示例：`__myVar`, `My__Class`
* **以下划线 `_` 开头，紧跟大写字母**
  示例：`_PrivateMember`, `_Max_Value`

#### 1.2 全局命名空间（Global Namespace）保留

* **以下划线 `_` 开头的名称**
  示例：`_name`, `_a_local_var`
  在局部作用域内可能安全，但全局范围内使用会违反标准。最佳实践：**避免任何下划线开头的自定义名称**。

#### 1.3 关键字 (Keywords)

绝对禁止使用 C++ 关键字作为标识符：

| C++ 关键字（部分）                                                            |                |             |            |
| :--------------------------------------------------------------------- | :------------- | :---------- | :--------- |
| `alignas`                                                              | `decltype`     | `if`        | `return`   |
| `auto`                                                                 | `do`           | `inline`    | `short`    |
| `bool`                                                                 | `double`       | `int`       | `sizeof`   |
| `break`                                                                | `else`         | `long`      | `struct`   |
| `case`                                                                 | `enum`         | `namespace` | `switch`   |
| `char`                                                                 | `extern`       | `new`       | `template` |
| `class`                                                                | `float`        | `private`   | `this`     |
| `const`                                                                | `for`          | `public`    | `try`      |
| `continue`                                                             | `goto`         | `register`  | `void`     |
| `default`                                                              | `thread_local` | `virtual`   | `while`    |

> **C++11/14/17/20 新增**：`constexpr`, `noexcept`, `override`, `concept`

### 2. 命名技巧与长度

* **避免单字母名称**：除循环迭代器（`i, j, k`）和数学变量（`x, y, z`）外，应使用描述性名称。

  **差：** `int s;`
  **好：** `int studentCount;`

* **避免匈牙利命名法**：现代 C++ 不推荐通过前缀标注类型，如 `iAge`、`szName`。类型信息已由编译器和 IDE 提供。

* **布尔值命名**：布尔变量或函数应以动词开头，清晰表达状态。
  示例：`bool isValid;`, `bool hasError();`, `bool isFinished;`

* **Getter/Setter 命名**：遵循约定提高可读性。

  * Getter: `getValue()` 或 `value()` (变量名 `value_`)
  * Setter: `setValue(int newValue)` 或 `set_value(int newValue)`

### 3. 文件命名规范

| 风格         | 头文件 (.h)         | 源文件 (.cpp)        | 备注                           |
| :--------- | :--------------- | :---------------- | :--------------------------- |
| PascalCase | `FileManager.h`  | `FileManager.cpp` | 常用于 Qt 等 IDE/框架，与类名一致        |
| snake_case | `file_manager.h` | `file_manager.cc` | **Google 风格推荐**，更便于跨平台和工具链使用 |

**头文件保护 (Header Guards)**：确保文件内容只被编译一次。推荐使用 **全大写 + 项目/目录/文件名** 的宏。

```cpp
// network/tcp_socket.h
#ifndef MYPROJECT_NETWORK_TCP_SOCKET_H
#define MYPROJECT_NETWORK_TCP_SOCKET_H

// ... 代码 ...

#endif // MYPROJECT_NETWORK_TCP_SOCKET_H
```

### 示例代码

```cpp
// 1. 类型 (PascalCase)
class EmployeeManager {
private:
    // 2. 成员变量 (推荐使用后缀，避免保留标识符)
    int employee_count_;

    // 错误示范（应避免）
    // int __bad_var;           // 双下划线
    // int _AnotherBadVar;      // 下划线开头 + 大写
    // int _safe_in_local_scope; // 全局范围不安全

public:
    // 3. 函数/方法 (camelCase)
    bool isValidEmployee(const std::string& name) {
        // 4. 局部变量 (camelCase)
        int maxHours = 40;

        if (this->employee_count_ < maxHours) { // 使用 this-> 或后缀区分成员
            return true;
        }
        return false;
    }
};

// 5. 宏常量 (UPPER_CASE)
#define MAX_BUFFER_SIZE 1024
```
