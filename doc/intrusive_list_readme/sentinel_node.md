# `Yc::basic_intrusive_list<vptr>::sentinel_node`

```C++
node_type& sentinel_node()noexcept;
const node_type& sentinel_node()const noexcept;
```

返回链表自带的哨兵 _节点_ 的引用。

- [`end()`](iterators.md) 就指向它；
- 链上最后一个 _节点_ 的 `next` 也指向它；
- 空链时，它的 `prev` / `next` 都指向自身。

它提供了一种"只拿位置、不拿链对象"的写法。因为 [`insert`](insert.md) / [`splice`](splice.md)
的静态重载只需要一个位置，可以从哨兵构造出 `end()`：

```C++
Yc::basic_intrusive_list<> l;
task a{1};
l.push_back(a);

Yc::basic_intrusive_list<>::iterator e{ &l.sentinel_node() };
assert(e == l.end());
```

在 [`Yc::trees`](https://github.com/YandereChan2/trees) 这类结构里，孩子链的表头直接就是
某个节点的成员，`sentinel_node` 让"外部链"和"成员链"可以使用同一套位置表示。

## 注意

- 不要把哨兵当作链上的元素：它没有用户数据，也不属于任何链；
- 不要试图修改它的链接（链接字段是私有的，也无法直接改）；
- 链表对象销毁后，这个引用随之失效。

## 复杂度

常数。

## 异常

（无）
