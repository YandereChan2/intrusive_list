# `Yc::basic_intrusive_list<vptr>::clear`

```C++
void clear()noexcept;
```

把链上所有 _节点_ 摘下来，使它们全部**脱管**；链表变为空链。

- 不销毁 _节点_、不析构任何用户数据、不释放内存；
- 复杂度与元素个数成线性关系；
- 之后可以安全地销毁链表对象，或销毁这些 _节点_。

```C++
Yc::basic_intrusive_list<> l;
task a{1}, b{2};
l.push_back(a);
l.push_back(b);

l.clear();
assert(l.empty());
assert(a.is_released());
assert(b.is_released());
```

## 实现说明

不能"边遍历边 `release()`"：`release()` 会改写邻居指针，哨兵的 `next` 又指回链内，
遍历会来回绕圈。实现是每一步先记下下一个 _节点_，再把当前 _节点_ 断开：

```C++
node_type* p = std::to_address(sentinel.next);
while (p != nullptr && p != &sentinel)
{
    node_type* next = std::to_address(p->next);   // 先取下一个位置
    p->connect_to(*p);                            // 自指：等同于摘链后 release
    if (next == p)
    {
        break;                                    // 已断开，防止绕圈
    }
    p = next;
}
sentinel.connect_to(sentinel);                    // 链表复位为空
```

## 复杂度

`O(n)`，`n` 为 _节点_ 数量。

## 异常

（无）
