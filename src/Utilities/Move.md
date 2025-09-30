# std::move

将一个对象的**所有权**从一个地方(特指引用或是指针)转移到另一个地方，相当于rust语言的所有权转让，对于`std::string`或是更复杂的数据结构使用`std::move`转移所有权可以比直接复制有着更高的效率。

其模板定义如下：

```cpp
template< class T >
typename std::remove_reference<T>::type&& move( T&& t ) noexcept; // (since C++11 and until C++14)
```

```cpp
template< class T >
constexpr std::remove_reference_t<T>&& move( T&& t ) noexcept; // (since C++14)
```

## 1. 引入

被包括在`<utility>`头文件中。

```C++
#include <utility>
```

## 2. `std::move`的工作原理

`std::move`并不真正“移动”对象的数据，它的作用是将一个左值引用转化为右值引用，从而启用移动语义。在C++中，右值引用和左值引用的主要区别在于右值引用允许资源的所有权转移，而左值引用则保持资源的所有权。

通过使用`std::move`，编译器能够区分何时使用**拷贝构造**（复制数据）和**移动构造**（转移所有权），从而优化程序性能，特别是在涉及大量数据拷贝的场景中。

### (1)右值引用与左值引用

* 左值：表示可以被引用且具有持久地址的对象（例如，变量）。
* 右值：表示临时对象或可以被销毁的对象（例如，常量、字面值、函数返回值）。

`std::move`的作用是将左值转换为右值引用，允许资源的移动。

### (2)为什么需要`std::move`？

C++11引入了移动语义（Move Semantics），目标是提高性能。复制一个大对象时，可能涉及到大量的内存分配和数据拷贝，而如果能将资源的所有权直接转移给另一个对象，就可以避免这些开销。`std::move`通过将左值转换为右值引用，促使移动构造函数或移动赋值运算符被调用。

例如：

```cpp
std::vector<int> v1 = {1, 2, 3};
std::vector<int> v2 = std::move(v1);  // 这里v2会接管v1的所有权，v1会处于一个"空"状态
```

在这个例子中，`std::move`将`v1`转换为右值引用，从而触发了移动构造函数。结果是，`v2`接管了`v1`的数据，而`v1`进入了一个有效但不确定的状态。

## 3. 使用`std::move`的场景

### (1)移动构造与移动赋值

在使用`std::move`时，通常会看到它与**移动构造函数**和**移动赋值运算符**一起使用。

* 移动构造函数：当一个对象通过右值引用初始化另一个对象时，调用移动构造函数，通常用于资源的转移。
* 移动赋值运算符：当一个对象被右值引用赋值给另一个已存在的对象时，调用移动赋值运算符。

例如：

```cpp
std::vector<int> v1 = {1, 2, 3};
std::vector<int> v2 = std::move(v1);  // 使用移动构造函数
```

对于赋值操作：

```cpp
std::vector<int> v1 = {1, 2, 3};
std::vector<int> v2;
v2 = std::move(v1);  // 使用移动赋值运算符
```

不管是哪一种操作，`v1`都会被剥夺所有权。

### (2)自定义类型与`std::move`

如果你有自定义类型，并希望通过移动语义来优化性能，那么你需要为这个类型定义**移动构造函数**和**移动赋值运算符**。这些函数通常需要显式地将资源的所有权从一个对象转移到另一个对象。

例如：

```cpp
class MyClass {
public:
    MyClass(MyClass&& other) noexcept {
        // 移动构造函数
        this->data = std::move(other.data);  // 移动数据
    }

    MyClass& operator=(MyClass&& other) noexcept {
        // 移动赋值运算符
        if (this != &other) {
            this->data = std::move(other.data);  // 移动数据
        }
        return *this;
    }

private:
    std::vector<int> data;
};
```

## 4. `std::move`的常见误用

### (1)移动之后使用原对象

在将对象通过`std::move`转移所有权之后，原对象的状态是未定义的。因此，**不应再使用原对象**，例如：

```cpp
std::vector<int> v1 = {1, 2, 3};
std::vector<int> v2 = std::move(v1);

// v1现在处于未定义状态，不能再安全使用
std::cout << v1.size();  // 错误
```

正确的做法是，只在转移所有权后“放弃”原对象的使用。

### (2)不必要的使用`std::move`

`std::move`的作用是告诉编译器“这不是一个左值，你可以安全地移动它”，但是如果你已经明确知道对象不会被复制（例如返回一个对象的右值引用），就不必使用`std::move`。

例如：

```cpp
std::vector<int> createVector() {
    std::vector<int> v = {1, 2, 3};
    return v;  // 编译器会自动进行返回值优化（RVO），不需要手动使用std::move
}
```

这里的返回值优化（RVO）允许编译器在返回`v`时进行移动，而不需要显式调用`std::move`。
