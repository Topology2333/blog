---
title: "De Bruijn index"
date: "2026-03-23"
categories: ["pl"]
tags: ["Programming Language"]
---

## 目的

考虑到 $\alpha$-等价性，$\lambda x. x$ 和 $\lambda y. y$ 在逻辑上是完全一样的，但写法不同，通过一套改写规则达到统一。

---

## 规则

### 定义

改写后的符号用自然数标识，标识为从当前位置往外数，第几个 $\lambda$ 符号是我的绑定者。一些例子：

- $\lambda x. x$ 被改写成 $\lambda 1$
- $\lambda x. \lambda y. x$ 改写成 $\lambda \lambda 2$
- $\lambda x. \lambda y. \lambda z. x z (y z)$ 改写成 $\lambda \lambda \lambda 3 1 (2 1)$

形式化地，用 De Bruijn index 表示的 $\lambda$ 项，其语法为：

$$M, N ::= n | M N | \lambda M$$

其中 $n$ 是大于 0 的自然数，表示变量；$M N$ 表示应用；$λ M$ 表示抽象。

若变量 $n$ 处在至少 $n$ 个 $\lambda$ 的作用域内，则它是绑定变量；否则是自由变量。变量 $n$ 的绑定位置，是它所处作用域中从内向外数的第 $n$ 个 $\lambda$。

---

### $\beta$ 规约

$(\lambda M) N$ 的 $\beta$-归约中，需要做三件事：

1. 找出 $M$ 中那些由最外层这个 $\lambda$ 绑定的变量；
2. 因为外层 $\lambda$ 被消去了，所以把 $M$ 中自由变量的编号整体减一；
3. 用参数 $N$ 替换对应位置，同时根据替换发生时所在的 $\lambda$ 层深，适当提升 $N$ 中自由变量的编号。

例子：$(\lambda\ \lambda\ 4\ 2\ (\lambda\ 1\ 3))\ (\lambda\ 5\ 1)$

对应普通记号：$(\lambda x.\ \lambda y.\ z\ x\ (\lambda u.\ u\ x))\ (\lambda x.\ w\ x)$

替换过程：

- 标出将被替换的位置：$\lambda\ 4\ \Box\ (\lambda\ 1\ \Box)$
- 外层 $\lambda$ 消失，自由变量编号整体减一：$\lambda\ 3\ \Box\ (\lambda\ 1\ \Box)$
- 把方框 $\Box$ 替换为 $\lambda\ 5\ 1$，并根据所在 $\lambda$ 层提升自由变量编号：
  - 第一个方框在 1 层 $\lambda$ 之下，因此替换成 $\lambda\ 6\ 1$
  - 第二个方框在 2 层 $\lambda$ 之下，因此替换成 $\lambda\ 7\ 1$
- 得到 $\lambda\ 3\ (\lambda\ 6\ 1)\ (\lambda\ 1\ (\lambda\ 7\ 1))$

---

替换的形式化定义：一个替换可以写成无限序列 $M_1.M_2.M_3\ldots$，其中第 $i$ 项 $M_i$ 表示第 $i$ 个自由变量将被替换成什么。所有相关变量编号加上 $k$ (shift) 记作 $\uparrow^k$。$\uparrow^0$ 是恒等替换。一个有限替换 $M_1.M_2.\ldots.M_n$ 实际是 $M_1.M_2.\ldots.M_n.(n+1).(n+2)\ldots$，也就是只替换前 $n$ 个变量，其余保持不变。

替换作用记作 $M[s]$，替换的组合满足 $M[s_1\,s_2] = (M[s_1])[s_2]$。

- 变量：第 $n$ 个变量在替换后变成第 $n$ 个替换项；
- 应用：分别对左右两边替换；
- 抽象：进入 $\lambda$ 以后，要把替换表整体向上调整一层
  - 即 $(\lambda M)[s] = \lambda (M[1 . (s \uparrow^1)])$
  - 递归地，有 $(\lambda \lambda P)[s] = \lambda(\lambda P [1 . (s \uparrow^1)]) = \lambda (\lambda ( P [1 . 2 . (s \uparrow^2)] ))$

因此，$\beta$-归约可以简洁地写成 $(\lambda M)N \rightarrow M[N.1.2.3\ldots]$。例如，$(\lambda x. \lambda y. x)N$，也就是 $(\lambda \lambda 2)N$。$M = (\lambda 2)$，$s = N.1.2.3\dots$，要规约 $(\lambda 2)\ [N.1.2.3\dots]$。调整之后，$s' = 1 . (N[\uparrow^1]) . (1[\uparrow^1]) . (2[\uparrow^1]) \dots = 1 . (N\uparrow^1) . 2 . 3 . 4 \dots$。找到 $s'$ 的第二项，也就是 $N\uparrow^1$。所以，$(\lambda (\lambda 2))\ N$ 归约后的结果是 $\lambda (N\uparrow^1)$。

对于 $\uparrow^1$ 的解释为：若 $(\lambda x. \lambda y. x)\ N$ 的 $N$ 中含有自由变量 $z$，那么它原来往上索引会超出最外层环境，而在 $\beta-$规约之后，最外层环境增加了一层，自由变量 shift 是非常合理的。

---

### 优缺点

这样做的好处很明显：

- 机器友好，判断两个表达式是否相等，只需要看数字序列是否一致，不需要进行复杂的变量更名（$\alpha$-conversion）。
- 两个 $\alpha$-等价的项在内存中的表示是完全唯一的。可以直接通过哈希或简单的内存比较

但也有坏处：

- 代换复杂：在进行 $\beta$-归约时，被代入项跨越的 $\lambda$ 层数发生变化，其内部所有指向外部的数字都必须进行加减校准。带来实现上的复杂度。
- 人类难读。

---

## 其他

也有其他的策略，比如混合策略，对绑定变量使用 De Bruijn index 以方便计算，对自由变量使用名字以方便阅读。
