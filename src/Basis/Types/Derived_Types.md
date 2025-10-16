# 派生数据类型

## C++派生数据类型

在 C++ 中，**派生数据类型（Derived Types）** 是由基本数据类型或其他派生类型构建而来的类型。
它们扩展了语言的表达能力，使得开发者能够通过组合与抽象创建更复杂的数据结构与行为模型。

派生类型包括数组、指针、引用、函数、结构体、类、联合体以及枚举类型等。

| 数据类型 | 描述                                           | 示例                               |
| -------- | ---------------------------------------------- | ---------------------------------- |
| 数组     | 相同类型元素的集合                             | `int arr[5] = {1, 2, 3, 4, 5};`    |
| 指针     | 存储变量内存地址的类型                         | `int* ptr = &x;`                   |
| 引用     | 变量的别名                                     | `int& ref = x;`                    |
| 函数     | 函数类型，表示函数的签名                       | `int func(int a, int b);`          |
| 结构体   | 用户定义的数据类型，可以包含多个不同类型的成员 | `struct Point { int x; int y; };`  |
| 类       | 用户定义的数据类型，支持封装、继承和多态       | `class MyClass { ... };`           |
| 联合体   | 多个成员共享同一块内存                         | `union Data { int i; float f; };`  |
| 枚举     | 用户定义的整数常量集合                         | `enum Color { RED, GREEN, BLUE };` |

## 数组（Array）

数组用于存储**相同类型元素的固定长度序列**，在内存中是连续存放的。

```cpp
int numbers[5] = {1, 2, 3, 4, 5};
std::cout << numbers[2]; // 输出 3
````

* **下标从 0 开始**，越界访问会导致未定义行为（Undefined Behavior）。
* 在函数参数中，数组会**退化为指针**：

  ```cpp
  void printArray(int arr[], int size); // 等价于 void printArray(int* arr, int size);
  ```

* 若需要安全的动态数组，请使用 `std::vector`。

## 指针（Pointer）

指针是存储变量地址的变量，是 C++ 的核心特性之一。

```cpp
int a = 10;
int* p = &a;    // 指针p指向a
std::cout << *p; // 输出10
```

**关键点：**

* `*` 用于定义或解引用指针；
* `&` 用于取地址；
* 指针类型必须与目标对象类型一致；
* `nullptr` 表示空指针（C++11 引入）。

**常见错误：**

* 解引用空指针或野指针会导致崩溃；
* 动态内存需成对使用：

  ```cpp
  int* p = new int(5);
  delete p;
  ```

推荐使用智能指针（`std::unique_ptr`, `std::shared_ptr`）来避免内存泄漏。

## 引用（Reference）

引用是另一个变量的**别名**，必须在定义时初始化。

```cpp
int value = 10;
int& ref = value;
ref = 20; // value 也变为 20
```

* 引用不能为空；
* 绑定后不能再更改引用目标；
* 常用于函数参数传递以避免拷贝：

  ```cpp
  void modify(int& x) { x *= 2; }
  ```

C++11 还引入了 **右值引用 (`T&&`)**，用于支持移动语义和完美转发。

## 函数类型（Function Type）

函数本身也是一种类型，其类型由**返回值类型**和**参数类型列表**组成。

```cpp
int add(int a, int b) { return a + b; }
```

函数类型也可通过**指针或引用**使用：

```cpp
int (*funcPtr)(int, int) = add;
std::cout << funcPtr(2, 3); // 输出5
```

C++11 提供了更安全的函数封装：

* `std::function`：可存储任意可调用对象；
* `auto` 和 Lambda 表达式使函数类型推导更简洁。

## 结构体（struct）

结构体是用户自定义的复合类型，可包含多个不同类型的成员。

```cpp
struct Point {
    int x;
    int y;
};

Point p1 = {10, 20};
std::cout << p1.x << ", " << p1.y;
```

* 默认成员访问权限是 **public**；
* 可以包含成员函数；
* 可以与类（`class`）结合使用面向对象设计。

## 类（class）

类是 C++ 的核心概念之一，支持 **封装（Encapsulation）**、**继承（Inheritance）** 和 **多态（Polymorphism）**。

```cpp
class Rectangle {
private:
    int width, height;

public:
    Rectangle(int w, int h) : width(w), height(h) {}
    int area() const { return width * height; }
};
```

* 成员默认为 **private**；
* 支持构造函数、析构函数、运算符重载等；
* 是对象与抽象数据类型的基础。

> [类与对象](../CLass&Object.md)是C++语言极为重要的内容。

## 联合体（union）

联合体（`union`）允许多个成员**共用同一块内存**。

```cpp
union Data {
    int i;
    float f;
    char c;
};

Data d;
d.i = 10;
std::cout << d.i; // 输出10
d.f = 3.14;       // i 被覆盖
```

* 占用的内存大小等于最大成员的大小；
* 只能同时存储一个有效成员；
* 可用于节省内存或实现类型复用。

## 枚举类型（enum）

枚举用于定义一组具名的整数常量，提高代码可读性与类型安全。

创建枚举类型时，使用关键字 `enum`。其一般形式为：

```cpp
enum 枚举名 {
     标识符[=整型常数],
     标识符[=整型常数],
     ...
     标识符[=整型常数]
} 枚举变量;
```

例如，下面的代码定义了一个颜色枚举，变量 `c` 的类型为 `color`。最后，`c` 被赋值为 `blue`：

```cpp
enum color { red, green, blue } c;
c = blue;
```

默认情况下，第一个名称的值为 0，第二个名称的值为 1，第三个名称的值为 2，以此类推。然而，您也可以给名称赋予一个特殊的值，只需要添加一个初始值即可。例如，在下面的枚举中，`green` 的值为 5：

```cpp
enum color { red, green=5, blue };
```

在此示例中，`blue` 的值为 6，因为默认情况下，每个名称都会比它前面一个名称大 1，而 `red` 的值仍然为 0。

C++11 引入 **强类型枚举**：

  ```cpp
  enum class Status { OK, ERROR };
  Status s = Status::OK;
  ```

> 强类型枚举不会隐式转换为整数，作用域也更加安全。

## C++类型别名

类型别名可通过 `typedef` 或 `using` 定义。

| 关键字       | 描述           | 示例                   |
| --------- | ------------ | -------------------- |
| `typedef` | 为已有类型定义别名    | `typedef int MyInt;` |
| `using`   | C++11 引入的新语法 | `using MyInt = int;` |

示例：

```cpp
typedef unsigned long ulong_t;
using ushort_t = unsigned short;
```

`using` 语法更直观，且支持模板别名：

```cpp
template <typename T>
using Vec = std::vector<T>;
Vec<int> v = {1, 2, 3};
```

````admonish tip title='<i>typedef</i>和<i>define</i>有什么区别？', collapsible=true
`typedef` 和 `#define` 虽然在表面上都可以用于“简化代码书写”，但它们的本质、用途以及生效阶段存在明显区别。

(1) **用法不同：**
`typedef` 用于为已有的数据类型定义一个新的**类型别名**，从而增强程序的可读性和可维护性。例如，可以通过 `typedef unsigned int uint;` 为 `unsigned int` 定义一个更简洁的别名。
而 `#define` 是一种**宏定义指令**，主要用于定义常量或书写复杂但使用频繁的宏表达式，如 `#define PI 3.14159` 或 `#define MAX(a, b) ((a) > (b) ? (a) : (b))`。
前者作用于类型层面，后者仅进行文本替换。

(2) **执行时间不同：**
`typedef` 属于编译阶段的一部分，编译器在处理类型定义时会进行**语法与类型检查**，确保类型合法。
而 `#define` 是在编译前的**预处理阶段**执行的，它仅做**简单的字符串替换**，并不参与类型检查，因此如果使用不当，容易造成潜在的逻辑错误。

(3) **作用域不同：**
`typedef` 受 C/C++ 作用域规则限制，只在其定义的作用域内有效，例如在函数内定义的类型别名在函数外无法使用。
相对地，`#define` 不受作用域约束，只要在定义之后且未被 `#undef` 取消，其宏名在整个文件（甚至被包含的其他文件）中都有效，因此若管理不当，可能造成命名冲突。

(4) **对指针的操作不同：**
`typedef` 和 `#define` 在定义指针类型时的行为存在显著区别。
例如：
```cpp
#define PINT1 int*
typedef int* PINT2;

PINT1 a, b;  // 等价于 int* a, b; => b是int类型
PINT2 c, d;  // 等价于 int* c, *d; => c和d都是int*类型
```

这表明：`#define` 仅仅进行文本替换，而 `typedef` 定义的是一种完整的类型。使用 `typedef` 可以避免宏替换带来的歧义。

(5) **语法要求不同：**
`typedef` 是一条语句，因此必须以分号结尾；
`#define` 是预处理指令，不属于语句，不能加分号，否则分号也会被替换到目标文本中，引发错误。
````

## C++标准库类型

| 数据类型      | 描述                       | 示例                                |
| ------------- | -------------------------- | ----------------------------------- |
| `std::string` | 字符串类型                 | `std::string s = "Hello";`          |
| `std::vector` | 动态数组                   | `std::vector<int> v = {1, 2, 3};`   |
| `std::array`  | 固定大小数组（C++11 引入） | `std::array<int, 3> a = {1, 2, 3};` |
| `std::pair`   | 存储两个值的容器           | `std::pair<int, float> p(1, 2.0);`  |
| `std::map`    | 键值对容器                 | `std::map<int, std::string> m;`     |
| `std::set`    | 唯一值集合                 | `std::set<int> s = {1, 2, 3};`      |

> std::string类型的详细介绍见[字符串类型](./Character.md)章节，其他标准库类型见[STL](../../STL.md)章节。
