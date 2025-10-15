# 类与对象

C++ 是一种支持面向对象编程（Object-Oriented Programming, OOP）的语言。
在面向对象的设计思想中，类（Class）是 C++ 的核心特性之一，常被称为用户自定义的数据类型（user-defined type）。

类用于定义对象的结构与行为，是一种将数据与操作这些数据的函数封装在一起的抽象描述。
在类中，用于存储数据的部分称为成员变量（Member Variables），而用于操作这些数据的函数称为成员函数（Member Functions）。

类本质上是一种模板（Template）或蓝图（Blueprint），通过它可以创建出多个具有相同属性和行为的具体**实例**，这些实例被称为对象（Objects）。每个对象都拥有属于自己的成员变量副本，并可以通过成员函数来执行特定的操作。

## 一、类的结构

在 C++ 中，**类（Class）**是一种由用户定义的数据类型（User-defined Data Type），
它将**数据（成员变量）**与**操作数据的函数（成员函数）**有机地结合在一起，
从而实现**封装（Encapsulation）**与**抽象（Abstraction）**。

一个类的基本结构如下：

```cpp
class ClassName {
private:
    // 私有成员（数据与函数）
protected:
    // 受保护成员
public:
    // 公有成员
};
```

### 1. 类的基本组成

类由以下几个主要部分构成：

| 组成部分                                    | 说明                                            |
| --------------------------------------- | --------------------------------------------- |
| 类名（Class Name）                      | 类的标识符，用于定义和引用该类。                              |
| 成员变量（Member Variables）              | 用于存储对象状态的数据。                                  |
| 成员函数（Member Functions）              | 用于操作数据或定义对象行为的函数。                             |
| 访问控制符（Access Specifiers）            | 控制外部对类成员的访问权限：`private`、`protected`、`public`。 |
| 构造函数与析构函数（Constructor & Destructor） | 对象创建与销毁时自动调用的特殊成员函数。                          |
| 静态成员（Static Members）                | 属于类本身而非某个对象的成员。                               |
| 友元（Friend）                          | 特殊访问权限，允许外部函数或类访问私有成员。                        |

### 2. 访问控制符（Access Specifiers）

访问控制符用于限定类成员的可见性和访问范围：

| 控制符         | 访问范围     | 典型用途        |
| ----------- | -------- | ----------- |
| `private`   | 仅类内可访问   | 封装内部实现细节    |
| `protected` | 类内和子类可访问 | 继承时保留部分访问权限 |
| `public`    | 任何地方都可访问 | 提供对外接口      |

示例：

```cpp
class Example {
private:
    int secret;          // 私有成员
protected:
    int semi_secret;     // 受保护成员
public:
    int visible;         // 公有成员
};
```

> 默认情况下，`class` 的成员默认是 **private**，
> 而 `struct` 的成员默认是 **public**。

### 3. 成员变量（Member Variables）

成员变量用于存储对象的状态。
每个对象都会拥有独立的一份成员变量副本。

```cpp
class Student {
private:
    std::string name;
    int age;
};
```

> 成员变量可以是基本类型、对象、指针、引用、数组、容器等。

也可以为成员变量提供默认初始值（C++11 起支持）：

```cpp
class Point {
    int x = 0;
    int y = 0;
};
```

### 4. 成员函数（Member Functions）

成员函数用于定义对象的行为，通常用于访问和修改成员变量。

```cpp
class Student {
private:
    std::string name;
    int age;

public:
    void setInfo(const std::string& n, int a) {
        name = n;
        age = a;
    }

    void display() {
        std::cout << "Name: " << name << ", Age: " << age << std::endl;
    }
};
```

也可以在类外定义成员函数：

```cpp
void Student::display() {
    std::cout << "Name: " << name << ", Age: " << age << std::endl;
}
```

### 5. 构造函数（Constructor）

构造函数在对象创建时自动执行，用于初始化成员变量。
它与类同名，无返回值。

```cpp
class Student {
private:
    std::string name;
    int age;

public:
    // 构造函数
    Student(const std::string& n, int a) : name(n), age(a) {
        std::cout << "Constructor called." << std::endl;
    }
};
```

构造函数的种类包括：

| 类型     | 说明         | 示例                          |
| ------ | ---------- | --------------------------- |
| 默认构造函数 | 无参版本       | `Student()`                 |
| 有参构造函数 | 初始化时传参     | `Student("张三", 20)`         |
| 拷贝构造函数 | 用已有对象创建新对象 | `Student(const Student& s)` |

构造函数支持多态，可以使用多个参数不同的构造函数，在使用时会自动匹配。

### 6. 析构函数（Destructor）

析构函数在对象销毁时自动调用，用于资源释放或清理。

```cpp
class Student {
public:
    ~Student() {
        std::cout << "Destructor called." << std::endl;
    }
};
```

> 析构函数名以 `~` 开头，无参、无返回值，每个类最多有一个析构函数。

### 7. this 指针

`this` 是一个隐含指针，指向调用成员函数的**当前对象**。
它可用于区分成员变量与同名参数：

```cpp
class Student {
private:
    std::string name;

public:
    Student(const std::string& name) {
        this->name = name; // 区分成员变量与参数
    }
};
```

### 8. 静态成员（Static Members）

静态成员属于类本身，而非具体对象。
所有对象共享同一份静态数据。

静态成员分为两类：

1. 静态成员变量（Static Member Variables）
2. 静态成员函数（Static Member Functions）

#### (1)静态成员变量

静态成员变量在所有对象之间共享同一份存储空间。
它不依赖于任何对象存在，无论创建多少个对象，这个变量都只有一份。

因此，静态成员变量常用于表示类级别的公共信息，例如计数器、配置、全局状态等。

```cpp
class Counter {
public:
    static int count;
    Counter() { count++; }
};

int Counter::count = 0;
```

#### (2)静态函数变量

静态成员函数同样属于类本身，而不是对象。
它的主要特征是：

* 不依赖任何对象实例
* 无法访问非静态成员变量或函数（因为没有具体对象可供操作）
* 通常用于类级别的操作，如访问静态数据或执行与对象无关的逻辑。
* 没有 `this` 指针，因为它不属于任何对象

```cpp
class Counter {
private:
    static int count;

public:
    Counter() { count++; }

    // 静态成员函数
    static void showCount() {
        std::cout << "Current count: " << count << std::endl;
    }
};

// 类外定义静态变量
int Counter::count = 0;

int main() {
    Counter a, b;
    Counter::showCount();  // 通过类名访问
    a.showCount();         // 也可通过对象访问
}
```

### 9. 常成员（const Members）

* **常成员函数**：在函数后加 `const`，表示不修改成员变量。
* **常对象**：对象一旦定义，其状态不可更改。

```cpp
class Point {
private:
    int x, y;

public:
    void display() const {
        std::cout << "(" << x << ", " << y << ")" << std::endl;
    }
};
```

### 10. 友元（Friend）

友元函数或友元类可以访问类的私有成员。
这在**操作符重载**、类间紧密协作时非常有用。

```cpp
#include <iostream>

class Box {
private:
    int width;
public:
    void set_width(int val){
        width = val;
    }
    friend void printWidth(Box b);
};

void printWidth(Box b) {
    std::cout << "Width: " << b.width << std::endl;
}

int main(){
    Box a;
    a.set_width(20);
    printWidth(a);
}
```

> 友元会破坏封装性，应谨慎使用。
>
> 友元函数不是成员函数，不能用 对象.函数() 方式调用。它只是可以访问类私有成员的普通函数，调用方式与普通函数相同。

### 11. 类的组合与嵌套

一个类可以将另一个类作为成员，这种关系称为组合（Composition）。

```cpp
class Address {
public:
    std::string city;
};

class Person {
public:
    std::string name;
    Address addr;  // 组合
};
```

> 当对象销毁时，其组合成员也会自动销毁。

## 二、对象

在 C++ 中，对象（Object）是类的实例（Instance）。
当我们定义了一个类后，这个类本身只是一个抽象的模板或蓝图，
而对象才是真正占用内存、可操作的具体实体。

类定义了“事物的共性”，对象体现了“事物的个性”。

### 1. 对象的创建与定义

定义一个类对象与定义普通变量非常相似：

```cpp
class Student {
public:
    std::string name;
    int age;
};

int main() {
    Student s1;  // 创建对象 s1
    s1.name = "张三";
    s1.age = 20;

    Student s2;  // 再创建一个对象 s2
    s2.name = "李四";
    s2.age = 21;
}
```

> 每个对象都有**独立的成员变量副本**，互不影响。

例如：

```cpp
s1.age = 20;
s2.age = 21;
// 修改 s1 的 age 不会影响 s2
```

### 2. 对象的初始化

当类中定义了构造函数时，对象可以在定义时直接初始化：

```cpp
class Point {
private:
    int x, y;

public:
    Point(int a, int b) {  // 构造函数
        x = a;
        y = b;
    }

    void show() {
        std::cout << "(" << x << ", " << y << ")" << std::endl;
    }
};

int main() {
    Point p1(1, 2);  // 调用构造函数
    Point p2 = Point(3, 4); // 另一种写法
    p1.show();
    p2.show();
}
```

> 注意：对象创建时，构造函数会被自动调用；对象销毁时，析构函数会被自动调用。

### 3. 对象的作用域与生命周期

对象的生命周期与其**定义的位置**有关。

| 定义位置     | 生命周期说明                      |
| -------- | --------------------------- |
| 局部对象（栈上） | 作用域结束时自动销毁                  |
| 全局对象     | 程序结束时销毁                     |
| 静态对象     | 程序结束时销毁                     |
| 动态对象（堆上） | 需要手动释放（使用 `new` / `delete`） |

示例：

```cpp
class Example {
public:
    Example() { std::cout << "Constructed\n"; }
    ~Example() { std::cout << "Destructed\n"; }
};

int main() {
    Example local;            // 局部对象
    static Example global;    // 静态对象
    Example* ptr = new Example();  // 动态对象

    delete ptr; // 手动释放
}
```

输出顺序体现了不同对象的生命周期管理。

### 4. 对象数组

可以定义一个包含多个对象的数组：

```cpp
class Point {
public:
    int x, y;
    Point(int a = 0, int b = 0) : x(a), y(b) {}
};

int main() {
    Point arr[3] = { {1,2}, {3,4}, {5,6} };
    for (auto& p : arr)
        std::cout << "(" << p.x << ", " << p.y << ")\n";
}
```

> 若类中没有默认构造函数，则必须在定义对象数组时显式提供初始化参数。

### 5.对象的指针与引用

#### (1)对象指针

和基本类型类似，可以使用指针指向对象。

```cpp
Student s1;
Student* ptr = &s1;
ptr->name = "王五";
ptr->age = 22;
```

> 使用 `->` 运算符访问对象的成员。

也可以使用 `new` 创建动态对象：

```cpp
Student* stu = new Student;
stu->name = "赵六";
stu->age = 18;
delete stu; // 释放内存
```

#### (2)对象引用

引用可以直接操作已有对象：

```cpp
Student s1;
Student& ref = s1;
ref.name = "张三";
```

> 引用不会创建新对象，只是为已有对象取别名。

### 6. 对象的拷贝与赋值

当我们用一个对象初始化另一个对象时，会自动调用**拷贝构造函数**。

```cpp
class Box {
public:
    int width;
    Box(int w) : width(w) {}
    Box(const Box& b) { // 拷贝构造函数
        width = b.width;
        std::cout << "Copy constructor called\n";
    }
};

int main() {
    Box b1(10);
    Box b2 = b1; // 调用拷贝构造函数
}
```

赋值操作调用的是**赋值运算符（operator=）**，而不是拷贝构造。

### 7. const 对象

可以将对象声明为常量，使其内容不可被修改：

```cpp
class Point {
public:
    int x, y;
    void show() const {
        std::cout << x << ", " << y << std::endl;
    }
};

int main() {
    const Point p = {1, 2};
    p.show();
    // p.x = 5; // 错误：常对象不可修改成员
}
```

> 常对象只能调用常成员函数（即声明为 `void func() const` 的函数）。

### 8. 对象之间的比较与赋值

对象可以相互赋值：

```cpp
Student s1, s2;
s1.name = "A";
s2 = s1; // 成员变量逐个拷贝
```

但若希望对象间的比较（==、< 等）有意义，需要**重载运算符**（进阶内容，后续讲述）。

### 9. 对象与内存模型

每个对象在内存中都有**独立的成员变量副本**：

```text
Student s1 ──► name="张三", age=20
Student s2 ──► name="李四", age=21
```

而静态成员在所有对象间共享。

## 三、完整示例

```cpp
#include <iostream>
#include <string>
using namespace std;

class Student {
public:
    string name;
    int age;

    Student(string n = "未知", int a = 0) : name(n), age(a) {
        cout << "Constructed: " << name << endl;
    }

    ~Student() {
        cout << "Destructed: " << name << endl;
    }

    void show() const {
        cout << "Name: " << name << ", Age: " << age << endl;
    }
};

int main() {
    Student s1("张三", 20);
    Student s2("李四", 21);

    s1.show();
    s2.show();

    Student* p = new Student("王五", 22);
    p->show();
    delete p;  // 手动释放动态对象
}
```

**输出：**

```text
Constructed: 张三
Constructed: 李四
Constructed: 王五
Name: 张三, Age: 20
Name: 李四, Age: 21
Name: 王五, Age: 22
Destructed: 王五
Destructed: 李四
Destructed: 张三
```
