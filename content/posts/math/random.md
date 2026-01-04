---
title: "随机分析笔记"
date: "2026-01-03"
categories: ["math"]
---


## 概率论基础

$\Omega$ 是非空集合，$\mathcal{F}$ 是 $\Omega$ 的子集族。

---

$\sigma$ 代数
: 称 $\mathcal{F}$ 是一个 $\sigma$ 代数，如果：
: (1) $\varnothing\in\mathcal{F}$
: (2) $A \in \mathcal{F} \implies A^c \in \mathcal{F}$
: (3) $A_1, A_2 \cdots \in \mathcal{F} \implies \bigcup _{n=1}^{\infin} A_n\in \mathcal{F} $

---

概率
: 概率测度 $\mathbb{P}  :\mathcal F \to [0,1],A\mapsto \mathbb{P} (A)$。称 $\mathbb{P} (A)$ 为 $A$ 的概率，三元组 $(\Omega,\mathcal{F} ,\mathbb{P})$ 为一个概率空间。要求：
: (1) $\mathbb{P} (\Omega) = 1$
: (2) $A_i \in \mathcal{F}, A_i \cap A_j = \varnothing \implies \mathbb{P} \Big(\bigcup _{n=1}^{\infin} A_n\Big) = \sum _{n=1} ^{\infin} \mathbb{P} (A_n)$

---

均匀测度
: 也称勒贝格测度 $\mathcal{L}$。
: 由全体闭区间出发而生成的 $\sigma$ 代数称为 [0,1] 的子集的 Borel $\sigma$ 代数，记为 $\mathcal{B}([0,1])$。
: $\mathbb{P}([a,b]) = b-a, 0\le a\le b\le 1$

---

几乎必然
: 设 $(\Omega,\mathcal{F} ,\mathbb{P})$ 是一个概率空间。如果 $A\in \mathcal{F} \land \mathbb{P}(A) = 1$，称事件 $A$ 几乎必然发生。

---

随机变量
: 设 $(\Omega,\mathcal{F} ,\mathbb{P})$ 是一个概率空间。称 $X: \Omega\to \mathbb{R}$ 是一个随机变量，如果：
: (1) $B\in \mathbb{B}(\mathbb{R}) \implies \{X\in B\} := \{\omega \in \Omega; X(\omega)\in B \}\in \mathcal{F} $

---

分布测度
: 设 $X$ 是概率空间 $(\Omega,\mathcal{F} ,\mathbb{P})$ 上的一个随机变量。$X$ 的分布测度是一个概率测度
: $\mu_X: \mathbb{B}(\mathbb{R}) \to [0,1], \mu_X(B) \mapsto \mathbb{P} \{X\in B\} $

---

指示函数
: 当 $\omega\in A$ 时 $\mathbb{I}_A(\omega) = 1$ 否则 $\mathbb{I}_A(\omega)=0$。随机变量 $\mathbb{I}_A$ 称为集合 $A$ 的指示函数。对于 $A\sub \Omega$，定义
$$\int _A X(\omega) d \mathbb{P}(\omega) = \int _\Omega \mathbb{I}_A(\omega) X(\omega) d \mathbb{P}(\omega)$$

---

期望
: $X$ 是在 $(\Omega,\mathcal{F} ,\mathbb{P})$ 上的随机变量。$X$ 的期望定义为：
: $$\mathbb{E}(X) = \int _\Omega X(\omega)d\mathbb{P}(\omega)$$
: 有詹森不等式：$\varphi$ 是 $\mathbb{R}$ 上的实值凸函数且 $\mathbb{E}(X)\le \infin$，则 $\varphi(\mathbb{E}X)\le \mathbb{E}\varphi(X)$  

## 信息和条件期望
