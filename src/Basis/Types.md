# 数据类型

在 C++ 中，**数据类型（Data Types）** 是程序设计的基础，它定义了变量在内存中的存储方式、可表示的取值范围以及能进行的操作。
理解数据类型不仅是掌握语法的关键，也是进行性能优化、内存管理和类型安全编程的前提。

C++ 的类型系统丰富而严格，既包含直接映射到硬件的**基本类型（Fundamental Types）**，也支持由这些类型构建的**派生类型（Derived Types）**，此外还提供了面向对象与泛型编程所需的抽象类型系统。

本章将系统介绍 C++ 中的数据类型体系结构，内容包括：

- [基本数据类型](./Types/Fundamental_Types.md)：
  描述整型、浮点型、布尔型、空类型等语言内建类型及其修饰符（`signed`、`unsigned`、`long` 等）。

- [派生数据类型](./Types/Derived_Types.md)：
  介绍数组、指针、引用、函数、结构体、类、枚举等由基本类型组合或扩展而来的类型。

- [字符串类型](./Types/Character.md)：
  涵盖 C 风格字符串 (`char[]`) 与 C++ 字符串类 (`std::string`、`std::wstring`) 的使用与区别。

- [类型转换](./Types/Type_Conversion.md)：
  探讨隐式与显式类型转换、C 风格转换与 C++ 四种安全转换（`static_cast`、`const_cast`、`reinterpret_cast`、`dynamic_cast`）。

- [类型推导](./Types/Deducing_Types.md)：
  说明 `auto`、`decltype`、模板参数推导等机制如何简化类型声明并保持类型安全。
