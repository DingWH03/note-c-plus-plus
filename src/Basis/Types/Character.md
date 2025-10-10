# C++ 字符串类型

C++ 提供了两种主要的字符串表示方式：C 风格字符串和 C++ 引入的 `string` 类类型。尽管 C++ 中 `string` 类型在许多情况下更为常用，但 C 风格字符串仍然是 C++ 中的一个重要组成部分，特别是在需要与旧有的 C 库代码兼容时。

## C 风格字符串

C 风格的字符串源自 C 语言，并在 C++ 中继续得到支持。这种字符串表示方式实际上是一个以 null 字符 `\0` 结尾的字符数组。C 风格字符串的本质是一个一维字符数组，末尾的 null 字符标识了字符串的结束。

### C 风格字符串的定义与初始化

您可以通过以下方式定义并初始化一个 C 风格字符串：

```cpp
char site[6] = {'H', 'E', 'L', 'L', 'O', '\0'};
```

也可以使用更简便的方式进行初始化：

```cpp
char site[] = "HELLO";
```

在这种情况下，C++ 编译器会自动在字符串的末尾添加 null 字符 `\0`，无需显式地写出。

```cpp
#include <iostream>

using namespace std;

int main() {
   char s[] = "HELLO"; // 字符串自动以 '\0' 结束

   cout << s << endl;

   return 0;
}
```

输出：

```text
HELLO
```

### C 风格字符串的常用操作函数

C 风格字符串有许多函数可以进行常见的操作。以下是常用的几个字符串操作函数：

| 函数             | 目的                                                         |
| ---------------- | ------------------------------------------------------------ |
| `strcpy(s1, s2)` | 复制字符串 `s2` 到字符串 `s1`。                              |
| `strcat(s1, s2)` | 将字符串 `s2` 连接到字符串 `s1` 的末尾。                     |
| `strlen(s1)`     | 返回字符串 `s1` 的长度（不包括 `\0`）。                      |
| `strcmp(s1, s2)` | 比较两个字符串 `s1` 和 `s2`，返回值为 0 时相同，负值表示 `s1 < s2`，正值表示 `s1 > s2`。 |
| `strchr(s1, ch)` | 返回指针，指向字符串 `s1` 中字符 `ch` 的首次出现位置。       |
| `strstr(s1, s2)` | 返回指针，指向字符串 `s1` 中子字符串 `s2` 的首次出现位置。   |

```cpp
#include <iostream>
#include <cstring>

using namespace std;

int main() {
   char str1[13] = "hello";
   char str2[13] = "world";
   char str3[13];
   int len;

   // 复制 str1 到 str3
   strcpy(str3, str1);
   cout << "strcpy(str3, str1): " << str3 << endl;

   // 连接 str1 和 str2
   strcat(str1, str2);
   cout << "strcat(str1, str2): " << str1 << endl;

   // 计算连接后的长度
   len = strlen(str1);
   cout << "strlen(str1): " << len << endl;

   return 0;
}
```

输出：

```text
strcpy(str3, str1): hello
strcat(str1, str2): helloworld
strlen(str1): 10
```

## C++ 中的 `string` 类

C++ 标准库提供了 `string` 类，它是 C 风格字符串的现代替代品。`string` 类提供了丰富的成员函数，支持字符串的动态操作，简化了字符串处理过程，并且避免了 C 风格字符串的许多潜在问题。

### `string` 类基本操作

C++ 中的 `string` 类提供了方便的字符串处理方法，使得字符串操作变得更加高效和直观。以下是一些常见的操作。

```cpp
#include <iostream>
#include <string>

using namespace std;

int main() {
   string str1 = "hello";
   string str2 = "world";
   string str3;
   int len;

   // 复制 str1 到 str3
   str3 = str1;
   cout << "str3: " << str3 << endl;

   // 连接 str1 和 str2
   str3 = str1 + str2;
   cout << "str1 + str2: " << str3 << endl;

   // 计算连接后的总长度
   len = str3.size();
   cout << "str3.size(): " << len << endl;

   return 0;
}
```

输出：

```text
str3: hello
str1 + str2: helloworld
str3.size(): 10
```

### `string` 类的常用成员函数

`string` 类提供了多种函数来执行常见的字符串操作，常见的成员函数包括：

- `append(str)`：将 `str` 追加到当前字符串末尾。
- `insert(pos, str)`：在位置 `pos` 处插入字符串 `str`。
- `erase(pos, len)`：从位置 `pos` 开始，删除 `len` 个字符。
- `substr(pos, len)`：返回从位置 `pos` 开始的长度为 `len` 的子字符串。
- `find(str)`：返回子字符串 `str` 在当前字符串中首次出现的位置。
- `replace(pos, len, str)`：将当前位置 `pos` 开始的 `len` 个字符替换为字符串 `str`。

```cpp
#include <iostream>
#include <string>

using namespace std;

int main() {
    string str = "Hello, world!";

    // 获取子字符串
    string sub = str.substr(7, 5);  // 从位置 7 开始，获取 5 个字符
    cout << "Substring: " << sub << endl;

    // 查找字符 'w' 在字符串中的位置
    size_t pos = str.find('w');
    cout << "'w' found at: " << pos << endl;

    // 替换子字符串
    str.replace(7, 5, "C++");
    cout << "After replace: " << str << endl;

    return 0;
}
```

输出：

```text
Substring: world
'w' found at: 7
After replace: Hello, C++!
```
