# `Yc::basic_intrusive_list<vptr>::~basic_intrusive_list`

析构链表对象**只销毁哨兵本身**：

- 不销毁任何 _节点_；
- 不把 _节点_ 脱管；
- 不释放任何内存。

如果析构时链上还有 _节点_，它们的 `prev` / `next` 仍指向这个哨兵，而哨兵已经不存在了。
之后再访问这些 _节点_ 就是未定义行为。因此销毁非空链表前应先 [`clear`](clear.md)：

```C++
Yc::basic_intrusive_list<> l;
task a{1}, b{2};
l.push_back(a);
l.push_back(b);

l.clear();   // a / b 脱管，之后销毁 l 是安全的
```

## 复杂度

常数。

## 异常

（无）
