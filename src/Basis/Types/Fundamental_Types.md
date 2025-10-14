# 基本数据类型

## C++ 基本数据类型

| 数据类型                 | 描述                       | 大小（字节）    | 范围/取值示例                                                |
| -------------------- | ------------------------ | --------- | ------------------------------------------------------ |
| `bool`               | 布尔类型，表示真或假               | 1         | `true` 或 `false`                                       |
| `char`               | 字符类型，通常用于存储 ASCII 字符     | 1         | -128 到 127 或 0 到 255（取决于有符号或无符号）                       |
| `signed char`        | 有符号字符类型                  | 1         | -128 到 127                                             |
| `unsigned char`      | 无符号字符类型                  | 1         | 0 到 255                                                |
| `wchar_t`            | 宽字符类型，用于存储 Unicode 字符    | 2 或 4     | 取决于平台，通常 2 或 4 字节                                      |
| `char16_t`           | 16 位 Unicode 字符类型（C++11） | 2         | 0 到 65,535                                             |
| `char32_t`           | 32 位 Unicode 字符类型（C++11） | 4         | 0 到 4,294,967,295                                      |
| `short`              | 短整型                      | 2         | -32,768 到 32,767                                       |
| `unsigned short`     | 无符号短整型                   | 2         | 0 到 65,535                                             |
| `int`                | 整型                       | 4         | -2,147,483,648 到 2,147,483,647                         |
| `unsigned int`       | 无符号整型                    | 4         | 0 到 4,294,967,295                                      |
| `long`               | 长整型                      | 4 或 8     | 取决于平台                                                  |
| `unsigned long`      | 无符号长整型                   | 4 或 8     | 取决于平台                                                  |
| `long long`          | 长长整型（C++11）              | 8         | -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807 |
| `unsigned long long` | 无符号长长整型（C++11）           | 8         | 0 到 18,446,744,073,709,551,615                         |
| `float`              | 单精度浮点数                   | 4         | 约 ±3.4e±38（6-7 位有效数字）                                  |
| `double`             | 双精度浮点数                   | 8         | 约 ±1.7e±308（15 位有效数字）                                  |
| `long double`        | 扩展精度浮点数                  | 8、12 或 16 | 取决于平台                                                  |

## C++ 修饰符

| 修饰符        | 描述                   | 示例                     |
| ---------- | -------------------- | ---------------------- |
| `signed`   | 有符号类型（默认）            | `signed int x = -10;`  |
| `unsigned` | 无符号类型                | `unsigned int y = 10;` |
| `short`    | 短整型                  | `short int z = 100;`   |
| `long`     | 长整型                  | `long int a = 100000;` |
| `static` | 静态存储期，或内部链接，或类级别共享 | `static int count = 0;` |
| `const`    | 常量，值不可修改             | `const int b = 5;`     |
| `constexpr` | 编译期常量，值在编译时计算，可用于常量表达式和元编程 | `constexpr int size = 10;` |
| `volatile` | 变量可能被意外修改，禁止编译器优化    | `volatile int c = 10;` |
| `mutable`  | 类成员可以在 `const` 对象中修改 | `mutable int counter;` |
| `extern`   | 声明一个在其他源文件中定义的变量或函数，用于跨文件共享全局符号      | `extern int global_var;`    |
| `register` | 建议编译器将变量存放在 CPU 寄存器中以提高访问速度（现代编译器多已自动优化） | `register int counter = 0;` |

## C++11 新增数据类型

| 数据类型                    | 描述                | 示例                                             |
| ----------------------- | ----------------- | ---------------------------------------------- |
| `auto`                  | 自动类型推断            | `auto x = 10;`                                 |
| `decltype`              | 获取表达式的类型          | `decltype(x) y = 20;`                          |
| `nullptr`               | 空指针常量             | `int* ptr = nullptr;`                          |
| `std::initializer_list` | 初始化列表类型           | `std::initializer_list<int> list = {1, 2, 3};` |
| `std::tuple`            | 元组类型，可以存储多个不同类型的值 | `std::tuple<int, float, char> t(1, 2.0, 'a');` |

## 说明

### **宽字符类型 (`wchar_t`)**

`wchar_t` 是 C++ 中用于存储宽字符的类型，广泛应用于需要处理 Unicode 字符集的程序中。与普通的 `char` 类型（通常用于存储 ASCII 字符）不同，`wchar_t` 的设计目的是为了支持更大的字符集，特别是 Unicode。由于 `wchar_t` 需要存储更多的字符信息，因此其大小取决于平台，通常在 2 或 4 字节之间。在一些平台上，`wchar_t` 被定义为 2 字节（16 位），在其他平台上则可能是 4 字节（32 位）。使用 `wchar_t` 可以轻松处理如中文、日文等非拉丁字符。

```cpp
#include <iostream>

int main() {
    wchar_t wide_char = L'我';  // 使用 wchar_t 存储一个 Unicode 字符
    std::wcout << wide_char << std::endl;  // 输出：我
    return 0;
}
```

### **`char16_t` 和 `char32_t`**

`char16_t` 和 `char32_t` 是 C++11 引入的专门用于存储 Unicode 字符的类型，分别表示 16 位和 32 位字符类型。`char16_t` 是为了支持 UTF-16 编码而设计的，而 `char32_t` 是为了支持 UTF-32 编码。`char16_t` 用 2 字节来存储一个字符，而 `char32_t` 用 4 字节存储一个字符。这两种类型能够直接表示 Unicode 字符，而无需进行额外的编码转换。

`char16_t` 和 `char32_t` 提供了对更广泛字符集的支持，尤其适合那些需要处理全球化文本的应用程序。`char16_t` 和 `char32_t` 作为 Unicode 字符的表示方式，分别与 UTF-16 和 UTF-32 编码兼容，能够表示包括基本多语言平面（BMP）以及更高平面字符在内的所有 Unicode 字符。

```cpp
#include <iostream>

int main() {
    char16_t char16 = u'你';  // 使用 char16_t 存储一个 UTF-16 编码的字符
    char32_t char32 = U'你';  // 使用 char32_t 存储一个 UTF-32 编码的字符

    std::wcout << "char16_t: " << char16 << std::endl;
    std::wcout << "char32_t: " << char32 << std::endl;

    return 0;
}
```

### `volatile`

在现代 CPU 中通常包含多个核心，每个核心都有独立的缓存。多个核心可能同时缓存了同一段主存的数据。一般情况下，缓存和主存的数据是一致的，但在多线程并发场景下，由于缓存写回策略的影响，数据的修改可能无法及时同步到主存，从而导致数据不一致的问题。

`volatile` 关键字用于告诉编译器：某个变量的值可能随时被外部因素（如其他线程、硬件设备或中断）修改，因此缓存中的值并不可靠。出于这个原因，编译器在访问该变量时不会进行过度优化，而是强制每次都从内存中读取最新的值。

1. 可见性
    `volatile` 保证变量的值在不同线程或不同硬件环境下始终是“最新可见”的。即使某个核心或寄存器中有缓存数据，访问 `volatile` 变量时也必须直接从内存或硬件中获取，而不是使用缓存副本。

2. 不可优化
    在编译阶段，为了提高执行效率，编译器会进行多种优化。例如：

    ```cpp
    int flag = 0;
    while (flag == 0) {
        // 等待 flag 改变
    }
    ```

    若 `flag` 未被声明为 `volatile`，编译器可能认为 `flag` 始终等于 0，于是直接将循环优化为 `while(false)`，导致死循环。

    使用 `volatile` 可以禁止类似的优化，包括 **消除优化**、**传播优化** 和 **合并优化**，从而保证变量的读写行为不会被错误简化。

3. 顺序性
    `volatile` 还会在一定程度上影响指令的顺序。编译器在处理 `volatile` 变量时，会保证读写操作不会因乱序优化而颠倒，从而维持必要的执行顺序。

`volatile` 在嵌入式编程和多线程编程中尤为重要：

* **多线程共享变量**：确保不同线程读取到的值保持一致。
* **中断处理**：中断可能随时修改某个变量，主程序通过 `volatile` 保证能正确检测到变化。
* **硬件寄存器访问**：在嵌入式系统中，硬件寄存器的值可能在后台自动更新，必须通过 `volatile` 确保每次访问都直接读取硬件寄存器的当前值。

尽管 `volatile` 在保证可见性、防止编译器优化、维持一定顺序性方面很有用，但它并不是并发编程的“万能钥匙”，主要局限性如下：

1. **不保证原子性**

   * `volatile` 仅保证读写操作不会被优化和缓存，但不能保证复合操作的原子性。
   * 例如：

     ```cpp
     volatile int counter = 0;
     counter++; // 实际分解为：读取 -> 修改 -> 写回
     ```

     在多线程环境下可能发生竞态条件，导致结果错误。

2. **不等同于内存屏障（Memory Barrier）**

   * `volatile` 的“顺序性”只作用于编译器层面，防止指令在编译时被重排。
   * 但在 CPU 的指令执行层面，仍然可能发生硬件乱序执行。若需要在多线程同步中严格保证内存访问顺序，还需要使用更强的同步原语（如 C++ 中的 `std::atomic` 或内存屏障指令）。

3. **性能开销**

   * 每次访问 `volatile` 变量都要从内存中读取最新值，无法使用寄存器缓存，可能造成一定性能损失。

4. **局限于特定场景**

   * `volatile` 更适合用于：

     * 标志位（如中断标志、任务完成标志）。
     * 硬件寄存器访问。
   * 但在复杂的多线程共享数据同步场景下，仅依赖 `volatile` 是不够的，往往需要互斥锁、原子操作或更高级的同步机制。

### `mutable`

`mutable` 关键字是用来修饰类的成员变量的，意味着即使该对象是常量（`const`），这些成员变量也可以被修改。通常，`mutable` 用于那些希望在 `const` 方法中进行修改的成员变量，比如用于缓存的成员变量。通过 `mutable`，我们可以在 `const` 方法中修改这些成员，而不会破坏 `const` 对象的常量性。

以下示例展示了在 `const` 方法中修改 `mutable` 成员变量的情况：

```cpp
class MyClass {
public:
    mutable int cache;  // 使用 mutable 修饰的成员变量

    MyClass() : cache(0) {}

    void updateCache() const {
        // 即使 updateCache 是 const 方法，cache 依然可以被修改
        cache++;
    }
};

int main() {
    const MyClass obj;
    obj.updateCache();  // 可在 const 对象上调用
    std::cout << obj.cache << std::endl;  // 输出：1
    return 0;
}
```

### `decltype`

`decltype` 是 C++11 引入的一个关键字，用于获取表达式的类型，而不需要显式地声明变量的类型。它通常用于模板编程中，或者当我们不确定某个表达式的类型时。`decltype` 可以非常方便地获取复杂类型，尤其是当类型通过复杂的表达式推导出来时。

例如，以下代码通过 `decltype` 获取了变量 `x` 的类型，并且通过 `auto` 使得代码更加简洁：

```cpp
int x = 5;
decltype(x) y = 10;  // y 的类型是 int，因为 x 是 int

auto z = x + y;  // z 的类型由编译器推断，类型为 int
```

### `std::initializer_list`

`std::initializer_list` 是 C++11 引入的一个模板类，用于支持初始化列表。它允许在创建对象时通过花括号 `{}` 来传递多个值。`initializer_list` 主要用于支持类的构造函数接收不定数量的参数，或者将多个值传递给函数，特别适用于那些需要接收多个初始值的容器类型。

```cpp
#include <initializer_list>
#include <iostream>

void printList(std::initializer_list<int> list) {
    for (auto i : list) {
        std::cout << i << " ";
    }
    std::cout << std::endl;
}

int main() {
    printList({1, 2, 3, 4, 5});  // 使用 initializer_list
    return 0;
}
```

### `std::tuple`

`std::tuple` 是 C++11 引入的一个模板类，允许存储多个不同类型的元素。与数组和 `std::vector` 不同，`tuple` 允许每个元素拥有不同的类型。`std::tuple` 是一个非常强大的工具，可以将多个不同类型的值打包在一起，并在需要时访问这些值。它常用于函数返回多个不同类型的值，或在需要将多种类型的参数组合起来时使用。

```cpp
#include <tuple>
#include <iostream>

int main() {
    std::tuple<int, double, char> t(1, 3.14, 'A');

    std::cout << std::get<0>(t) << ", ";  // 1
    std::cout << std::get<1>(t) << ", ";  // 3.14
    std::cout << std::get<2>(t) << std::endl;  // A

    // 修改 tuple 的元素
    std::get<0>(t) = 42;
    std::cout << std::get<0>(t) << std::endl;  // 42

    return 0;
}
```

### `static`

`static` 关键字在 C++ 中有三种主要用途，取决于它所修饰的对象和作用域，而在C语言中由于不支持类从而只支持修饰**局部静态变量**和**外部静态变量、函数**。

#### 1\. 局部变量（函数内）- 改变存储期

当 `static` 用于函数内的局部变量时，它改变了变量的**存储期**（Storage Duration）。

* **存储期**：`static` 局部变量在程序运行期间**只会被初始化一次**，且生命周期与整个程序相同（静态存储期），但其**作用域**仍限定在定义它的函数内部。
* **用途**：用于记录函数被调用的次数，或者在多次调用中保持某个状态。

<!-- end list -->

```cpp
void func() {
    static int count = 0; // 只在程序启动时初始化一次
    count++;
    std::cout << "Count: " << count << std::endl;
}
// 每次调用 func()，count 都会递增，而不是重置为 0
```

#### 2\. 全局变量和函数（文件作用域）- 改变链接性

当 `static` 用于全局变量或普通函数时，它改变了它们的**链接性**（Linkage）。

* **链接性**：将默认的**外部链接**（External Linkage，可以在其他源文件访问）改为**内部链接**（Internal Linkage）。
* **用途**：使变量或函数只在其定义的\*\*当前翻译单元（源文件）\*\*中可见和可用，避免与其他源文件中的同名标识符发生冲突。

<!-- end list -->

```cpp
// file1.cpp
static int global_data = 10; // 只能在 file1.cpp 中访问
static void helper_func() {  // 只能在 file1.cpp 中调用
    // ...
}
```

#### 3\. 类成员（成员变量和成员函数）- 类级别共享

当 `static` 用于类内部的成员时，它使成员成为**类级别**的共享资源，而不是每个对象独有的资源。

* **静态成员变量**：
  * 该变量为**所有**类的对象所共享，**只存在一个副本**。
  * 它必须在**类外部**进行定义和初始化（除非是 `const static` 整数类型）。
  * 可以通过类名或对象访问。
* **静态成员函数**：
  * 它不依赖于任何特定的类对象。
  * 它**不能**直接访问非静态的成员变量或成员函数（因为它没有 `this` 指针）。
  * 通常用于访问和操作静态成员变量，或作为工具函数。

<!-- end list -->

```cpp
class MyClass {
public:
    static int object_count; // 静态成员变量声明

    MyClass() {
        object_count++;
    }

    static int get_count() { // 静态成员函数
        return object_count;
    }
};

// 在类外定义和初始化静态成员
int MyClass::object_count = 0;

int main() {
    MyClass obj1;
    MyClass obj2;
    // 使用类名直接访问静态成员
    std::cout << MyClass::get_count() << std::endl; // 输出：2
    return 0;
}
```

### `auto`

`auto` 是 C++11 引入的一个关键字，允许编译器自动推导出变量的类型。`auto` 使得代码更加简洁，并且减少了显式指定类型的需求，尤其在处理复杂类型时非常有用。`auto` 通常用于变量声明时，让编译器根据赋值的表达式自动推断类型，这在迭代器和模板编程中尤其常见。

```cpp
int main() {
    auto x = 10;  // x 的类型是 int
    auto y = 3.14;  // y 的类型是 double

    std::vector<int> vec = {1, 2, 3, 4, 5};
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

### `const`

在 C 语言和 C++ 中，`const` 都用于限定变量为“只读”。
但是 C++ 对 `const` 的支持更为强大和灵活，它不仅影响编译器的检查机制，还会在类型系统中起作用。

1. 基本特性

   * 在 C 中，`const` 修饰的变量默认是只读存储（readonly），但本质上仍然是变量，而不是常量。

     ```c
     const int x = 10;
     int *p = (int*)&x; // 通过强制转换依然能修改
     *p = 20;           // UB（未定义行为）
     ```

   * 在 C++ 中，`const` 更严格，编译器会将其视为类型的一部分。

     ```cpp
     const int x = 10;
     x = 20;  // 编译错误，禁止修改
     ```

2. 修饰位置

   `const` 可以修饰不同对象，表达不同含义：

   * 修饰变量：值不可修改。
   * 修饰指针：

     ```cpp
     const int *p;  // 指向常量的指针（*p 不可改，p 可改）
     int *const p;  // 常量指针（p 不可改，*p 可改）
     const int *const p; // 指向常量的常量指针
     ```

   * 修饰函数参数：保证函数体内不会修改该参数。
   * 修饰成员函数：表示该成员函数不会修改对象的成员变量，本质上是修饰**this指针**。

3. 作用域与链接

   * 在 C 中，`const` 全局变量默认是 **外部链接**，除非显式加 `static`使得在本文件可见。
   * 在 C++ 中，`const` 全局变量默认是 **内部链接**（只在本翻译单元内可见），若要在多个文件共享，需加 `extern`。

4. 局限性

   * `const` 并不保证编译期求值，它仅仅保证“运行时不能被修改”。
   * 如果需要编译期常量（如数组大小、模板参数），在 C++11 之前通常使用 `#define` 或 `enum hack`。

### `constexpr`

`constexpr` 是 C++11 引入的关键字，用于声明“编译期常量表达式”。它不仅意味着值不可变，更重要的是：
编译器必须在编译期对其进行求值（只要表达式满足常量表达式要求）。

1. 基本特性

   * `constexpr` 变量一定是常量，并且能在编译期被计算：

     ```cpp
     constexpr int size = 10;
     int arr[size]; // 合法
     ```

   * 与 `const` 不同，`constexpr` 要求初始化表达式必须是编译期可计算的常量。

2. 函数支持

   * `constexpr` 还可以修饰函数：

     ```cpp
     constexpr int square(int x) {
         return x * x;
     }
     int arr[square(5)]; // 在编译期计算为 25
     ```

   * 这样的函数可以在编译期使用，也可以在运行时调用（若传入非常量参数）。

3. 类与构造函数

   * C++11 起，`constexpr` 可以修饰构造函数，表示该类的对象可以在编译期生成常量。
   * C++14/17 对 `constexpr` 的限制逐步放宽，例如允许有分支、循环，更接近普通函数。

4. 区别于 `const`

   * `const`：运行期常量（只读），初始化表达式可以是运行时值，可以使用`const_cast`去除限定。
   * `constexpr`：编译期常量，初始化表达式必须在编译期可求值。

   对比示例：

   ```cpp
   const int a = std::time(nullptr); // 合法，运行期决定
   constexpr int b = std::time(nullptr); // 错误，不能在编译期求值
   ```

### `extern`

`extern` 关键字用于**声明**而非定义变量或函数。
它通常出现在多文件项目中，用于在一个文件中访问另一个文件定义的全局变量或函数。

* 作用：

  * 告诉编译器“该变量或函数在别处定义”；
  * 不会为其分配存储空间（除非在定义处）；
  * 避免重复定义全局符号。

```cpp
// file1.cpp
int count = 10;  // 定义全局变量

// file2.cpp
#include <iostream>
extern int count;  // 声明外部变量（非定义）
int main() {
    std::cout << count << std::endl;  // 输出 10
    return 0;
}
```

* 在C++中：

  C++ 默认情况下，`const` 全局变量具有内部链接（`internal linkage`），也就是仅在本文件内可见。
  若希望跨文件共享一个常量变量，必须结合 `extern` 使用：

  ```cpp
  // file1.cpp
  extern const int BUFFER_SIZE = 1024;

  // file2.cpp
  extern const int BUFFER_SIZE;  // 声明外部常量
  ```

* 与函数结合使用：

  对于函数来说，`extern` 是默认属性，即所有非 `static` 函数都具有外部链接性，因此通常可省略：

  ```cpp
  extern void foo();  // 与 void foo(); 等价
  ```

### `register`

`register` 是早期 C/C++ 时代用于**提升变量访问速度**的关键字，提示编译器将变量存储在 CPU 寄存器中，而非内存中。

* 特点：

  * 变量可能被存放在 CPU 寄存器中，而非内存；
  * 不能对 `register` 变量使用取地址操作符 `&`；
  * 仅能修饰局部变量或函数参数；
  * 在现代编译器中通常**被自动优化机制取代**，因此多用于教学或历史理解。

```cpp
#include <iostream>

int main() {
    register int i;  // 建议编译器将 i 放入寄存器中
    for (i = 0; i < 5; ++i) {
        std::cout << i << " ";
    }
    std::cout << std::endl;
    return 0;
}
```

* **局限性：**

  * 编译器不保证一定会将其放入寄存器；
  * 不能取地址（即 `&i` 是非法的）；
  * 在现代 C++ 中几乎没有实际性能提升，优化器会自动选择合适的寄存器分配策略。
