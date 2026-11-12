# Yc::intrusive_list

在文件 [`"intrusive_list.h"`](../../intrusive_list.h) 中定义。提供一套侵入式双向链表设施：

```C++
template<class vptr = void*>
class basic_intrusive_list;

template<class vptr = void*>
class basic_intrusive_list_iterator;

template<class vptr = void*>
class basic_intrusive_list_const_iterator;

template<class vptr = void*>
class intrusive_list_node_base;

template<class vptr, class T>
class intrusive_list_node;
```

_链表_ 不拥有任何 _节点_：节点的创建、析构由用户负责，链表只维护链接。
每个 _节点_ 自带 `prev` / `next`，因此插入、摘除、拼接、排序、合并、逆序都只是改指针，
不分配内存、不搬移元素，元素地址在链表操作前后保持不变。

> 本头文件只提供结构性的链表设施。拥有元素、接口与 `std::list` 一致的容器是
> [`Yc::list_without_size`](https://github.com/YandereChan2/list_without_size)，
> 它内部就持有一条 `basic_intrusive_list`。

如果对侵入式链接、哨兵、`container_of` 一类的背景不熟悉，可以先阅读
[概念说明](easy_for_understand.md)。

## 成员类型

`basic_intrusive_list<vptr>` 的成员类型：

|成员类型|定义|
|-:|:-|
|`node_type`|[`intrusive_list_node_base<vptr>`](node_base.md)；用户类型继承它|
|`node_ptr`|`pointer_traits<vptr>::rebind<node_type>`，节点指针（可能是缀饰指针）|
|`const_node_ptr`|`pointer_traits<vptr>::rebind<const node_type>`|
|[`iterator`](iterators.md)|可修改的双向迭代器，解引用得到 `node_type&`|
|[`const_iterator`](iterators.md)|只读的双向迭代器，解引用得到 `const node_type&`|

另外两个带 `T` / 不带 `T` 的节点类型：

|类型|定义|
|-:|:-|
|[`intrusive_list_node_base<vptr>`](node_base.md)|所有 _节点_ 的公共基类，提供 `prev` / `next` 与 `is_released` / `release`|
|[`intrusive_list_node<vptr, T>`](node.md)|在前者基础上就地存放一个 `T`（`construct_value` / `destroy_value`）|

## 成员函数

| | |
|:-|:-|
|[（构造函数）](constructor.md)|只有默认构造；链表不可复制、不可移动|
|[（析构函数）](destructor.md)|析构链表对象，不销毁 _节点_|
|[`operator=`](operator_assign.md)|被隐式删除|
|[`empty`](empty.md)|是否为空|
|[`clear`](clear.md)|摘掉所有 _节点_，让它们重新脱管|

### 迭代器与访问

| | |
|:-|:-|
|[`begin` / `end`](iterators.md)|首 _节点_ 与哨兵位置（含 const 版本）|
|[`iterator` / `const_iterator`](iterators.md)|节点迭代器|
|[`sentinel_node`](sentinel_node.md)|链表自带的哨兵 _节点_|
|[`front` / `back`](front_back.md)|首 / 尾 _节点_|

### 修改器

| | |
|:-|:-|
|[`insert`](insert.md)|把脱管的 _节点_ 挂到某位置之前（静态）|
|[`erase`](erase.md)|摘下某 _节点_ 并返回下一个位置|
|[`push_front` / `push_back`](push_pop.md)|挂到表头 / 表尾|
|[`pop_front` / `pop_back`](push_pop.md)|摘下首 / 尾 _节点_（只脱管，不销毁）|
|[`splice`](splice.md)|搬移整条链 / 单个 _节点_ / 一段 _节点_（含静态重载）|

### 操作

| | |
|:-|:-|
|[`sort`](sort.md)|稳定归并排序，比较器作用于 _节点_|
|[`merge`](merge.md)|稳定合并两条有序链|
|[`reverse`](reverse.md)|整条链逆序|
|[`remove_if` / `unique`](remove_unique.md)|按谓词删除 / 去重，删除后的 _节点_ 交给回收器|

## 模板参数

|||
|-:|:-|
|`vptr`|指定侵入式链接使用的"void 指针"类型。默认 `void*`；使用缀饰指针（fancy pointer）的分配器可传对应的 `fancy<void>`，_节点_、哨兵、迭代器会整体换成缀饰指针。|
|`T`|仅用于带数据的类型 `intrusive_list_node<vptr, T>`。|

## 生命周期

- 每个 _节点_ 默认处于**脱管**状态：`prev` / `next` 都指向自身。
- [`insert`](insert.md) / [`push_front`](push_pop.md) / [`push_back`](push_pop.md) 把 _节点_
  纳入某条链；此时它处于**纳管**状态。
- [`release`](node_base.md) 让 _节点_ 自己脱管（把前后邻居接起来，再让自身链接指向自己）。
  [`erase`](erase.md) / [`pop_front`](push_pop.md) / [`pop_back`](push_pop.md) /
  [`clear`](clear.md) 内部都会调用它。
- 链表对象只持有一个哨兵 _节点_。**析构链表对象不会销毁任何 _节点_**；如果析构时链上还有
  _节点_，它们的链接仍指向这个已销毁的哨兵，因此是未定义行为。析构前应先 [`clear`](clear.md)。
- 纳管的 _节点_ 在析构前必须先脱管，否则其他 _节点_ 会留下指向它的链接，同样是未定义行为。
- 拼接（[`splice`](splice.md)）只是把 _节点_ 从一条链转到另一条链，_节点_ 的地址、迭代器
  始终不变。

## 复杂度总览

- 常数：`insert` / `erase` / `push_*` / `pop_*` / `splice`（全部重载）/ `front` / `back` /
  `empty` / `sentinel_node` / `is_released` / `release`。
- 线性：`clear` / `reverse` / `remove_if` / `unique` / `merge`（与涉及的 _节点_ 数量成线性）。
- `sort`：`O(n log n)`，不分配内存。

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

    // 解引用得到 node_type&，再 static_cast 到自己的类型
    assert(static_cast<task&>(l.front()).id == 1);
    assert(static_cast<task&>(*l.begin()).id == 1);

    // 把 b 移到表头
    auto it = l.begin();
    ++it;                       // 指向 b
    list::splice(l.begin(), l, it);
    assert(static_cast<task&>(l.front()).id == 2);

    l.sort([](const list::node_type& x, const list::node_type& y)
        {
            return static_cast<const task&>(x).id < static_cast<const task&>(y).id;
        });
    assert(static_cast<task&>(l.front()).id == 1);

    l.reverse();
    assert(static_cast<task&>(l.front()).id == 3);

    l.clear();                  // 只脱管，a / b / c 仍然存活
    assert(a.is_released() && b.is_released() && c.is_released());
}
```

## 与拥有型容器的关系

- [`Yc::list_without_size`](https://github.com/YandereChan2/list_without_size)：
  拥有元素的 `std::list` 风格容器，内部持有一条 `basic_intrusive_list`，
  负责节点分配与 `T` 的构造 / 析构。
- [`Yc::trees`](https://github.com/YandereChan2/trees)：森林容器把"孩子链"表示为
  `basic_intrusive_list`，把节点挂在根链 / 孩子链上。

## 自检

[`test_intrusive_list.cpp`](../../test_intrusive_list.cpp) 覆盖了排序稳定性、合并、
删除 / 去重与回收器、`reverse` 后的迭代器稳定性、比较器抛异常时节点不丢、
空链 / 单节点等情形。
