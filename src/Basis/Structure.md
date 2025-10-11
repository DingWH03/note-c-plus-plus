# 控制结构（Control Structures）

在 C++ 中，控制结构用于控制程序语句的执行顺序。C++ 的控制流主要分为三种类型：

1. **顺序结构**：从上到下依次执行语句，是程序的默认执行方式。
2. **选择结构**：根据条件判断，执行不同的分支语句。
3. **循环结构**：重复执行一段代码，直到某个条件满足为止。

在理解控制结构时，最关键的是掌握**表达式求值时机**、**作用域规则**与**循环/分支退出机制**。

## 一、循环结构（Loop Structure）

循环结构的本质是**在某个条件满足时重复执行代码块**。
C++ 提供了三种主要循环语法：`for`、`while` 和 `do...while`。

虽然三者都能实现循环逻辑，但它们在**执行顺序**与**条件判断时机**上存在显著差异。

### 1. `for` 循环

```cpp
for (初始化表达式; 条件表达式; 迭代表达式) {
    // 循环体
}
```

在执行时，`for` 循环遵循如下过程：

1. **初始化表达式**在循环开始时执行一次；
2. 计算**条件表达式**，若为 `true`，则进入循环体；
3. 执行循环体中的语句；
4. 执行**迭代表达式**（通常为自增或自减操作）；
5. 再次判断条件表达式；
6. 若条件仍为真，继续循环；否则退出循环。

```cpp
#include <iostream>
using namespace std;

int main() {
    for (int i = 0; i < 5; ++i) {
        cout << "当前 i = " << i << endl;
    }
}
```

程序执行顺序如下：

```text
初始化：i = 0
判断条件：i < 5 → true
执行循环体：输出 i
迭代表达式：i++
再判断...
```

* **局部作用域**：在 `for` 循环中声明的变量（如 `int i`）仅在循环内部可见，循环结束后变量被销毁。

* **可省略项**：

  ```cpp
  for (;;) {
      // 无限循环
  }
  ```

  如果条件表达式被省略，默认为 `true`。

* **循环控制建议**：

  * 在循环内部修改循环变量容易出错；
  * 条件表达式应当保证能在有限次迭代后终止，否则会造成死循环。

C++11 引入了**基于范围的 for 循环**（range-based for loop）：

```cpp
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> nums = {1, 2, 3, 4, 5};
    for (int x : nums) {
        cout << x << " ";
    }
}
```

等价于传统形式：

```cpp
for (auto it = nums.begin(); it != nums.end(); ++it)
    cout << *it << " ";
```

这种写法更简洁、安全，也更符合 STL 容器的使用习惯。

### 2. `while` 循环

```cpp
while (条件表达式) {
    // 循环体
}
```

`while` 循环先判断条件，再执行循环体。执行流程如下：

1. 判断条件；
2. 若条件为 `true`，执行循环体；
3. 循环体执行完毕后，再次判断条件；
4. 若条件为 `false`，循环终止。

```cpp
int i = 0;
while (i < 5) {
    cout << "i = " << i << endl;
    ++i;
}
```

与 `for` 相比，`while` 更适用于**循环次数不确定但终止条件明确**的情况，例如文件读取或网络接收：

```cpp
string line;
while (getline(cin, line)) {
    cout << line << endl;
}
```

这种结构常见于 IO 流循环，因为输入次数事先未知。

一个经典的初学者错误：

```cpp
int i = 0;
while (i < 10);
{
    cout << i << endl;
    ++i;
}
```

这里的分号 `;` 结束了 `while` 语句，导致循环体为空。
实际运行时这段代码会陷入死循环，因为条件判断永远为真而循环体无法修改变量。

### 3. `do...while` 循环

```cpp
do {
    // 循环体
} while (条件表达式);
```

与 `while` 不同，`do...while` **先执行循环体一次，再判断条件**。
因此，无论条件是否为真，循环体至少会被执行一次。

```cpp
int n;
do {
    cout << "请输入一个正数：";
    cin >> n;
} while (n <= 0);
```

程序会反复要求输入，直到输入值大于零。
这种结构常用于需要**先执行一次再决定是否继续**的交互逻辑。

### 4. 循环控制语句

* `break` —— 提前结束循环

    `break` 用于**立即终止当前循环**或 `switch` 语句，不会再执行后续的循环体和条件判断。

    ```cpp
    for (int i = 0; i < 10; ++i) {
        if (i == 5)
            break;
        cout << i << " ";
    }
    ```

* `continue` —— 跳过当前迭代

    `continue` 会**跳过本次循环剩余部分**，直接进入下一次条件判断。

    ```cpp
    for (int i = 0; i < 10; ++i) {
        if (i % 2 == 0)
            continue; // 跳过偶数
        cout << i << " ";
    }
    ```

* `goto` —— 无条件跳转（极少使用）

    ```cpp
    int i = 0;
    loop_start:
    if (i < 5) {
        cout << i << endl;
        ++i;
        goto loop_start;
    }
    ```

    虽然语法合法，但这种写法不符合结构化编程思想，现代 C++ 中几乎不使用。

### 5. 嵌套循环与性能考量

循环可以嵌套，例如输出九九乘法表：

```cpp
for (int i = 1; i <= 9; ++i) {
    for (int j = 1; j <= i; ++j) {
        cout << j << "x" << i << "=" << i * j << "\t";
    }
    cout << endl;
}
```

嵌套循环会带来**O(n²)** 级别的时间复杂度，因此在数据量大时应当考虑优化：

* 减少内层循环的迭代次数；
* 将不必要的计算提前至循环外；
* 使用 `break` 或条件提前退出。

## 二、判断结构（Selection Structure）

判断结构的目的是让程序**在不同条件下执行不同语句**。
C++ 提供了两种主要形式：`if` 与 `switch`，另有三目运算符用于简单条件判断。

### 1. `if` 条件语句

```cpp
if (条件表达式) {
    // 条件成立时执行
}
else {
    // 条件不成立时执行
}
```

在执行时，编译器首先计算条件表达式的布尔值：

* 若为 `true`，执行 `if` 块；
* 否则执行 `else` 块（如果存在）。

除此以外，`if`还支持多分支判断：

```cpp
if (score >= 90)
    cout << "优秀";
else if (score >= 80)
    cout << "良好";
else if (score >= 60)
    cout << "及格";
else
    cout << "不及格";
```

编译器从上至下依次判断条件表达式。
当某个条件为真时，后续分支将被跳过。

* 短路求值（short-circuit evaluation）

逻辑运算符 `&&` 和 `||` 在判断中具有**短路行为**：

```cpp
if (x != 0 && 10 / x > 1)  // 安全
```

若 `x == 0`，第一个条件为 `false`，第二个表达式不会被求值，从而避免除零错误。
短路机制是 C++ 逻辑判断中非常重要的性能与安全特性。

* 悬挂 else（dangling else）问题

```cpp
if (a > 0)
    if (b > 0)
        cout << "A";
    else
        cout << "B";
```

此时 `else` 默认匹配**最近的未匹配 if**，即第二个 `if`。
建议始终使用大括号 `{}` 明确逻辑边界。

### 2. `switch` 语句

```cpp
switch (表达式) {
    case 常量1:
        // 语句
        break;
    case 常量2:
        // 语句
        break;
    default:
        // 默认语句
        break;
}
```

`switch` 表达式的值会与每个 `case` 标签进行匹配，若相等，则从该处开始执行，直到遇到 `break` 或 `switch` 结束。

```cpp
int day = 3;
switch (day) {
    case 1: cout << "Monday"; break;
    case 2: cout << "Tuesday"; break;
    case 3: cout << "Wednesday"; break;
    default: cout << "Invalid";
}
```

* 语义分析与注意事项

* `switch` 的判断值必须是**整数类型或枚举类型**；
* `case` 标签必须是**常量表达式**；
* 若 `break` 省略，将导致**贯穿 (fall through)**，继续执行后续分支；
* `default` 分支可选，但建议始终保留。

* 现代 C++ 的枚举与 `switch`

```cpp
enum class Color { Red, Green, Blue };

Color c = Color::Green;
switch (c) {
    case Color::Red: cout << "Red"; break;
    case Color::Green: cout << "Green"; break;
    case Color::Blue: cout << "Blue"; break;
}
```

使用 `enum class` 可避免命名冲突和隐式转换，提高类型安全性。

### 3. 条件运算符（三目运算符）

C++ 提供了简洁的条件表达式：

```cpp
表达式1 ? 表达式2 : 表达式3;
```

求值逻辑：

* 若表达式1为真，则整个表达式结果为表达式2；
* 否则结果为表达式3。

示例：

```cpp
int a = 10, b = 20;
int max = (a > b) ? a : b;
```

该语句的效果等价于：

```cpp
if (a > b) max = a;
else max = b;
```

三目运算符常用于简单赋值，但不适合复杂逻辑，因为可读性较差。
