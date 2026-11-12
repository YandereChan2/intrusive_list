# `Yc::basic_intrusive_list<vptr>::front` , `back`

```C++
node_type& front()noexcept;
node_type& back()noexcept;
```

返回首 _节点_ / 尾 _节点_ 的引用。

- 空链时调用是未定义行为；
- 没有 const 重载：const 链表请用 `*l.begin()` / `*--l.end()`；
- 只返回 _节点_，用户数据要 `static_cast` 回自己的类型。

```C++
Yc::basic_intrusive_list<> l;
task a{1}, b{2};
l.push_back(a);
l.push_back(b);

assert(static_cast<task&>(l.front()).id == 1);
assert(static_cast<task&>(l.back()).id == 2);

const auto& cl = l;
assert(static_cast<const task&>(*cl.begin()).id == 1);
auto last = cl.end();
--last;
assert(static_cast<const task&>(*last).id == 2);
```

## 复杂度

常数。

## 异常

（无）
