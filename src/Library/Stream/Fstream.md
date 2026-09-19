# 文件流（fstream）

`<fstream>` 把文件抽象成流，让文件读写和 `cin` / `cout` 用同一套 `<<` / `>>` 接口。这样从控制台改成从文件读，往往只需要换掉流对象的类型。

三个核心类分工明确：

| 类 | 方向 | 说明 |
| :--- | :--- | :--- |
| `std::ifstream` | 输入 | 从文件读取 |
| `std::ofstream` | 输出 | 写入文件 |
| `std::fstream` | 双向 | 既可读也可写 |

它们都继承自 `iostream` 体系，因此 `getline`、`seekg`、`tellp` 这些通用接口都能用。

## 一、打开文件

### 1. 构造时打开

```c++
#include <fstream>

std::ifstream in("input.txt");          // 默认以文本模式读取
std::ofstream out("output.txt");        // 默认截断文件（内容清空）
std::fstream io("data.bin", std::ios::in | std::ios::out | std::ios::binary);
```

### 2. 打开模式

模式通过位或组合：

| 模式 | 含义 |
| :--- | :--- |
| `std::ios::in` | 读取 |
| `std::ios::out` | 写入 |
| `std::ios::app` | 追加，每次写都在末尾 |
| `std::ios::ate` | 打开后定位到末尾 |
| `std::ios::trunc` | 截断文件（默认用于 `out`） |
| `std::ios::binary` | 二进制模式 |

几个容易混淆的点：

- `out` 默认隐含 `trunc`，会**清空文件**。想保留原内容要加 `app` 或 `ate`。
- `app` 和 `ate` 不同：`app` 强制每次写入都在末尾，`ate` 只是打开时定位到末尾，之后可以 `seekp` 到别处。
- `in | out` 打开的文件必须已存在，不会自动创建。

```c++
// 追加写入
std::ofstream log("app.log", std::ios::app);

// 读写已有文件，不清空
std::fstream db("data.db", std::ios::in | std::ios::out | std::ios::binary);
```

### 3. 检查是否成功

打开失败不会抛异常，必须显式检查：

```c++
std::ifstream in("maybe_missing.txt");
if (!in)
{
    std::cerr << "打开失败\n";
    return;
}

// 或者用 is_open()
if (!in.is_open()) { /* ... */ }
```

**这是最常见的坑**——忘记检查，后面所有读取都静默失败，得到一堆空数据。

## 二、读取文件

### 1. 逐词读取

`>>` 按空白字符（空格、制表符、换行）分割：

```c++
std::ifstream in("numbers.txt");
int n;
while (in >> n)
{
    std::cout << n << '\n';
}
```

`in >> n` 返回流对象，在布尔上下文中表示「读取是否成功」。读到文件末尾或格式不匹配时返回 `false`，循环自然结束。

### 2. 逐行读取

`std::getline` 读取整行（不含换行符）：

```c++
std::ifstream in("data.txt");
std::string line;
while (std::getline(in, line))
{
    std::cout << line << '\n';
}
```

**不要用 `while (!in.eof())`**，这是经典错误：

```c++
// 错误：最后一行会被处理两次
while (!in.eof())
{
    std::getline(in, line);
    process(line);   // 读到 EOF 后 line 还是上一行的内容
}

// 正确
while (std::getline(in, line))
    process(line);
```

原因是 `eof()` 只有在**尝试读取并失败后**才为真。用 `!eof()` 判断时，最后一次读取已经失败了，但循环体还是会执行一遍。

### 3. 混合使用 >> 和 getline

`>>` 会留下换行符在缓冲区里，紧接着的 `getline` 会读到空行：

```c++
int id;
std::string name;

in >> id;                    // 读完数字，换行符还在缓冲区
std::getline(in, name);      // 读到空字符串！

// 正确：先吃掉换行符
in >> id;
in.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
std::getline(in, name);
```

### 4. 读取整个文件

```c++
// 方法一：用迭代器
std::ifstream in("data.txt");
std::string content((std::istreambuf_iterator<char>(in)),
                     std::istreambuf_iterator<char>());

// 方法二：先求大小再读（适合二进制）
in.seekg(0, std::ios::end);
std::size_t size = in.tellg();
in.seekg(0, std::ios::beg);

std::string buffer(size, '\0');
in.read(buffer.data(), size);
```

方法二更快（一次读取），但要求文件大小可求——管道、标准输入这类流不支持。

## 三、写入文件

```c++
std::ofstream out("output.txt");

out << "Hello\n";
out << 42 << ' ' << 3.14 << '\n';

// 检查写入是否成功
if (!out)
    std::cerr << "写入失败\n";
```

### 1. 缓冲区与刷新

写入先进入缓冲区，以下情况才真正落盘：

- 缓冲区满
- 显式调用 `flush()`
- 流对象析构
- 程序正常结束

```c++
out << "important data";
out.flush();          // 立即写入
out << std::endl;     // 换行并刷新（等价于 '\n' << flush）
```

**程序崩溃时缓冲区内容会丢失**。关键数据要主动 `flush`。

### 2. 检查写入错误

写入错误（磁盘满、权限问题）不会立即抛出，需要检查：

```c++
out << data;
if (!out)
{
    std::cerr << "写入失败\n";
    // 处理错误
}
```

## 四、二进制读写

文本模式会做换行符转换（Windows 上 `\n` ↔ `\r\n`），二进制模式不做任何转换。

```c++
// 写入
struct Record { int id; double value; };

Record r{1, 3.14};
std::ofstream out("data.bin", std::ios::binary);
out.write(reinterpret_cast<const char*>(&r), sizeof(r));

// 读取
Record r2;
std::ifstream in("data.bin", std::ios::binary);
in.read(reinterpret_cast<char*>(&r2), sizeof(r2));
```

**注意**：直接读写结构体内存在跨平台问题——字节序、对齐、`sizeof` 都可能不同。可移植的格式需要逐字段序列化。

## 五、定位

文件流支持随机访问：

```c++
std::fstream file("data.bin", std::ios::in | std::ios::out | std::ios::binary);

// 定位到第 100 字节
file.seekg(100, std::ios::beg);    // 读指针
file.seekp(100, std::ios::beg);    // 写指针

// 获取当前位置
auto pos = file.tellg();

// 定位到末尾
file.seekg(0, std::ios::end);
auto size = file.tellg();
```

三个参考位置：`std::ios::beg`（开头）、`std::ios::cur`（当前位置）、`std::ios::end`（末尾）。

`g` 后缀是 get（读），`p` 后缀是 put（写）。`fstream` 有两个独立的指针。

## 六、关闭与作用域

`close()` 可以手动关闭，但通常不需要——析构函数会自动关闭：

```c++
{
    std::ofstream out("file.txt");
    out << "data";
}   // 离开作用域，自动关闭并刷新
```

**关闭后流的错误状态会保留**。如果之前写入失败，析构时不会抛异常，错误被静默吞掉。所以关键写入要显式检查：

```c++
out << data;
if (!out) { /* 处理错误 */ }
out.close();   // 此时可以检查 close 是否成功
```

## 七、异常处理

默认情况下文件流**不抛异常**，通过状态位报告错误。可以开启异常：

```c++
std::ifstream in;
in.exceptions(std::ios::failbit | std::ios::badbit);   // 打开这些状态时抛异常
try
{
    in.open("missing.txt");
}
catch (const std::ios_base::failure& e)
{
    std::cerr << "打开失败: " << e.what() << '\n';
}
```

但要注意：开启 `failbit` 后，**读到文件末尾也会抛异常**，因为 EOF 会设置 `failbit`。这通常不是想要的，所以很多项目选择不开异常。

## 八、注意事项

**必须检查打开是否成功**。这是文件操作最常见的错误来源。

**不要用 `while (!in.eof())`**。用 `while (in >> x)` 或 `while (getline(in, line))`。

**`ofstream` 默认截断文件**。想追加必须显式指定 `std::ios::app`。

**`>>` 和 `getline` 混用要清缓冲区**。用 `ignore()` 吃掉残留的换行符。

**文本模式会转换换行符**。跨平台处理二进制数据时记得加 `std::ios::binary`。

**缓冲区内容可能丢失**。程序崩溃时未刷新的数据会丢，关键数据主动 `flush()`。

**不要直接读写结构体二进制**。字节序和对齐差异会导致跨平台不兼容。

**`std::endl` 有性能开销**。它每次都刷新缓冲区，循环里输出大量数据时用 `'\n'` 更快。

**文件句柄是有限资源**。打开大量文件而不关闭会耗尽句柄，用 RAII 让析构自动处理。

## 九、相关章节

- [文件与流](../Stream.md)：流模型与类层次结构
- [标准输入输出流](./Iostream.md)：`cin` / `cout` 与格式化控制
- [字符串流](./Sstream.md)：内存字符串的读写
- [文件系统](../Filesystem.md)：路径操作与目录遍历
- [格式化输出](../Format.md)：`std::format` 格式化写入内容
