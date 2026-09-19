# 文件系统 (filesystem)

C++17 引入的文件系统库，提供跨平台的路径操作、目录遍历与文件属性查询。

在此之前，这些操作要么用 C 的 `<dirent.h>`、`<sys/stat.h>`（POSIX）或 `<windows.h>`（Windows），要么用 Boost.Filesystem。现在标准库有了统一接口。

## 1. 引入

```c++
#include <filesystem>
namespace fs = std::filesystem;   // 标准做法：起个短别名
```

## 2. 核心概念

### 1. path：路径

`std::filesystem::path` 表示一个路径，它**不检查路径是否存在**，只是字符串的封装，负责处理不同平台的路径格式差异。

```c++
fs::path p1 = "/home/user/file.txt";      // POSIX
fs::path p2 = "C:\\Users\\file.txt";      // Windows
fs::path p3 = "/home" / "user" / "file.txt";   // 用 / 拼接
```

路径拼接用 `operator/`，它会自动处理分隔符：

```c++
fs::path dir = "/home/user";
fs::path file = dir / "data" / "input.txt";   // /home/user/data/input.txt
```

### 2. 路径的组成部分

```c++
fs::path p = "/home/user/data.txt";

p.root_name();      // ""（Windows 上是 "C:"）
p.root_directory(); // "/"
p.parent_path();    // "/home/user"
p.filename();       // "data.txt"
p.stem();           // "data"
p.extension();      // ".txt"
```

### 3. 文件类型

```c++
enum class file_type {
    none,            // 不存在或无法确定
    not_found,       // 不存在
    regular,         // 普通文件
    directory,       // 目录
    symlink,         // 符号链接
    block,           // 块设备
    character,       // 字符设备
    fifo,            // 命名管道
    socket,          // socket
    unknown,         // 存在但类型未知
};
```

## 3. 用法

### 1. 查询文件状态

```c++
fs::path p = "data.txt";

// 是否存在
if (fs::exists(p)) { }

// 类型判断
fs::is_regular_file(p);   // 普通文件
fs::is_directory(p);      // 目录
fs::is_symlink(p);        // 符号链接

// 文件大小（字节）
auto size = fs::file_size(p);

// 最后修改时间
auto time = fs::last_write_time(p);
```

这些函数都有两个版本：一个是 `fs::exists(p)`，失败时抛异常；另一个是 `fs::exists(p, ec)`，把错误写进 `std::error_code`：

```c++
std::error_code ec;
if (fs::exists(p, ec)) { }
if (ec)
    std::cerr << "错误: " << ec.message() << '\n';
```

**在预期可能失败的地方（比如检查不存在的文件），用 `error_code` 版本**，避免异常开销。

### 2. 目录操作

```c++
// 创建目录
fs::create_directory("newdir");              // 父目录必须存在
fs::create_directories("a/b/c");             // 递归创建，父目录不存在也行

// 删除
fs::remove("file.txt");                      // 删除文件或空目录
fs::remove_all("dir");                       // 递归删除

// 重命名/移动
fs::rename("old.txt", "new.txt");

// 复制
fs::copy("src.txt", "dst.txt");
fs::copy("srcdir", "dstdir", fs::copy_options::recursive);   // 递归复制
```

### 3. 遍历目录

**非递归遍历**：

```c++
for (const auto& entry : fs::directory_iterator("."))
{
    std::cout << entry.path() << '\n';
}
```

**递归遍历**：

```c++
for (const auto& entry : fs::recursive_directory_iterator("."))
{
    if (entry.is_regular_file())
        std::cout << entry.path() << " (" << entry.file_size() << " 字节)\n";
}
```

遍历顺序是**未指定的**。如果需要排序，先收集到容器再排：

```c++
std::vector<fs::path> files;
for (const auto& entry : fs::directory_iterator("."))
    files.push_back(entry.path());

std::sort(files.begin(), files.end());
```

遍历过程中可以控制：

```c++
for (auto it = fs::recursive_directory_iterator("."); it != fs::recursive_directory_iterator(); ++it)
{
    if (it->path().extension() == ".tmp")
        it.disable_recursion_pending();   // 不进入这个目录
    // 或者 it.pop() 跳过当前目录的剩余内容
}
```

### 4. 路径转换

```c++
fs::path p = "/home/user/data.txt";

// 转字符串
std::string s = p.string();              // 本地编码
std::wstring ws = p.wstring();           // 宽字符
std::u8string u8 = p.u8string();         // UTF-8（C++20）
std::u16string u16 = p.u16string();      // UTF-16

// 从字符串构造
fs::path p2 = std::string("/tmp/file");
```

> 在 Windows 上 `path::string()` 可能因为编码问题失败或产生乱码。跨平台代码建议用 `u8string()`（C++20 起返回 `std::u8string`）。

### 5. 实用示例

**递归统计目录大小**：

```c++
std::uintmax_t directorySize(const fs::path& dir)
{
    std::uintmax_t total = 0;
    for (const auto& entry : fs::recursive_directory_iterator(dir))
    {
        if (entry.is_regular_file())
            total += entry.file_size();
    }
    return total;
}
```

**查找特定扩展名的文件**：

```c++
std::vector<fs::path> findFiles(const fs::path& dir, const std::string& ext)
{
    std::vector<fs::path> result;
    for (const auto& entry : fs::recursive_directory_iterator(dir))
    {
        if (entry.is_regular_file() && entry.path().extension() == ext)
            result.push_back(entry.path());
    }
    return result;
}

auto cppFiles = findFiles(".", ".cpp");
```

**确保目录存在**：

```c++
void ensureDirectory(const fs::path& dir)
{
    if (!fs::exists(dir))
        fs::create_directories(dir);
}
```

## 4. 注意事项

**`path` 不验证存在性**。构造 `fs::path` 只是封装字符串，不会检查路径是否有效。要检查得调用 `fs::exists()`。

**`directory_iterator` 的顺序未指定**。不要假设遍历顺序，需要有序就自己排序。

**遍历时修改目录是未定义行为**。不要在遍历过程中创建或删除文件，先收集路径再操作。

**注意异常与 `error_code` 的选择**。默认版本抛 `fs::filesystem_error`，带 `error_code` 参数的版本不抛。在「文件可能不存在」这类正常情况用后者。

**`file_size` 对目录无效**。目录的大小是平台相关的，没有可移植的含义。

**符号链接的处理**。默认情况下，`is_directory(p)` 会跟随符号链接。要判断链接本身，用 `is_symlink(p)`；要获取链接目标，用 `read_symlink(p)`。

**路径分隔符**。写代码时用 `/`，`path` 会在 Windows 上自动转换。不要硬编码 `\\`。

**C++17 支持情况**。需要链接 `stdc++fs`（GCC 8）或 `c++fs`（Clang 7），较新版本已不需要：

```bash
# 老版本 GCC
g++ -std=c++17 main.cpp -lstdc++fs
```

**`last_write_time` 的时钟**。它用的是 `std::filesystem::file_time_type`，不是 `system_clock`，与 `chrono` 互转需要额外处理。

## 5. 相关章节

- [文件与流](./Stream.md)：`fstream` 读写文件内容
- [字符串](./String.md)：路径字符串的处理
- [时间库](./Chrono.md)：文件时间戳
- [正则表达式](./Regex.md)：按模式筛选文件名
