# `Yc::basic_intrusive_list<vptr>::sort`

```C++
template<class Compare>
void sort(Compare comp);
```

按 `comp` 对整条链做**稳定**排序。

- 自底向上归并排序：`O(n log n)`、不分配内存、不搬移元素；
- 相等元素保持原有相对顺序；
- 排序只改 `prev` / `next`，_节点_ 地址不变，迭代器仍然指向同一个元素；
- 空链 / 单 _节点_ 直接返回。

## 参数

|||
|-:|-|
|`comp`|比较器，接收两个 `const node_type&`（_节点_ 基类）；相等返回 `false`|

用户数据需要自己转换：

```C++
l.sort([](const list::node_type& x, const list::node_type& y)
    {
        return static_cast<const task&>(x).id < static_cast<const task&>(y).id;
    });
```

## 返回值

（无）

## 复杂度

`O(n log n)` 次比较，`O(1)` 额外空间（固定大小的临时链数组，与 `n` 无关）。

## 异常

比较器抛异常时，已拆到临时链上的 _节点_ 会被全部拼回 `*this`：
链表仍然完整（顺序不保证），不会丢 _节点_，也不会泄漏。

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

    task a{3}, b{1}, c{2}, d{1};
    list l;
    l.push_back(a);
    l.push_back(b);
    l.push_back(c);
    l.push_back(d);

    l.sort([](const list::node_type& x, const list::node_type& y)
        {
            return static_cast<const task&>(x).id < static_cast<const task&>(y).id;
        });

    // 稳定：两个 id == 1 的 b、d 保持原有先后
    assert(static_cast<task&>(l.front()).id == 1);
    assert(&*l.begin() == static_cast<list::node_type*>(&b));
    auto it = l.begin();
    ++it;
    assert(&*it == static_cast<list::node_type*>(&d));

    l.clear();
}
```
