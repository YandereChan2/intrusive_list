# `Yc::basic_intrusive_list<vptr>::merge`

```C++
template<class Compare>
void merge(basic_intrusive_list& other, Compare comp);

template<class Compare>
void merge(basic_intrusive_list&& other, Compare comp);
```

把**已经按 `comp` 有序**的 `other` 稳定合并进 `*this`；合并后 `other` 变为空链。

- 相等元素优先保留 `*this` 中的（稳定）；
- 只改链接：不分配、不搬移元素、迭代器保持有效；
- `&other == *this` 时无效果；
- 与 [`sort`](sort.md) 一样，比较器接收两个 `const node_type&`。

## 参数

|||
|-:|-|
|`other`|另一条有序链；合并后为空|
|`comp`|比较器，接收两个 `const node_type&`|

## 前置条件

两条链都按 `comp` 排好序；否则结果未定义（不会崩溃，但顺序没有意义）。

## 复杂度

`O(n)`，`n` 为两条链 _节点_ 数之和。

## 异常

比较器抛异常时，已经搬过来的 _节点_ 留在 `*this` 中，其余留在 `other` 中，
两条链都仍然完整。

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

    auto less = [](const list::node_type& x, const list::node_type& y)
    {
        return static_cast<const task&>(x).id < static_cast<const task&>(y).id;
    };

    task a1{1}, a3{3}, b2{2}, b3{3};
    list a, b;
    a.push_back(a1);
    a.push_back(a3);
    b.push_back(b2);
    b.push_back(b3);

    a.merge(b, less);
    assert(b.empty());

    // 1, 2, 3, 3；相等时 a 的节点在前
    auto it = a.begin();
    ++it;
    ++it;
    assert(&*it == static_cast<list::node_type*>(&a3));

    a.clear();
}
```
