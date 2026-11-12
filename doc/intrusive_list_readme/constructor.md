# `Yc::basic_intrusive_list<vptr>::basic_intrusive_list`

```C++
basic_intrusive_list() = default;
```

只有默认构造。构造出来的链表是**空链**：哨兵 _节点_ 的 `prev` / `next` 都指向自身。

链表对象本身只有一个哨兵，没有任何其他状态。

## 不可复制、不可移动

```C++
basic_intrusive_list(const basic_intrusive_list&) = delete;
basic_intrusive_list& operator=(const basic_intrusive_list&) = delete;
```

复制与移动都被隐式删除。原因是哨兵是**自指**的：链表里每个 _节点_ 的 `end()` 都指向这个
哨兵的地址。按值复制 / 移动会得到一个"新哨兵"，但 _节点_ 里的链接仍指向旧哨兵，链表结构
会静默损坏。

要转移内容，使用 [`splice`](splice.md)：

```C++
Yc::basic_intrusive_list<> a, b;
// ... 往 a 里挂节点 ...
b.splice(b.begin(), a);   // a 的节点整体搬到 b，a 变空
```

要把一条链复制到另一条链，需要自己分配新 _节点_（或使用拥有型容器
[`Yc::list_without_size`](https://github.com/YandereChan2/list_without_size)）。

## 复杂度

常数。

## 异常

（无）
