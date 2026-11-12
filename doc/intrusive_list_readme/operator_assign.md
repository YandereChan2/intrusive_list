# `Yc::basic_intrusive_list<vptr>::operator=`

```C++
basic_intrusive_list& operator=(const basic_intrusive_list&) = delete;
basic_intrusive_list& operator=(basic_intrusive_list&&) = delete;
```

复制赋值与移动赋值都被隐式删除，原因与[构造函数](constructor.md#不可复制不可移动)相同：
哨兵是自指的，_节点_ 里的链接指向哨兵地址，按值复制 / 移动会让链接指向错误的哨兵。

要转移内容，使用 [`splice`](splice.md)：

```C++
Yc::basic_intrusive_list<> a, b;
// ... 往 a 里挂节点 ...
b.splice(b.begin(), a);   // 节点整体搬到 b，a 变空
```

要复制内容，需要新建 _节点_ 再挂链；拥有型容器
[`Yc::list_without_size`](https://github.com/YandereChan2/list_without_size) 已经实现了
完整的复制 / 移动语义。

## 复杂度

（不适用）

## 异常

（不适用）
