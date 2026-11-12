# `Yc::basic_intrusive_list<vptr>::iterator` , `const_iterator`

```C++
using iterator = basic_intrusive_list_iterator<vptr>;
using const_iterator = basic_intrusive_list_const_iterator<vptr>;
```

双向迭代器，解引用得到 _节点_（`node_type&` / `const node_type&`），而不是用户数据。
用户数据要从 _节点_ 转换回自己的类型：

```C++
auto& n = *l.begin();                     // node_type&
auto& t = static_cast<task&>(n);          // 回到完整的 task
```

## 迭代器接口

|成员|说明|
|:-|:-|
|`iterator() = default`|默认构造；与任何迭代器的比较结果取决于底层指针是否为空|
|`explicit iterator(node_ptr)`|从节点指针构造（一般不需要直接用）|
|`base()`|返回底层节点指针 `node_ptr`（可能是缀饰指针）|
|`operator*`|`node_type&` / `const node_type&`|
|`operator->`|`node_type*` / `const node_type*`|
|`operator++` / `operator--`|前置、后置，常数时间|
|`operator==`|同类型比较；`iterator` 与 `const_iterator` 之间可以先隐式转换再比较|
|`operator const_iterator()`|`iterator` → `const_iterator` 的隐式转换|

## `begin` / `end`

```C++
iterator begin()noexcept;
iterator end()noexcept;
const_iterator begin()const noexcept;
const_iterator end()const noexcept;
```

- `begin()` 指向首 _节点_；空链时等于 `end()`；
- `end()` 指向链表自带的哨兵（[`sentinel_node`](sentinel_node.md)）；
- const 版本返回 `const_iterator`，只读。

## 失效规则

迭代器只保存一个节点指针，因此：

- 插入 / 摘除**其他** _节点_ 不会使迭代器失效；
- 只要 _节点_ 还活着（在链里或脱管），迭代器就有效；
- [_节点_ 被销毁](node_base.md#析构)后，指向它的迭代器失效；
- [`splice`](splice.md) 之后迭代器仍指向同一个 _节点_（只是它可能已在另一条链上）；
- [`sort`](sort.md) / [`merge`](merge.md) / [`reverse`](reverse.md) 只改链接，迭代器仍指向
  同一个元素，只是它所在的位置变了。

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

    int sum = 0;
    for (auto it = l.begin(); it != l.end(); ++it)
    {
        sum += static_cast<task&>(*it).id;
    }
    assert(sum == 6);

    // const 版本
    const list& cl = l;
    for (auto it = cl.begin(); it != cl.end(); ++it)
    {
        static_assert(std::is_same_v<decltype(*it), const list::node_type&>);
    }

    // iterator -> const_iterator
    list::const_iterator ci = l.begin();
    assert(ci == l.begin());
    assert(l.begin() == ci);

    l.clear();
}
```

## 复杂度

`++` / `--` / `*` / `->` / `==`：常数。

## 异常

（无）
