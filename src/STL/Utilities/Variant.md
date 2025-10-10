# `std::variant`

`std::variant` 是 C++17 引入的 **类型安全联合（type-safe union）**。在任一时刻，`variant<...>` 要么保存其候选类型列表中的某一类型的对象（active alternative），要么在异常情况等导致的特殊情形下处于**无值**状态（`valueless_by_exception()`）。

头文件：

```cpp
#include <variant>
```

## 1. 模板定义

```cpp
template< class... Types >
class variant;
```

* 模板参数为 `Types...`：每个 `T` 必须满足 **Destructible**（能被析构）。
* **不允许**持有引用类型、数组类型或 `void`。
* 可以重复出现相同类型（例如 `variant<int,int>` 合法），也可以出现不同 cv 限定的同一基础类型（如 `int` 与 `const int`）。
* **注意**：如果你用同一个具体类型多次，基于类型的访问（`std::get<T>` / `get_if<T>`）会变得**歧义 / 编译失败**（只能用索引或明确 `in_place_type`/`in_place_index`）。
* 默认构造：**默认构造会构造第一个候选类型的默认值**，如果第一个候选类型不可默认构造，则 `variant` 本身也不可默认构造。可以把 `std::monostate` 放在首位以保证可默认构造。

## 2. 存储与对象布局

* `variant` 内部**存储了 discriminator（索引）与一个能容纳最大候选类型的缓冲区**；当 `variant` 持有某个类型 `T` 时，一个 `T` 对象会嵌套（placement-new）在该缓冲区内。
* 因此 `variant` 的大小≈（max sizeof(alternatives)）+ 对齐 + discriminator 大小。
* 在异常情况下（构造/赋值期间）有可能变为 **valueless_by_exception**（见下文）。

## 3. 主要成员函数 / 重载

> 下面列出常用操作、签名（伪签名风格）与行为说明与例子。

### 构造与析构

* `variant()`

  * 默认构造：构造第一个候选类型的默认值（若可行）。
  * 否则 `variant` 不可默认构造。

* `variant(const variant&)` / `variant(variant&&)`

  * 拷贝 / 移动构造。条件：候选类型支持相应操作；若某些候选类型不可拷贝/移动，相应操作会被删除。
  * 如果在移动过程中抛出异常，可能导致 `valueless_by_exception`（取决于具体实现与异常传播）。

* converting constructors（从某个值构造）

  * 如果传入 `U` 可明确/唯一地构造某个候选类型，`variant` 会构造该候选。若存在二义性（能构造多个候选），编译失败。

* in-place 构造（直接在 variant 内就地构造）

  ```cpp
  variant(in_place_type<T>, Args&&...);
  variant(in_place_index<I>, Args&&...);
  ```

  * `in_place_type_t` / `in_place_index_t` 用于在 variant 内直接构造目标 alternative，避免先创建临时再赋值。

* `~variant()`

  * 默认析构：会调用当前活动 alternative 的析构函数（如果有值）。

### 赋值（operator=）

* `variant& operator=(const variant&);`
* `variant& operator=(variant&&);`

  * 这两个做拷贝/移动赋值。赋值行为在不同情况下（同类型 index / 不同 index）会调用相应 alternative 的赋值/析构+构造。
  * 赋值过程中若抛异常，可能使 `variant` 进入 `valueless_by_exception`。
* `template<class T> variant& operator=(T&&);`

  * converting assignment：当 `T` 可以唯一构造某个候选类型时，执行相应赋值/替换。
* `variant& operator=(std::monostate)` 等（视候选类型而定）。

### 观察器（Observers）

* `std::size_t index() const noexcept;`

  * 返回当前活动的候选类型的零基索引（0..N-1）。
  * 如果处于 `valueless_by_exception`，返回 `variant_npos`（常为 `std::size_t(-1)`）。
* `bool valueless_by_exception() const noexcept;`

  * 如果 `variant` 处于无值状态（例如在变更 active alternative 时异常导致）返回 `true`。

### 修改（Modifiers）

* `template<class T, class... Args> T& emplace(Args&&...);`

  * `emplace<T>(args...)`：在 `variant` 中就地构造类型 `T`（T 必须是某个 alternative）；会销毁旧的 active 值（若有），然后就地构造新值。
  * 异常安全：如果构造抛出，`variant` 可能进入 `valueless_by_exception`（取决于实现与被替换对象的销毁时机）。
* `template<size_t I, class... Args> variant& emplace(in_place_index_t<I>, Args&&...);`

  * 使用索引 I 就地构造。
* `void swap(variant& other) noexcept( /* depends */ );`

  * 交换两个 variant 的状态与内容。noexcept 与具体候选类型的 swap/移动操作相关。

### 访问（get / get_if）

* `std::get<T>(variant&)` / `std::get<I>(variant&)`

  * `get<T>`（按类型访问）要求 `T` 在候选类型中**唯一**，否则编译错误。
  * `get<I>`（按索引访问）直接访问索引为 `I` 的候选类型。
  * 若 `variant` 当前不保存请求的 alternative，`std::get` 会 **抛出 `std::bad_variant_access`（运行时异常）**。
* `std::get_if<T>(&variant)` / `std::get_if<I>(&variant)`

  * `get_if` 返回指向当前值的指针（非 `nullptr` 表示匹配），失败时返回 `nullptr`。不会抛异常，通常是更安全的访问方式。
  * 有 `const` / 非 `const` 重载：`const T* get_if<const T>(&const variant)` 等。

示例：

```cpp
std::variant<int,std::string> v = "hi";
if (auto p = std::get_if<std::string>(&v)) {
    std::cout << *p << "\n";
}
try {
    std::cout << std::get<int>(v); // 抛出 std::bad_variant_access
} catch (const std::bad_variant_access& e) { ... }
```

### 访问辅助：`std::holds_alternative<T>(v)`

* 返回 `true` 当且仅当 `v` 当前持有类型 `T`（同 `get_if<T>` 非空）。`T` 必须唯一出现在 alternatives 中。

### 访问索引常量

* `constexpr std::size_t variant_npos = /* often size_t(-1) */;`

  * 表示无效索引（用于 `index()` 返回值在 `valueless_by_exception()` 时）。

## 4. 访问与遍历：`std::visit`（Visitor 模式）

### 非成员 `std::visit`（自 C++17 起）

签名（概念）：

```cpp
template <class Visitor, class... Variants>
decltype(auto) visit(Visitor&& vis, Variants&&... vars);
```

* `std::visit` 会将 visitor（可调用对象）以当前 variant（或多个 variants）的活动值作为参数调用。
* 当传入多个 `variant` 时，visitor 会被调用，参数顺序与 `variant` 顺序对应。
* 如果任一 `variant` 为 `valueless_by_exception()`，`std::visit` 通常会抛出 `std::bad_variant_access`。
* 返回值类型由 visitor 决定（可以返回 `void` 或其他类型）。
* 常用技巧：用 `overloaded` （多个 lambda 继承合并）来实现多分支处理：

```cpp
// helper
template<class... Fs> struct overloaded : Fs... { using Fs::operator()...; };
template<class... Fs> overloaded(Fs...) -> overloaded<Fs...>;

// 使用
std::variant<int,std::string> v = 42;
std::visit(overloaded {
    [](int i){ std::cout<<"int "<<i<<"\n"; },
    [](const std::string& s){ std::cout<<"str "<<s<<"\n"; }
}, v);
```

### 成员 `visit`（C++26 提案：member visit）

* C++26 引入（或将引入）`v.visit(visitor)` 的成员形式作为便捷写法（请注意你使用的编译器/标准支持情况）。非成员 `std::visit` 在 C++17 就有。

## 5. 比较运算与哈希

* `operator==` 等（C++17 起）与 `operator<=>`（C++20）有定义：通常两个 `variant` 先比较是否都 `valueless_by_exception()`，再比较 `index()`，在 index 相同时比较包含的值（按对应类型的比较运算）。

  * `==`：若两者 `index()` 相同且 contained values 相等 => true；若两个都 valueless => true；否则 false。
  * `<` / `>`：若 `index()` 不同，通常以 `index()` 的大小决定排序；若相同，则调用 contained type 的 `<`。
  * 详细边界（valueless 等）以标准详细定义为准，但通常结果符合“按 index 首先排序，然后按值比较”的直觉。
* `std::hash<std::variant<...>>` 在标准库有特化（要求所有候选类型可哈希）。

## 6. 辅助类型与特性（type traits / helper classes）

* `std::monostate`（C++17）

  * 一个空占位类型，常用于将 `variant` 设置为默认可构造：`std::variant<std::monostate, T1, T2>`。
* `std::bad_variant_access`（C++17）

  * 当用 `std::get<T>` / `std::get<I>` 访问但 `variant` 未持有该 alternative 时抛出。
* `std::variant_size<Variant>` / `std::variant_size_v<Variant>`（C++17）

  * 编译期获取候选类型数量（常量表达式）。
  * 例： `std::variant_size_v<std::variant<int,double>> == 2`。
* `std::variant_alternative<I, Variant>::type` / `std::variant_alternative_t<I, Variant>`（C++17）

  * 编译期获取索引 `I` 对应的类型（类型别名）。
  * 例： `std::variant_alternative_t<0,std::variant<int,double>>` 等于 `int`。
* `variant_npos`：表示无值索引（如 `index()` 在 valueless 时返回此值）。

## 7. 异常安全与 `valueless_by_exception`

* 在某些变更 active alternative 的操作中（例如赋值、就地构造时），如果构造/移动/复制新的 alternative 的构造函数抛出异常，而旧对象已被销毁，`variant` 可能无法恢复到原先状态，从而进入 `valueless_by_exception()`。
* 一旦处于 `valueless_by_exception()`：

  * `index()` 返回 `variant_npos`；
  * `std::get` 抛出 `std::bad_variant_access`；
  * `std::get_if` 返回 `nullptr`；
  * 一些操作（比如 `std::visit`）会抛出 `bad_variant_access`（取决于实现）。
* 预防策略：当替换可能抛异常的类型时，优先使用 `emplace` 并在必要时进行异常处理；确保候选类型的构造/移动操作尽可能 `noexcept`，可以降低进入无值状态的风险。

## 8. 常用例子

基本使用与 get/get_if/holds_alternative

```cpp
std::variant<int,std::string> v = "hello";
if (std::holds_alternative<std::string>(v)) {
    std::cout << std::get<std::string>(v) << "\n";
}
if (auto p = std::get_if<int>(&v)) {
    std::cout << "int: " << *p << "\n";
} else {
    std::cout << "not int\n";
}
```

emplace / in_place

```cpp
std::variant<std::monostate, std::string, std::vector<int>> v;
v.emplace<std::string>("abc");              // 就地构造 std::string
v.emplace<std::vector<int>>(3, 42);         // 就地构造 vector(3,42)
v.emplace<in_place_index_t<1>>("xyz");      // 使用索引就地构造（index=1 => std::string）
```

visit 与 overloaded 工具

```cpp
auto handle = overloaded {
    [](int i){ std::cout<<"int "<<i<<"\n"; },
    [](const std::string& s){ std::cout<<"str "<<s<<"\n"; }
};
std::variant<int,std::string> v = 10;
std::visit(handle, v);
```

使用 monostate 使可默认构造

```cpp
std::variant<std::monostate, std::string> v; // 默认构造后 v 持有 monostate
```

## 9. 实用建议 / 常见误用

* **不要把 `variant` 作为替代所有情况**：类型过多会导致代码复杂和 visitor 分支膨胀。若候选类型集合非常大或松散，考虑设计别的抽象（多态/策略等）。
* **当候选类型有重复的具体类型时，避免 `get<T>`**：因为会编译错误；使用 `get<index>` 或 `in_place_type` 显式选择。
* **注意异常安全**：替换 active alternative（赋值、emplace）如果构造抛异常，可能进入 `valueless_by_exception`；为关键路径确保候选类型的移动/复制构造尽可能 `noexcept`。
* **避免把对 `variant` 的访问当作频繁反射**：大量类型判断/切换会影响可读性和性能（虽然 `variant` 本质上是常数时间的判定与访问，但分支与 visitor 的实现复杂度需考虑）。
* std::variant 类似于 Rust 的 enum，都能表示“一种类型中的多种可能”。
    不同点在于Rust 的 enum 语法更简洁，且模式匹配是强制的；C++ 的 std::variant 需要 std::visit 或 get 来显式处理。

## 10. 标准/特性备注

* `std::variant` 自 C++17 引入（特性宏：`__cpp_lib_variant` 等）。
* 标准后续对 `variant` 做过修订（例如 `std::visit` 扩展、constexpr 能力增强等）。例如有成员形式 `visit`（C++26 提议/扩展），以及使 `variant` 更多操作支持 `constexpr`（不同标准版本的支持程度由编译器/标准库实现决定）。
* 使用时注意你的编译器和标准库版本对 `variant` 的各项特性的支持情况（尤其是 `constexpr`、成员 `visit` 等较新特性）。

## 11. 快速 API 参考

* 头文件：`<variant>`
* 构造：`variant()`, `variant(in_place_type_t<T>, ...)`, `variant(in_place_index_t<I>, ...)`, converting constructors
* 赋值：`operator=(variant)`, `operator=(T&&)`（converting）
* 访问：`std::get<T>(v)`, `std::get<I>(v)`, `std::get_if<T>(&v)`, `std::get_if<I>(&v)`
* 情况检测：`v.index()`, `v.valueless_by_exception()`, `std::holds_alternative<T>(v)`
* 就地构造：`v.emplace<T>(args...)`, `v.emplace<in_place_index_t<I>>(args...)`
* 访问模式：`std::visit(visitor, v1, v2, ...)`，C++26 可能支持成员 `v.visit(visitor)`
* 辅助类型：`std::monostate`, `std::bad_variant_access`, `std::variant_size`, `std::variant_alternative_t`, `std::hash<std::variant<...>>`
