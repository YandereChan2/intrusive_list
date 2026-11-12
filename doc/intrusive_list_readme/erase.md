# `Yc::basic_intrusive_list<vptr>::erase`

```C++
iterator erase(iterator pos);
```

摘下 `pos` 所指的 _节点_（内部调用 [`release()`](node_base.md#release)），
返回指向下一个 _节点_ 的迭代器。

- 不销毁 _节点_、不析构用户数据、不释放内存；
- `pos` 必须是有效位置，`pos == end()` 是未定义行为；
- 常数时间；
- 其他迭代器不受影响。

## 参数

|||
|-:|-|
|`pos`|要摘下的 _节点_ 的位置|

## 返回值

下一个位置的迭代器；摘掉的是最后一个 _节点_ 时返回 `end()`。

## 复杂度

常数。

## 异常

（无）—— 不分配，也不抛异常。

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
    l.push_back(b);

    auto it = l.erase(l.begin());                // 摘掉 a
    assert(static_cast<task&>(*it).id == 2);
    assert(a.is_released());                     // a 仍然存活，只是脱管

    // 依次摘完
    for (auto i = l.begin(); i != l.end();)
    {
        i = l.erase(i);
    }
    assert(l.empty());
    assert(b.is_released());
}
```

## 备注

- 与 `std::list::erase` 不同：这里不销毁元素，只改链接；
- 如果摘链后要销毁 _节点_，应先确保它已经脱管（`erase` 已经替你做了）。
