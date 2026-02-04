这是一个很经典的C++性能优化技巧，涉及到 `unordered_set`（以及所有基于节点的容器如 `map`、`set`）的内部实现原理。

## 根本原因：内存管理方式不同

### 1. **`clear()` 的工作原理**
```cpp
void clear() noexcept {
    // 1. 遍历所有桶，释放每个节点（元素）
    // 2. 但通常保留桶数组本身的内存不变
    // 3. 将每个桶的指针设为 nullptr
}
```
- **时间复杂度**：O(n)，需要遍历并释放每个元素节点
- **内存处理**：只释放元素节点内存，**不释放桶数组**（bucket array）
- **操作密集**：对每个节点调用析构函数，需要遍历整个数据结构

### 2. **`swap()` + 空容器 的工作原理**
```cpp
unordered_set<int> myset = {1, 2, 3, 4, 5};
// 快速清空：
unordered_set<int>().swap(myset);
```
- **swap 操作**：O(1)，只交换内部指针
- **实际清理**：临时空对象接管原容器内存，临时对象析构时一次性释放
- **内存处理**：**完全释放所有内存**（包括桶数组和元素节点）

## 关键差异对比

| 操作 | 时间复杂度 | 内存释放 | 适合场景 |
|------|-----------|----------|----------|
| `clear()` | O(n) - 线性 | 只释放元素，保留桶 | 后续还要添加大量元素 |
| `swap()` | O(1) - 常数 | 完全释放所有内存 | 容器不再使用或重置 |

## 底层实现示例

```cpp
// 简化的 unordered_set 结构
template<typename T>
class unordered_set {
    struct Node { T value; Node* next; };
    
    Node** buckets;     // 桶数组指针
    size_t bucket_count; // 桶数量
    size_t element_count;// 元素数量
};
```

**`clear()` 的实际操作：**
```cpp
for(size_t i = 0; i < bucket_count; ++i) {
    Node* current = buckets[i];
    while(current) {
        Node* next = current->next;
        delete current;  // 释放每个节点
        current = next;
    }
    buckets[i] = nullptr;
}
element_count = 0;
// bucket_count 不变，buckets 数组不释放
```

**`swap()` 的实际操作：**
```cpp
// 基本就是交换几个指针
std::swap(this->buckets, other.buckets);
std::swap(this->bucket_count, other.bucket_count);
std::swap(this->element_count, other.element_count);
```

## 实际测试示例

```cpp
#include <iostream>
#include <unordered_set>
#include <chrono>

void test_clear() {
    std::unordered_set<int> s;
    for(int i = 0; i < 1000000; ++i) {
        s.insert(i);
    }
    
    auto start = std::chrono::high_resolution_clock::now();
    s.clear();  // 慢
    auto end = std::chrono::high_resolution_clock::now();
    std::cout << "clear() took: " 
              << std::chrono::duration_cast<std::chrono::milliseconds>(end-start).count()
              << " ms" << std::endl;
}

void test_swap() {
    std::unordered_set<int> s;
    for(int i = 0; i < 1000000; ++i) {
        s.insert(i);
    }
    
    auto start = std::chrono::high_resolution_clock::now();
    std::unordered_set<int>().swap(s);  // 快
    auto end = std::chrono::high_resolution_clock::now();
    std::cout << "swap() took: " 
              << std::chrono::duration_cast<std::chrono::milliseconds>(end-start).count()
              << " ms" << std::endl;
}
```

## 什么时候该用哪种方法？

### 用 `clear()` 的情况：
```cpp
// 需要重用容器，后面还要添加很多元素
std::unordered_set<int> cache;
while(processing) {
    cache.clear();  // 保留桶容量，后续插入更快
    // ... 重新填充 cache
}
```

### 用 `swap()` 的情况：
```cpp
// 容器不再使用或需要完全重置
std::unordered_set<int> temp_set;
// ... 使用 temp_set
temp_set.clear();  // 先清空元素

// 或者完全释放内存：
std::unordered_set<int>().swap(temp_set);  // 彻底释放
```

## 其他类似容器

这个技巧同样适用于：
- `std::unordered_map`
- `std::unordered_multiset`
- `std::unordered_multimap`
- `std::set` / `std::multiset`
- `std::map` / `std::multimap`

**注意**：对于 `std::vector` 等连续内存容器，情况相反：
- `vector::clear()` 很快（只是修改 size）
- `vector::swap()` 也会释放内存，但可能比 clear() 慢

## 现代 C++ 的替代方案

C++11 之后，也可以使用：
```cpp
// 方法1：shrink_to_fit 配合 clear
myset.clear();
myset.rehash(0);  // 释放桶内存

// 方法2：直接赋值为新容器（C++11）
myset = std::unordered_set<int>();  // 移动赋值
```

**总结**：`swap()` 快是因为它把内存释放工作推迟到临时对象析构，而 `clear()` 必须立即完成所有清理工作并保留桶结构。选择哪种方式取决于你是否需要保留容器的容量以备后续使用。