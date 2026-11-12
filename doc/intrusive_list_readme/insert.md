# `Yc::basic_intrusive_list<vptr>::insert`

```C++
static iterator insert(iterator pos, node_type& node)noexcept;
```

在 `pos` 之前挂接 `node`，返回指向 `node` 的迭代器。

- `insert` 先对 `node` 调用 `release()`：如果它已经在某条链上，会先被摘下、再插到 `pos`
  之前（效果等同于一次单点 [`splice`](splice.md)）；
- `pos` 可以是 `end()`，此时挂到表尾；
- 静态函数：挂接只需要 `pos` 所指 _节点_ 的链接，不需要链对象本身，因此 `pos` 可以来自任意
  一条"兄弟链"（包括表头嵌在别的对象成员里的链）；
- 不分配内存、不搬移元素，也不会使其他迭代器失效。

## 参数

|||
|-:|-|
|`pos`|插入位置；新 _节点_ 会插在它之前|
|`node`|要挂接的 _节点_|

## 返回值

指向 `node` 的迭代器。

## 前置条件

- `pos` 必须是某条链上的有效位置（不能是"未初始化"的迭代器）；
- 不要把 `pos` 传成指向 `node` 自身的位置：`insert` 会先摘下 `node`，结果是把 `node` 从链上
  移除并保持脱管，而不是插回原处。正常代码不应这样调用。

## 复杂度

常数。

## 异常

（无）

## 示例

```C++
#include "intrusive_list.h"
#include <cassert>

struct task : Yc::intrusive_list_node_base<>
{
    int id;
    explicit task(int i) : id(i) {}
};

int main()
{
    using list = Yc::basic_intrusive_list<>;

    task a{1}, b{2};
    list l;
    l.push_back(a);

    auto it = list::insert(l.begin(), b);        // b 插到 a 之前
    assert(static_cast<task&>(*it).id == 2);
    assert(static_cast<task&>(l.front()).id == 2);

    list other;
    task c{3};
    other.push_back(c);

    // 跨链插入：c 会被先从 other 摘下，再挂到 l 的表尾
    list::insert(l.end(), c);
    assert(other.empty());
    assert(static_cast<task&>(l.back()).id == 3);

    l.clear();
}
```

## 备注

- 与 `std::list::insert` 不同：这里不构造元素，只挂接已经存在的 _节点_；
- 因为 `insert` 会先 `release()`，所以"把一条链上的 _节点_ 插到另一条链"不需要事先手动摘链。
