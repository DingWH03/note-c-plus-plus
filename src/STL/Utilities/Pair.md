# std::pair

`std::pair` 是一个结构体模板，其可于一个单元内存储两个相异对象。是 std::tuple 的拥有两个元素的特殊情况。一般来说，pair 可以封装任意类型的对象，可以生成各种不同的 `std::pair<T1, T2>` 对象，可以是数组对象或者包含 `std::pair<T1,T2>` 的 vector 容器。pair 还可以封装两个序列容器或两个序列容器的指针。

## 1. 引入

被包括在`<utility>`头文件中。

```C++
#include <utility>
```

## 2. 存储方式

`std::pair` 存储两个对象，分别通过 `first` 和 `second` 成员访问。`std::pair` 自动将初始化参数的值赋给 `first` 和 `second`。

## 3. 方法

### (1)构造方法

1. 默认构造函数：创建一个未初始化的 `std::pair` 对象。

    ```cpp
    std::pair<int, std::string> myPair;
    ```

2. 值构造函数：通过传入两个对象初始化 `std::pair`。

    ```cpp
    std::pair<int, std::string> myPair(1, "example");
    ```

3. 工厂函数 `make_pair`：生成 `std::pair` 对象。

    ```cpp
    auto anotherPair = std::make_pair(2, "test");
    ```

### (2)成员访问

1. first：访问或修改 `std::pair` 的第一个元素。
2. second：访问或修改 `std::pair` 的第二个元素。

### (3)比较操作符

1. 相等 (`==`) 和不相等 (`!=`)：比较两个 `std::pair` 对象是否相等。
2. 小于 (`<`) 和大于 (`>`)和小于等于 (`<=`) 和大于等于 (`>=`)：根据 `first` 和 `second` 的字典顺序比较两个 `std::pair` 对象。

### (4)交换

1. swap：交换两个 `std::pair` 对象的值。

    ```cpp
    myPair.swap(anotherPair);
    ```
