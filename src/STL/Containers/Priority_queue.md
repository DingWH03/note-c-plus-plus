# std::priority_queue

`std::priority_queue` 是一个 **容器适配器 (Container Adapter)**，它基于堆 (heap) 实现，能够在对数时间内插入元素，并且始终允许在常数时间内访问到“优先级最高”的元素。

默认情况下，它是一个 **最大堆 (Max Heap)**，即 `top()` 返回容器中的最大元素。通过自定义比较器，可以变成最小堆或实现任意的优先级规则。

典型应用场景：调度系统、Dijkstra 最短路、Huffman 编码、任务队列等。

## 1. 引入

```c++
#include <queue>
```

## 2. 存储方式

`std::priority_queue` 内部使用一个 **底层容器 (Underlying Container)** 和一个 **比较器 (Compare)** 来实现堆的功能。

### 模板定义

```c++
template<
    class T,
    class Container = std::vector<T>,
    class Compare = std::less<typename Container::value_type>
> class priority_queue;
```

* **T**：存储的元素类型。
* **Container**：底层容器，默认为 `std::vector<T>`。必须支持随机访问迭代器和 `push_back()`/`pop_back()`。
* **Compare**：比较器，默认为 `std::less<T>`（大顶堆）。若改为 `std::greater<T>`，则变为小顶堆。

### 默认实现

* **大顶堆**：`Compare = std::less<T>`
* **小顶堆**：`Compare = std::greater<T>`

### 示例：指定底层容器和比较器

```c++
// 默认：大顶堆
std::priority_queue<int> pq1;

// 小顶堆
std::priority_queue<int, std::vector<int>, std::greater<int>> pq2;

// 使用 deque 作为底层容器
std::priority_queue<int, std::deque<int>> pq3;
```

## 3. 方法

`std::priority_queue` 同样不提供迭代器（不能遍历），只能通过专门的接口进行操作。

### (1) 构造方法

1. `priority_queue()`：构造一个空的优先队列。

   ```c++
   std::priority_queue<int> pq;  // 空的大顶堆
   ```

2. `priority_queue(const Compare& comp, const Container& cont)`：用已有容器构造。

   ```c++
   std::vector<int> vec = {1, 5, 3};
   std::priority_queue<int> pq(std::less<int>(), vec);
   ```

3. 复制构造 / 移动构造。

### (2) 大小函数

1. `size_t size() const`：返回元素个数。

   ```c++
   std::priority_queue<int> pq;
   pq.push(10); pq.push(20);
   std::cout << pq.size(); // 输出 2
   ```

2. `bool empty() const`：检查是否为空。

   ```c++
   if (pq.empty()) std::cout << "Empty\n";
   ```

### (3) 元素访问函数

1. `const_reference top() const`：返回堆顶元素（优先级最高的元素），复杂度 O(1)。
   调用前必须确保不为空。

   ```c++
   pq.push(10);
   pq.push(30);
   pq.push(20);
   std::cout << pq.top(); // 输出 30（大顶堆）
   ```

### (4) 增加函数 (插入)

1. `void push(const T& value)`：插入元素到队列，自动保持堆序。复杂度 O(log n)。

   ```c++
   pq.push(15);
   pq.push(40);
   ```

2. `template<class... Args> void emplace(Args&&... args)`：原地构造元素并插入。

   ```c++
   std::priority_queue<std::pair<int,int>> pq;
   pq.emplace(1, 2); // 构造 pair(1,2)
   ```

### (5) 删除函数 (移除)

1. `void pop()`：移除堆顶元素，复杂度 O(log n)。调用前必须非空。

   ```c++
   pq.push(10);
   pq.push(20);
   pq.pop(); // 移除 20（大顶堆）
   ```

### (6) 其他函数

1. `void swap(priority_queue& other)`：交换两个队列的内容。

   ```c++
   std::priority_queue<int> pq1, pq2;
   pq1.push(1); pq2.push(2);
   pq1.swap(pq2);
   ```

## 4. 示例

### 大顶堆

```c++
#include <iostream>
#include <queue>
using namespace std;

int main() {
    priority_queue<int> pq;

    pq.push(10);
    pq.push(5);
    pq.push(20);

    cout << pq.top() << endl; // 20

    pq.pop();
    cout << pq.top() << endl; // 10
}
```

### 小顶堆

```c++
priority_queue<int, vector<int>, greater<int>> min_heap;
min_heap.push(10);
min_heap.push(5);
min_heap.push(20);

cout << min_heap.top() << endl; // 5
```

### 自定义比较器（例如：比较 pair 的第二个元素）

```c++
struct Compare {
    bool operator()(const pair<int,int>& a, const pair<int,int>& b) {
        return a.second > b.second; // 小顶堆，按第二个元素比较
    }
};

priority_queue<pair<int,int>, vector<pair<int,int>>, Compare> pq;
pq.push({1, 10});
pq.push({2, 5});
cout << pq.top().first << endl; // 输出 2，因为 (2,5) 的 second 更小
```
