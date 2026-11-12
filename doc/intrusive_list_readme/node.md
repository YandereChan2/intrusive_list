# `Yc::intrusive_list_node<vptr, T>`

同时存放一个 `T` 与侵入式链接的类型，继承 [`intrusive_list_node_base<vptr>`](node_base.md)：

```C++
template<class vptr, class T>
class intrusive_list_node : public intrusive_list_node_base<vptr>;
```

`T` 的创建、销毁与生命周期管理由调用方负责；_节点_ 本身可以挂到任意
`basic_intrusive_list<vptr>` 上。`T` 放在一个 union 里，因此：

- 可以"先挂链、后构造值"，也可以"先构造值、后挂链"；
- 默认构造 **不会** 构造 `T`，不要求 `T` 可默认构造。

## 成员函数

### 构造函数

```C++
intrusive_list_node() = default;
```

默认构造只建立一个**脱管**的 _节点_；`T` 尚未构造，此时只能挂链 / 摘链，不能访问 `T`。

### `construct_value`

```C++
template<class Alloc, class... Args>
void construct_value(Alloc& alloc, Args&&... args)
    noexcept(std::is_nothrow_constructible_v<T, Args...>);
```

在该 _节点_ 内就地构造 `T`，构造通过 `alloc`（`allocator_traits<Alloc>::construct`）完成。

### `destroy_value`

```C++
template<class Alloc>
void destroy_value(Alloc& alloc)noexcept;
```

析构该 _节点_ 内的 `T`。调用前必须已经 `construct_value` 过。

### `value`

```C++
T& value()noexcept;
const T& value()const noexcept;
```

访问该 _节点_ 内的 `T`。`T` 未构造时行为未定义。

## 生命周期

`T` 的构造 / 析构与"挂链 / 摘链"是分开的两件事：

1. 创建 / 分配该类型的对象（_节点_ 脱管，`T` 未构造）；
2. 用 `construct_value` 构造 `T`；
3. 把 _节点_ 挂到链上（[`insert`](insert.md) / [`push_back`](push_pop.md) 等）；
4. 用 [`erase`](erase.md) / [`clear`](clear.md) / `release()` 摘链，使 _节点_ 重新脱管；
5. 用 `destroy_value` 析构 `T`；
6. 销毁该对象。

若第 6 步之前 _节点_ 仍处于纳管状态，析构会导致未定义行为。

## 复杂度

`construct_value` / `destroy_value` 取决于 `T` 的构造 / 析构函数；`value` 为常数。

## 异常

`construct_value` 可能抛出 `T` 构造函数产生的异常；此时 `T` 未构造，_节点_ 仍可正常销毁。
`destroy_value` 为 `noexcept`。

## 示例

```C++
#include "intrusive_list.h"
#include <cassert>
#include <memory>

int main()
{
    using node = Yc::intrusive_list_node<void*, int>;
    using list = Yc::basic_intrusive_list<>;

    std::allocator<node> alloc;
    node n;                         // 脱管，T 未构造
    n.construct_value(alloc, 42);   // 就地构造 T

    list l;
    l.push_back(n);                 // 挂链（不复制、不分配）
    assert(static_cast<node&>(*l.begin()).value() == 42);

    l.clear();                      // 摘链，n 重新脱管
    n.destroy_value(alloc);         // 析构 T
}
```

## 备注

`Yc::list_without_size` 就用 `intrusive_list_node<void_pointer, T>` 作为元素节点：
分配器分配节点，`construct_value` / `destroy_value` 负责 `T` 的构造 / 析构。
