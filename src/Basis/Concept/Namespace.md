# 命名空间（Namespace）

## 一、概念与作用

* 定义：命名空间是**声明性区域**，它提供了一种将代码组织成**逻辑组**并**避免命名冲突**的机制。
* 作用：
  * 将类型、函数、变量等**标识符**包含在一个有名称的范围（Scope）内。
  * 在大型程序中，尤其当程序包含多个库时，有效解决**名称冲突**问题（例如，不同库中可能存在同名的函数或变量）。

## 二、命名空间声明

使用 `namespace` 关键字来声明一个命名空间。

```cpp
// 文件: MyCode.h

namespace MyNamespace {
    int value;

    void FunctionA() {
        // ...
    }

    class MyClass {
        // ...
    };
}
```

## 三、访问命名空间成员

要访问命名空间内的成员，需要使用**范围解析运算符** `::`。

### 方式一：完整限定名称（Fully Qualified Name）

在命名空间外部使用完整的名称来限定成员。

```cpp
// 访问 MyNamespace 中的 value 和 FunctionA
MyNamespace::value = 10;
MyNamespace::FunctionA();
```

### 方式二：`using` 声明（`using` Declaration）

将命名空间中的**特定名称**引入当前作用域。

```cpp
#include "MyCode.h"

// 引入 MyNamespace 中的 FunctionA
using MyNamespace::FunctionA;

void SomeOtherFunction() {
    // 此时可以直接使用 FunctionA，而不需要 MyNamespace::
    FunctionA();

    // 访问 value 仍需限定
    MyNamespace::value = 20;
}
```

### 方式三：`using` 指示词（`using` Directive）

将命名空间中的**所有名称**引入当前作用域。

```cpp
#include "MyCode.h"

// 将 MyNamespace 中的所有名称引入
using namespace MyNamespace;

void AnotherFunction() {
    // 此时可以直接使用所有成员
    value = 30;
    FunctionA();
}
```

> **最佳实践**：
>
> 1. **避免**在**头文件**（`.h` 或 `.hpp`）中使用 `using namespace` 指示词，因为这会将命名空间中的所有名称带入所有包含该头文件的文件中，可能导致难以调试的名称冲突或隐藏问题。
> 2. 在**实现文件**（`.cpp`）的顶部或函数内部，可以使用 `using namespace` 指示词以简化代码。
> 3. 如果只使用一两个标识符，建议使用**完整限定名称**或`using` 声明。

## 四、命名空间的特性

### 1. 分散定义（Separated Definition）

命名空间可以在单个文件中的**多个块**中声明，也可以在**多个文件**中声明。编译器会将所有部分**连接**起来。

```cpp
// 文件 A.h
namespace Utils {
    void func1();
}

// 文件 B.h
namespace Utils {
    void func2(); // 即使在不同文件，也属于同一个 Utils 命名空间
}
```

### 2. 全局命名空间（Global Namespace）

* 未在任何显式命名空间中声明的标识符属于**隐式全局命名空间**。
* 要显式访问全局命名空间中的标识符，可以使用**没有名称**的范围解析运算符 `::`：

    ```cpp
    int globalVar = 100;

    namespace Local {
        int globalVar = 200; // 隐藏全局变量

        void test() {
            int val1 = globalVar; // 使用 Local::globalVar (200)
            int val2 = ::globalVar; // 使用 全局命名空间 的 globalVar (100)
        }
    }
    ```

### 3. 嵌套命名空间（Nested Namespaces）

命名空间可以嵌套，C++17 引入了**内联嵌套**简化语法。

```cpp
namespace Outer {
    namespace Inner {
        void innerFunc();
    } // 结束 Inner
} // 结束 Outer

// 访问方式：
Outer::Inner::innerFunc();

// C++17 简化写法：
namespace Outer::Inner {
    void anotherInnerFunc();
}
```

### 4. 匿名或未命名命名空间（Anonymous/Unnamed Namespaces）

声明一个没有名称的命名空间。

```cpp
namespace {
    int MyFunc() { return 0; }
    int internalVar;
}
```

* **作用**：未命名命名空间中的变量和函数**只在当前编译单元（文件）内可见**，它们具有**内部链接（internal linkage）**，等同于在全局作用域中使用 `static` 关键字。
* **用途**：避免在其他文件中看到变量声明，替代老式的 `static` 全局/函数定义。

### 5. 命名空间别名（Namespace Alias）

用于为过长的命名空间名称创建别名，以提高代码可读性和输入效率。

```cpp
namespace VeryLongNamespaceName {
    void func();
}

// 创建别名
namespace VLNN = VeryLongNamespaceName;

// 使用别名访问
VLNN::func();
```
