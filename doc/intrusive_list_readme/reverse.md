# `Yc::basic_intrusive_list<vptr>::reverse`

```C++
void reverse()noexcept;
```

把整条链逆序。

实现是把每个 _节点_ 的 `prev` / `next` 对调，最后对调哨兵的：

```C++
node_type* p = std::to_address(sentinel.next);
while (p != std::addressof(sentinel))
{
    node_type* next = std::to_address(p->next);   // 对调前的 next
    node_ptr prev = p->prev;
    p->prev = p->next;
    p->next = prev;
    p = next;
}
node_ptr prev = sentinel.prev;
sentinel.prev = sentinel.next;
sentinel.next = prev;
```

- 只改指针：不分配、不搬移元素；
- _节点_ 地址不变，所以迭代器仍然有效，仍指向同一个元素（只是位置反了）；
- 空链 / 单 _节点_ 无效果。

## 复杂度

`O(n)`。

## 异常

（无）—— `noexcept`。

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

    task a{1}, b{2}, c{3};
    list l;
    l.push_back(a);
    l.push_back(b);
    l.push_back(c);

    auto keep = l.begin();          // 指向 a
    l.reverse();
    assert(static_cast<task&>(l.front()).id == 3);
    assert(static_cast<task&>(*keep).id == 1);   // 迭代器仍指向同一个元素
    auto after = keep;
    ++after;
    assert(after == l.end());       // a 现在在表尾

    l.reverse();
    assert(static_cast<task&>(l.front()).id == 1);

    l.clear();
}
```
