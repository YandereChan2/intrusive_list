# `Yc::basic_intrusive_list<vptr>::remove_if` , `unique`

```C++
template<class Pred, class Deleter>
std::size_t remove_if(Pred pred, Deleter del);

template<class BinaryPred, class Deleter>
std::size_t unique(BinaryPred pred, Deleter del);
```

按谓词删除 _节点_，并返回实际删除的数量。

- `remove_if`：删除所有使 `pred(node)` 为真的 _节点_；
- `unique`：对相邻的两个 _节点_，若 `pred(前一个, 后一个)` 为真，删除后一个；
  于是"连续相等"的一段只保留第一个。

两个函数是**侵入式**的：它们不销毁 _节点_，而是把 _节点_ 摘下来之后交给回收器 `del`。

## 删除协议

对每一个被删除的 _节点_：

1. 先从链上摘下（`release()`），此时它已脱管；
2. 调用 `del(node)`，`node` 是 `node_type&`；
3. 由 `del` 负责销毁 / 释放这个 _节点_（例如 `delete static_cast<task*>(&node)`）。

因此 `del` 里可以安全地转换回用户类型并销毁对象；但**不能**再把该 _节点_ 挂回任何链。

## 参数

|||
|-:|-|
|`pred`|`remove_if`：一元谓词，接收 `const node_type&`<br>`unique`：二元谓词，接收两个 `const node_type&`|
|`del`|回收器，接收脱管后的 `node_type&`|

## 返回值

删除的 _节点_ 数量。

## 复杂度

`O(n)`。

## 异常

- `pred` 抛异常：链表保持完整，已删除的 _节点_ 已经交给 `del`，未删除的仍在链上；
- `del` 抛异常：该 _节点_ 此时已经脱管，不会被重新挂回；异常继续向外传播。

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

    list l;
    l.push_back(*new task(1));
    l.push_back(*new task(2));
    l.push_back(*new task(2));
    l.push_back(*new task(3));

    auto del = [](list::node_type& n) { delete static_cast<task*>(&n); };
    auto same_id = [](const list::node_type& x, const list::node_type& y)
    {
        return static_cast<const task&>(x).id == static_cast<const task&>(y).id;
    };

    assert(l.unique(same_id, del) == 1);     // 1, 2, 3
    assert(l.remove_if(
        [](const list::node_type& n)
        {
            return static_cast<const task&>(n).id == 2;
        },
        del) == 1);                          // 1, 3

    while (!l.empty())
    {
        list::node_type& n = *l.begin();
        l.erase(l.begin());
        del(n);
    }
}
```

## 备注

- 与 `std::list::remove_if` / `unique` 不同：这里不析构元素，删除后的 _节点_ 交给调用方的
  回收器处理；
- `unique` 只比较相邻元素，要先去重请先 [`sort`](sort.md)；
- 若只想要"摘下来但保留 _节点_"，可以自己遍历并调用 [`erase`](erase.md)，不需要回收器。
