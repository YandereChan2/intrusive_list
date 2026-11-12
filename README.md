# Yc::intrusive_list

侵入式双向链表的 header-only 实现。链表**不拥有节点**：节点的创建、析构由用户负责，链表只维护链接；插入、摘除、拼接、排序、合并、逆序都只改指针。

# 目标

- 和 C++ 标准库保持风格一致
- 零开销：不分配、不搬移元素，节点地址稳定
- 与 `std::list` 相同的位置表示：哨兵 + 双向迭代器
- header-only，要求 C++23

# 非目标

- 不是拥有型容器：不构造 / 析构元素、不分配内存
- 不记录元素个数
- 不提供 `std::list` 的值级接口（`push_back(value)`、`insert(pos, value)` 等）

# 主要代码架构

- [`intrusive_list.h`](intrusive_list.h) 提供 `Yc::basic_intrusive_list<vptr>`、两套迭代器和两种节点类型。
- 链表对象只持有一个哨兵节点；每个节点自带 `prev` / `next`，因此插入 / 摘除 / 拼接 / 排序 / 合并 / 逆序都是常数或线性的指针操作。
- 节点默认处于**脱管**状态（链接指向自身）；挂链后为**纳管**状态。`release()` 是幂等的摘链操作。
- `insert` / 单点 `splice` / 区间 `splice` 是静态函数：它们只需要位置，不需要链对象，因此可以作用于"表头嵌在别的对象里"的兄弟链。
- 模板参数 `vptr` 决定节点指针的表示：默认 `void*`；换成缀饰指针（fancy pointer）时，节点、哨兵、迭代器整体使用缀饰指针。

完整文档见 [`doc/intrusive_list_readme/intrusive_list.md`](doc/intrusive_list_readme/intrusive_list.md)；
概念说明见 [`doc/intrusive_list_readme/easy_for_understand.md`](doc/intrusive_list_readme/easy_for_understand.md)。

# 文件

| 文件 | 说明 |
|---|---|
| [`intrusive_list.h`](intrusive_list.h) | 侵入式链表的实现 |
| [`test_intrusive_list.cpp`](test_intrusive_list.cpp) | 自测：排序稳定性、合并、删除 / 去重、逆序、异常安全 |
| [`doc/intrusive_list_readme/`](doc/intrusive_list_readme/intrusive_list.md) | 使用文档 |

# 快速开始

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

    // 解引用得到 node_type&，再 static_cast 回自己的类型
    assert(static_cast<task&>(l.front()).id == 1);

    // 把 b 移到表头
    auto it = l.begin();
    ++it;
    list::splice(l.begin(), l, it);
    assert(static_cast<task&>(l.front()).id == 2);

    l.sort([](const list::node_type& x, const list::node_type& y)
        {
            return static_cast<const task&>(x).id < static_cast<const task&>(y).id;
        });
    l.reverse();

    l.clear();      // 只摘链：a / b / c 仍然存活，可以正常析构
}
```

# 构建测试

使用 MSVC（C++23）：

```bat
cl /nologo /std:c++latest /EHsc /W4 /utf-8 test_intrusive_list.cpp
test_intrusive_list.exe
```

使用 CMake：

```bat
cmake -S . -B build
cmake --build build --config Release
ctest --test-dir build -C Release --output-on-failure
```

预期输出：

```text
test_intrusive_list: ok
```

# 与拥有型容器的关系

- [`Yc::list_without_size`](https://github.com/YandereChan2/list_without_size)：接口与 `std::list` 一致的拥有型容器；内部持有一条 `basic_intrusive_list`，负责节点分配与 `T` 的构造 / 析构。它**没有 `size()`**。
- [`Yc::trees`](https://github.com/YandereChan2/trees)：森林容器，用 `basic_intrusive_list` 表示根链与孩子链。

# 许可证

MIT，见 [LICENSE](LICENSE)。
