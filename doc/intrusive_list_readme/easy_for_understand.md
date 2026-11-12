# 侵入式链表相关概念的阐述

本文解释几件事：

- 为什么链表接口会以 _节点_ 为单位，而不是以元素值为单位；
- 哨兵与"指向自身"的链接，如何带来**脱管 / 纳管**两种状态；
- 为什么 `insert` / `splice` 是静态函数，以及"兄弟链"是怎么回事；
- `container_of` 一类的背景，以及 C++ 中更安全的替代方式；
- `vptr` 模板参数的作用：节点指针可以是缀饰指针。

## 从普通链表说起

一般的链表由容器分配链表元素：

```C++
template<class T>
struct list_node
{
    list_node* prev;
    list_node* next;
    T value;
};

template<class T>
class list
{
    list_node<T>* head;
    // ...
};
```

值和链接在同一个由容器分配的对象里。容器决定一切：插入要分配节点，删除要释放节点，
元素被容器"拥有"。`std::list` 就是这种模型。

## 侵入式链表：链接放进用户对象

侵入式链表反过来：**链接放在用户自己的对象里**，链表不分配、不拥有对象。

```C++
struct task
{
    task* prev;
    task* next;
    int id;
};

task a{nullptr, nullptr, 1};
task b{nullptr, nullptr, 2};
```

链表只需要保存一个"表头位置"（本库用哨兵，见下一节）。插入和删除只改 `prev` / `next`：

```C++
// 把 b 插到 a 后面：只改几个链接，不分配
b.prev = &a;
b.next = a.next;
a.next->prev = &b;
a.next = &b;
```

好处：

- 元素地址稳定：链表操作不会搬移元素，指针 / 引用 / 迭代器一直有效；
- 容器不拥有元素：元素的创建和销毁完全由用户决定；
- 删除不需要查找：只要手里有元素，就能 O(1) 摘链；
- 同一个对象可以同时出现在多条链里——前提是每个用途各带一组链接。

限制也很明显：

- 一组链接在同一时刻只能属于一条链；
- 对象被销毁前必须先把它从所有链中摘下；
- 用户要自己管理节点内存（本库的 `intrusive_list_node` 只负责把 `T` 和链接放在一起）。

## 哨兵与自指：脱管 / 纳管

侵入式链表通常不用"空指针"表示表头，而是用一个**哨兵 _节点_** 把链首尾连成环：

```text
        ┌──────────────────────────────┐
        │                              ▼
   [sentinel] <-> [a] <-> [b] <-> [c] ─┘
```

哨兵不属于用户数据，它只提供 `begin()` / `end()` 的位置。这样：

- 空链就是哨兵的 `prev` / `next` 指向自己；
- 插入 / 删除不需要对"表头"和"中间位置"写两套代码；
- `end()` 始终是同一个位置（哨兵）。

判断一个 _节点_ 是否在链上的方式也随之统一：看它的链接是否指向自己。

- **脱管**：`prev` / `next` 都指向自身。默认状态，表示它不属于任何链。
- **纳管**：链接由链表操作维护，指向前后邻居。

本库的 [`intrusive_list_node_base`](node_base.md) 提供两个操作：

- `is_released()`：是否脱管（`prev` 指向自身）；
- `release()`：脱管——把前后邻居接起来，再让自身的链接指向自己。

`release()` 是幂等的：脱管的 _节点_ 再 `release()` 什么也不做。这让"摘链"可以统一写成
`release()`，而不必先判断它在不在链上。

## 为什么接口以 _节点_ 为单位

`std::list<T>` 的接口以值为单位（`push_back(value)`、`insert(pos, value)`），因为容器
负责分配节点、构造值。

侵入式链表不拥有对象，**不能凭空创建元素**，所以它只接受"已经存在、处于脱管状态的 _节点_"：

```C++
Yc::basic_intrusive_list<> l;
task a{1};
l.push_back(a);        // 不复制、不分配，直接把 a 挂上去
```

同理，删除只把 _节点_ 摘下，不销毁它：

```C++
l.erase(l.begin());    // a 脱管，但 a 仍然存在
```

要销毁元素，由用户（或拥有型容器）在摘链之后自己做。

## 为什么 `insert` / `splice` 是静态函数

链表对象唯一的状态是哨兵。而"在 `pos` 之前插入一个 _节点_"这件事，只用到 `pos` 所指 _节点_
的 `prev` / `next`，**不需要链对象**。

于是 `insert` / 单点 `splice` / 区间 `splice` 都写成静态函数：

```C++
static iterator insert(iterator pos, node_type& node)noexcept;
```

这样它们可以作用于"任意一条兄弟链"——只要 `pos` 是某条链上的位置，哪怕那条链的表头
嵌在别的对象里（例如 [`Yc::trees`](https://github.com/YandereChan2/trees) 把每个节点的
孩子链头存成该节点的一个成员），插入照样成立。

```C++
// 把 b 挂到"a 的孩子链"里：目标链的表头是某个节点成员，而不是一个独立链表对象
list::insert(a.children.begin(), b);
```

## `container_of` 一类的背景

链表算法手里通常只有"链接"的地址。要从"链接的地址"反推"宿主对象地址"，内核里常用
`container_of`：

```C
#define container_of(ptr, type, member) \
    ((type*)((char*)(ptr) - offsetof(type, member)))
```

本库不使用手写偏移，而是利用"继承 + `static_cast`"：

```C++
struct task : Yc::intrusive_list_node_base<>
{
    int id;
};

task t{1};
Yc::basic_intrusive_list<> l;
l.push_back(t);

auto& n = *l.begin();                     // node_type&
auto& same = static_cast<task&>(n);       // 回到完整对象
assert(same.id == 1);
```

这和 `container_of` 的目的相同：从"链接/基类子对象"回到"完整用户对象"。区别是：

- 不需要手写偏移；
- 不需要 `reinterpret_cast`；
- 由 C++ 的基类到派生类转换规则保证正确性（基类必须明确且非虚）。

## `vptr`：节点指针的表示

模板参数 `vptr` 指定侵入式链接里"指向 _节点_ 的指针"用哪种类型。默认 `void*`：

```C++
Yc::basic_intrusive_list<>            // 节点指针是 intrusive_list_node_base<>*
Yc::basic_intrusive_list<fancy_ptr<void>>  // 节点指针是 fancy_ptr<intrusive_list_node_base<fancy_ptr<void>>>
```

内部所有链接都通过 `std::pointer_traits` 完成：`rebind` 得到节点指针类型，
`pointer_to` 从引用得到指针，取地址用 `std::to_address`。因此把 `vptr` 换成缀饰指针
（fancy pointer，例如"存相对偏移的指针"）时，链表的结构、迭代器会整体换成缀饰指针，
不需要改算法。

对使用者来说，通常保持默认即可；只有分配器使用缀饰指针时（例如
[`Yc::list_without_size`](https://github.com/YandereChan2/list_without_size) 会从
`Alloc::void_pointer` 推导 `vptr`）才需要关心它。

## 与 `std::list` 的关系

本库是**底层设施**：不分配、不拥有、不构造元素，接口以 _节点_ 为单位。

按 `std::list` 接口工作、拥有元素的容器是
[`Yc::list_without_size`](https://github.com/YandereChan2/list_without_size)：

- 它内部持有一条 `basic_intrusive_list`；
- 每个元素放在一个 `intrusive_list_node<vptr, T>` 里，由分配器管理生命周期；
- 它把节点迭代器适配成 `T` 的迭代器，把节点上的比较器适配成 `T` 上的比较器；
- 它不维护元素个数，因此没有 `size()`。

## 小结

- 侵入式链表把链接放进用户对象，链表不拥有元素；插入 / 删除 / 拼接只改指针。
- 哨兵让空链和非空链、表头和非表头使用同一套位置表示；"指向自身"表示脱管。
- `release()` 幂等，是"摘链"的统一操作；`insert` / `splice` 静态化后可以作用于任意兄弟链。
- `container_of` 从成员地址反推宿主；本库用"继承 + `static_cast`"完成同样的事情。
- `vptr` 决定节点指针的表示，普通指针与缀饰指针走同一套 `pointer_traits` 代码。
