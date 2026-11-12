# `Yc::intrusive_list_node_base<vptr>`

侵入式链表的 _节点_ 类型。用户类型通过继承它，使自己可以挂到链上：

```C++
struct task : Yc::intrusive_list_node_base<>
{
    int id;
    explicit task(int i) : id(i) {}
};
```

通常不需要直接操作它的任何成员；导航请使用 [迭代器](iterators.md)。

## 脱管与纳管

每个 _节点_ 有两种状态：

- **脱管**：默认状态。`prev` / `next` 都指向自身，表示它不属于任何链；
- **纳管**：已经挂到某条链上，链接由链表操作维护。

链接字段是私有的，只有 `basic_intrusive_list` 和迭代器能读写。节点自己只暴露
[`is_released`](#is_released) 与 [`release`](#release)。

## 构造与复制

```C++
intrusive_list_node_base() = default;
intrusive_list_node_base(const intrusive_list_node_base&) = delete;
intrusive_list_node_base& operator=(const intrusive_list_node_base&) = delete;
```

- 默认构造得到**脱管**的 _节点_；
- 不可复制。链接是自指的，逐位复制会让副本的链接仍指向源对象，链表结构会静默损坏。
  节点本来就是就地构造、挂在链里的对象，不支持搬迁语义。

## `is_released`

```C++
bool is_released()const noexcept;
```

是否处于脱管状态。常数时间。

## `release`

```C++
void release()noexcept;
```

主动脱管：把前后邻居接起来，再让 `prev` / `next` 指向自身。

- 已脱管时无效果（幂等）；
- 只改指针：不销毁 _节点_，也不分配内存；
- 摘链后其他 _节点_ 与迭代器仍然有效。

## 析构

- 脱管的 _节点_ 可以正常析构；
- 纳管的 _节点_ 析构会让链上的邻居 / 哨兵留下悬空链接，因此是未定义行为。
  析构前应先 `release()`，或通过 [`erase`](erase.md) / [`clear`](clear.md) 摘链。
- 反过来，析构**链表对象**不会销毁任何 _节点_。

## 复杂度

`is_released` / `release`：常数。

## 异常

（无）
