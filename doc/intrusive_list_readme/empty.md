# `Yc::basic_intrusive_list<vptr>::empty`

```C++
bool empty()const noexcept;
```

链表是否为空。

判断方式就是哨兵是否**脱管**（`prev` 指向自身），因此是常数时间，不需要数元素。

```C++
Yc::basic_intrusive_list<> l;
assert(l.empty());

task a{1};
l.push_back(a);
assert(!l.empty());
```

## 复杂度

常数。

## 异常

（无）
