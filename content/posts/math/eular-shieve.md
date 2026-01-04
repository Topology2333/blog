---
title: "质数筛"
date: "2026-01-04"
categories: ["math","algorithm"]
---

## 两种筛的分析

### 埃拉托斯特尼筛

对于每一个质数 $p$，标记其倍数 $p^2,p(p+1),p(p+2)\cdots$，对于足够大的 $n$ 和固定的 $p$，标记次数约为
$$T(n)\sim \sum_{p\le n} \frac{n}{p} = n\sum _{p\le n}\frac{1}{p}$$
利用质数调和级数的渐近式
$$\sum _{p\le n}\log\log n+B+o(1)$$
从而 $T(n)=n\log\log(n)$。有一个 $\log\log n$ 的因子，说明重复访问。

---

### 欧拉筛

记 $\text{lp}⁡(n)$ 为 $n$ 的最小质因子（*least prime factor*）。由最小质因数引入划分 $\mathcal{L}_p=\{n\ge 2;\text{lp}(n)=p\}$。

由二元对 $(n,p)$ 双射地标记一个合数 $x=n\times p$，满足性质 $\text{lp}(x)=p$。

为了给出遍历算法，我们让 $n$ 自然生长，而限制 $p\in \mathcal{P}$ 的范围 $p\le \text{lp}(n)$，以保持 $\text{lp}(x)=p$ 的性质。

---

$p\le \text{lp}(n)$ 的证明
: 设 $p_n:=\text{lp}(n)$，那么 $\text{lp}(n\times p)=\min\{p,p_n\} = p$，说明当 $p\le ln(n)$ 时，性质得到延续。
: 若 $p\gt p_n$，$\text{lp}(n\times p)=\min\{p,p_n\} = p_n\neq p$，性质被破坏。此时得到的合数 $n\times p$ 应该隶属于 $\mathcal{L}_{p_n}$ 而非 $\mathcal{L}_p$，因此不应在此时筛去。

---

## 积性函数的自然分解

### 基本概念

$f:\mathbb{N}\to \mathbb{C}$ 称积性的，如果 $f(mn) = f(m)(n)$。如果对所有 $m,n$ 都满足，则称为完全积性的。

定义狄利克雷卷积

$$ (f*g)(n):= \sum_{d \mid n}f(d)g \Big(\frac{n}{d} \Big ) $$

狄利克雷卷积可以保持积性。

定义 $\varepsilon := \llbracket n=1\rrbracket, 1(n) = 1, \text{id}(n)=n$，那么 $1*\mu=\varepsilon, 1*1=\tau, \text{id}*1=\sigma, 1*\varphi=\text{id}$

**TODO**: 补充详细描述和代码
