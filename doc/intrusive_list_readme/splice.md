# `Yc::basic_intrusive_list<vptr>::splice`

```C++
void splice(iterator pos, basic_intrusive_list& other);
void splice(iterator pos, basic_intrusive_list&& other);

static void splice(iterator pos, basic_intrusive_list&, iterator it);
static void splice(iterator pos, basic_intrusive_list&&, iterator it);

static void splice(iterator pos, basic_intrusive_list&, iterator first, iterator last);
static void splice(iterator pos, basic_intrusive_list&&, iterator first, iterator last);
```

把 `other` 的 _节点_ 搬到 `pos` 之前：

- 整条链版本：`other` 的全部 _节点_ 搬到 `pos` 前，`other` 变为空链；
- 单点版本：把 `it` 所指的一个 _节点_ 搬到 `pos` 前；
- 区间版本：把 `[first, last)` 中的 _节点_ 搬到 `pos` 前。

所有版本都只改链接：**不分配、不复制、不搬移元素**，_节点_ 地址与迭代器保持不变。

## 语义细节

- `pos` 是 `*this` 上的有效位置（可以是 `end()`）；
- 整条链版本：`other` 为空、或 `&other == *this` 时无效果；
- 单点版本：`pos == it`、或 `pos == ++it`（插到自己后面）时无效果；
- 区间版本：`first == last`、或 `pos == first` 时无效果；`pos` 不能落在 `[first, last)` 内；
- 静态版本不需要 `other` 参数里的链对象，只需要 `pos` / `it` / `first` / `last` 的位置，
  因此可以作用于"表头嵌在别的对象里"的兄弟链——参数里的 `other` 只是为了让签名与
  `std::list::splice` 一致，实现不使用它。

## 复杂度

- 整条链 / 单点：常数；
- 区间：常数（双向迭代器可以 O(1) 找到 `last` 的前一个，不需要数元素）。

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

    task a{1}, b{2}, c{3}, d{4};
    list x, y;
    x.push_back(a);
    x.push_back(b);
    y.push_back(c);
    y.push_back(d);

    // 整条链：y 的节点搬到 x 尾部
    x.splice(x.end(), y);
    assert(y.empty());
    assert(static_cast<task&>(x.back()).id == 4);

    // 单点：把最后一个节点搬到表头
    auto last = x.end();
    --last;
    list::splice(x.begin(), x, last);
    assert(static_cast<task&>(x.front()).id == 4);

    // 区间：把 [第一个, 第二个) 搬到尾部
    auto first = x.begin();
    auto second = first;
    ++second;
    list::splice(x.end(), x, first, second);   // 4 移到尾部：1, 2, 3, 4
    assert(static_cast<task&>(x.back()).id == 4);

    x.clear();
}
```
