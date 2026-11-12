# `Yc::basic_intrusive_list<vptr>::push_front` , `push_back` , `pop_front` , `pop_back`

```C++
void push_front(node_type& node);
void push_back(node_type& node);
void pop_front();
void pop_back();
```

- `push_front` 等价于 `insert(begin(), node)`，挂到表头；
- `push_back` 等价于 `insert(end(), node)`，挂到表尾；
- `pop_front` 等价于 `erase(begin())`，摘下首 _节点_；
- `pop_back` 等价于 `erase(--end())`，摘下尾 _节点_。

四个操作都只改链接：不分配、不搬移元素、不销毁 _节点_。`pop_*` 只是让 _节点_ 脱管。

空链上调用 `pop_front` / `pop_back` 是未定义行为。

## 复杂度

均为常数。

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

    task a{1}, b{2}, c{3};
    list l;

    l.push_back(a);
    l.push_front(b);
    l.push_back(c);
    // 链：b, a, c

    assert(static_cast<task&>(l.front()).id == 2);
    assert(static_cast<task&>(l.back()).id == 3);

    l.pop_front();                  // b 脱管
    l.pop_back();                   // c 脱管
    assert(static_cast<task&>(l.front()).id == 1);
    assert(b.is_released() && c.is_released());

    l.clear();
}
```
