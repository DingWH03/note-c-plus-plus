# 函数 (Functions)

函数是 C++ 程序的基本组成部分，它们用于将程序分解为可管理、可重复使用的代码块。

## 一、函数的基本概念

### 1. 定义

函数是一段执行特定任务的命名代码块。

### 2. 优点

* **模块化 (Modularity):** 使程序结构清晰，易于理解和维护。
* **代码重用 (Code Reusability):** 一次编写，多次调用，避免重复劳动。
* **简化调试 (Easier Debugging):** 易于隔离和测试代码。

## 二、函数的构成要素

一个函数通常由以下几个部分组成：

### 1. 函数原型 (Function Prototype) / 函数声明 (Declaration)

告诉编译器函数的名称、返回类型和参数列表，但没有函数体。

**格式:**

```cpp
返回类型 函数名(参数类型 参数名, ...);
```

**示例:**

```cpp
int add(int a, int b); // 函数原型
```

* **注意:** 函数原型通常放在 `main` 函数之前或头文件 (`.h` 文件) 中。

### 2. 函数定义 (Function Definition)

包含函数头和函数体（实际执行的代码）。

**格式:**

```cpp
返回类型 函数名(参数类型 参数名, ...)
{
    // 函数体：执行任务的代码
    // ...
    return 表达式; // 如果返回类型不是 void
}
```

**示例:**

```cpp
int add(int a, int b) // 函数头
{
    // 函数体
    return a + b;
}
```

### 3. 函数调用 (Function Call)

在程序中执行函数。

**格式:**

```cpp
函数名(实参1, 实参2, ...);
```

**示例:**

```cpp
int sum = add(5, 3); // 调用 add 函数，5 和 3 是实参
```

## 三、函数的关键要素

### 1. 返回类型 (Return Type)

函数执行完毕后返回给调用者的值的类型。

* 可以是任何有效的数据类型（如 `int`, `double`, `bool`, 自定义类型等）。
* **`void`:** 如果函数不返回任何值，则使用 `void`。

### 2. 函数名 (Function Name)

用于唯一标识函数。遵循 C++ 命名规则。

### 3. 参数 (Parameters) / 形参 (Formal Parameters)

定义在函数头中，用于接收调用者传递的数据。

* **格式:** `类型 名称` (例如 `int x`)

### 4. 实参 (Arguments)

调用函数时传递给形参的实际值。

### 5. `return` 语句

用于：

* **返回值:** 将一个值返回给调用者。
* **结束执行:** 立即终止函数的执行。
* **注意:** 对于返回类型非 `void` 的函数，必须有 `return` 语句返回一个与返回类型兼容的值。

## 四、参数传递 (Argument Passing)

函数调用时，将实参的值复制或引用到形参中，主要有三种方式：

### 1. 值传递 (Pass by Value) **(最常用)**

* **机制:** 将实参的值复制给形参。
* **效果:** 函数内部对形参的任何修改，**不会** 影响到外部的实参。
* **声明:**

    ```cpp
    void func(int x); // x 是一个副本
    ```

### 2. 地址传递 (Pass by Pointer) **(C 风格，C++ 中不推荐)**

* **机制:** 传递实参的内存地址（指针）。
* **效果:** 函数可以通过指针修改实参的值。
* **声明:**

    ```cpp
    void func(int *ptr); // ptr 是一个指向 int 的指针
    ```

### 3. 引用传递 (Pass by Reference) **(C++ 推荐)**

* **机制:** 形参成为实参的别名（引用）。
* **效果:** 函数内部对形参的任何修改，**会** 直接影响到外部的实参。
* **声明:**

    ```cpp
    void func(int &ref); // ref 是实参的引用
    ```

  * **优点:** 既能修改外部变量，又避免了指针的复杂性和值传递的开销。

## 五、函数的进阶特性

### 1. 默认参数 (Default Arguments)

允许在函数声明时给参数设定一个默认值。调用时若不提供该实参，则使用默认值。

* **规则:** 默认参数必须从右向左依次设置。一旦一个参数有了默认值，它右边的所有参数都必须有默认值。

**示例:**

```cpp
void print_info(int a, int b = 10, int c = 20); // b 和 c 是默认参数

// 调用方式
print_info(1);      // a=1, b=10, c=20
print_info(1, 5);   // a=1, b=5, c=20
print_info(1, 5, 8); // a=1, b=5, c=8
```

### 2. 函数重载 (Function Overloading)

在同一作用域内，允许存在多个同名函数，但它们的 **参数列表** 必须不同（数量、类型或顺序）。

* **作用:** 使程序能够使用一个统一的名称来执行相似但操作不同数据类型的任务。

**示例:**

```cpp
int operate(int a, int b)
{ return a + b; }

double operate(double a, double b)
{ return a * b; }

int operate(int a) // 与前两个函数参数数量不同
{ return a * a; }
```

### 3. 内联函数 (Inline Functions)

使用 `inline` 关键字修饰的函数。

* **目的:** 建议编译器在函数被调用的地方直接将函数体的代码展开，而不是执行常规的函数调用机制。
* **优点:** 消除函数调用开销，提高小函数的执行效率。
* **缺点:** 可能导致代码膨胀（占用更多内存）。
* **适用场景:** 函数体非常简单（只有一两行代码）的情况。

**示例:**

```cpp
inline int square(int x) { return x * x; }
```

* **注意:** `inline` 只是对编译器的**建议**，编译器可以忽略它。

## 示例代码

```cpp
#include <iostream>

// 函数原型 (声明)
int add(int a, int b);
void swap_by_ref(int &x, int &y);

// 带有默认参数的函数原型
void greet(std::string name, std::string greeting = "Hello");

// main 函数 (程序入口)
int main() {
    int num1 = 10, num2 = 5;

    // 1. 函数调用
    int sum = add(num1, num2); // num1, num2 是实参
    std::cout << "Sum: " << sum << std::endl; // 输出: 15

    // 2. 引用传递示例
    int x = 100, y = 200;
    std::cout << "Before swap: x=" << x << ", y=" << y << std::endl;
    swap_by_ref(x, y); // 交换 x 和 y 的值
    std::cout << "After swap: x=" << x << ", y=" << y << std::endl; // 输出: x=200, y=100

    // 3. 默认参数示例
    greet("Alice");           // 使用默认问候语: Hello Alice
    greet("Bob", "Good Morning"); // 使用自定义问候语: Good Morning Bob

    return 0;
}

// 函数定义
int add(int a, int b) // a, b 是形参 (值传递)
{
    return a + b;
}

// 引用传递实现值交换
void swap_by_ref(int &x, int &y) // x, y 是引用
{
    int temp = x;
    x = y;
    y = temp;
    // x 和 y 的改变会影响到 main 函数中的实参
}

// 默认参数实现
void greet(std::string name, std::string greeting)
{
    std::cout << greeting << " " << name << std::endl;
}
```
