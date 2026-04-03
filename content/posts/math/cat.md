---
title: "范畴论小记"
date: "2026-03-25"
math_engine: "mathjax"
categories: ["math"]
tags: ["category theory"]
---

## 起因

答辩完了，好舒服，让 LLM（流浪猫）带我学习。

> 等等！  
> 友人云：**单子不过是自函子范畴上的一个幺半群而**已。何意味？

为了解答这个问题，我在流浪猫的带领下重构了段落。

## 基本概念

所谓**幺半群** (Monoid)，即一个集合上定义一个二元运算，存在单位元，并满足结合律。这个和范畴无关。

（不过结合范畴定义可知，一个幺半群本身就可以看作是一个只有一个对象的范畴：态射就是幺半群的元素，态射的复合就是二元运算）

---

下面解释**范畴**的概念

一个范畴 (Category) $\mathcal{C}$ 由以下三个要素组成：

- 对象 (Objects)：$A, B, C$
- 态射 (Morphisms)：$f: A \to B$
- 组合律 (Composition)：考虑态射 $f: A \to B,\ g: B \to C$，必存在一个合态射 $g \circ f: A \to C$

另需满足：

- 结合律：对于 $h \circ (g \circ f) = (h \circ g) \circ f$
- 单位律：每个对象 $A$ 都存在一个态射 $id_A: A\to A$，使得 $\forall f, f\circ id_A = f = id_A \circ f$

---

**题外话**：同构是特殊的态射。

若对态射 $f: A \to B$ **存在**态射 $g: B \to A$，s.t. $g \circ f = id_A$ 且 $f \circ g = id_B$  
那么称：通过态射 $f$，$A$ 与 $B$ 是同构的。

一点例子：

- 集合范畴，集合为对象，集合间的映射为态射
  - 双射且互为逆映射，那么两个集合在这个范畴意义下是同构的。
- 群范畴，群为对象，群同态为态射
  - 群同构对应的对象是同构的。
- 偏序集范畴，自然数为对象，若 $n \le m$，构造 $n$ 到 $m$ 到一个态射。
  - 同构对象只可能是同一个自然数。

---

接下来在范畴之间定义 **函子** (Functor)：$F: \mathcal{C} \to \mathcal{D}$

- 把 $\mathcal{C}$ 里的每个对象 $A$ 变成 $\mathcal{D}$ 里的对象 $F(A)$。
- 把 $\mathcal{C}$ 里的每个态射 $f: A \to B$ 变成 $\mathcal{D}$ 里的态射 $F(f): F(A) \to F(B)$。

注：函子本身也必须保持结合律和单位元性质。

例如，考虑一个从 Grp (群范畴) 到 Set (集合范畴) 的函子 $U$（遗忘函子 Forgetful Functor）：对给定群，得到群所在的集合。

---

自函子为映射到一个范畴自身的函子。自函子范畴 $[\mathcal{C}, \mathcal{C}]$ 的对象是所有的自函子，态射是这些函子之间的**自然变换** (Natural Transformation)。

对于 $\mathcal{C}$ 中的每一个对象 $X$，在 $\mathcal{D}$ 中都对应一个态射（箭头）：$$\alpha_X: F(X) \to G(X)$$这个箭头被称为**自然变换** $\alpha$ 在 $X$ 处的分量。

需满足自然性条件 (Naturality Condition)：如果在 $\mathcal{C}$ 中有一个态射 $f: X \to Y$，那么在 $\mathcal{D}$ 中，以下两条路径的结果必须完全相同：  
A：先通过 $\alpha_X$ 从 $F(X)$ 变到 $G(X)$，再通过 $G(f)$ 变到 $G(Y)$  
B：先通过 $F(f)$ 从 $F(X)$ 变到 $F(Y)$，再通过 $\alpha_Y$ 变到 $G(Y)$。

即：

$$\begin{CD}
F(X) @>\alpha_X>> G(X) \\
@VF(f)VV @VVG(f)V \\
F(Y) @>>\alpha_Y> G(Y)
\end{CD}$$

即：$G(f) \circ \alpha_X = \alpha_Y \circ F(f)$。

---

回到开篇那句话，函子复合就是这个范畴里的乘法运算。自函子范畴 $[\mathcal{C}, \mathcal{C}]$ 上的对象是自函子，"乘法"就是函子的复合 $\circ$：于是单子就是这个范畴上的幺半群。

**单子** (Monad) 的组成：
- 一个自函子 $T: \mathcal{C} \to \mathcal{C}$
- 单位自然变换 $\eta: Id \to T$（对应幺半群的单位元）
- 乘法自然变换 $\mu: T^2 \to T$（对应幺半群的二元运算，这里 $T^2 = T \circ T$）

**由此**：

- 约定左复合 $T\mu$。其分量为：$(T\mu)_X = T(\mu_X)$。
- 约定右复合 $\mu T$。其分量为：$(\mu T)_X = \mu_{T(X)}$。
- 对于 $\eta$，有类似的约定。

这里视角比较巧妙，需要进一步的解释。  

- 具体而言，自然变换 $\mu: T^2 \Rightarrow T$ 为底层范畴 $\mathcal{C}$ 中的对象 $X$ 指派了一个底层的态射 $\mu_X$。所以 $\mu_X$ 是底层范畴 $\mathcal{C}$ 里的一个普通态射。由于 $T$ 是一个函子，且 $\mu_X$ 是 $\mathcal{C}$ 中的一个态射，那么根据函子的性质，$T$ 能把这个态射映射成另一个态射，而这个新态射就是**左复合**的 $X$ 分量。
- 自然变换 $\mu: T^2 \Rightarrow T$ 为底层范畴 $\mathcal{C}$ 中的任何对象指派一个态射。考虑**右复合** $\mu T$ 在对象 $X$ 处的分量时，我们首先应用自函子 $T$ 于对象 $X$，得到底层范畴 $\mathcal{C}$ 中的一个新对象 $T(X)$。由于 $\mu$ 是一个自然变换，它为这个新对象 $T(X)$ 指派一个对应的底层态射。根据 $\mu$ 的定义，这个指派给对象 $T(X)$ 的态射就是 $\mu_{T(X)}$。这个态射的类型是从 $T^2(T(X))$ 指向 $T(T(X))$，即从 $T^3(X)$ 指向 $T^2(X)$。这个由 $X$ 诱导出的新态射 $\mu_{T(X)}$，就是**右复合**的 $X$ 分量。

在定义了左右复合后，给出单子需要满足的交换条件：

**结合律**：
$$\begin{CD}
T^3 @>T\mu>> T^2 \\
@V\mu TVV @VV\mu V \\
T^2 @>>\mu> T
\end{CD}$$

即 $\mu \circ T\mu = \mu \circ \mu T$。

**单位律**（左箭头渲染有问题故如此别扭表示）：
$$\begin{CD}
T @>\eta T>> T^2 \\
@| @VV\mu V \\
T @= T
\end{CD}
\qquad
\begin{CD}
T @>T\eta>> T^2 \\
@| @VV\mu V \\
T @= T
\end{CD}$$

即 $\mu \circ \eta T = id_T = \mu \circ T\eta$。

---

## 代码

以 Haskell 为例。

---

自函子 $T$ $\leftrightarrow$ `Functor`  
在 Haskell 里，对象是类型（`Int`, `String`），态射是函数（`a -> b`）。
自函子对应 `Functor` 类型类，它提供了一个上下文（比如 `Maybe` 或 `[]`），并用 `fmap` 映射态射：

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

---

单位自然变换 $\eta$ (eta) $\leftrightarrow$ `return` / `pure`  
$\eta$ 的作用是把底层范畴的一个普通对象 $X$，放入到自函子 $T$ 的上下文中，即 $Id \to T$。  
在 Haskell 中对应 `Monad` 的 `return`：把一个普通值放入最小的默认上下文中。

```haskell
return :: a -> m a
```

---

乘法自然变换 $\mu$ $\leftrightarrow$ `join`  
$\mu$ 的作用是 $T^2 \to T$。
在 Haskell 里，$T^2$ 就是**嵌套了两次的上下文**，如 `Maybe (Maybe Int)` 或者 `[[Int]]`。  
$\mu$ 的作用是 Flatten，对应 `Control.Monad` 里的 `join`。

```haskell
join :: Monad m => m (m a) -> m a
```

---

Haskell 里的单子标志性动作是 `>>=` (bind)。**`>>=` 可以看作是 `fmap` 和 `join` ($\mu$) 的组合**。

```haskell
(>>=) :: Monad m => m a -> (a -> m b) -> m b
x >>= f = join (fmap f x)
```

解释：

1. 考虑值 `x :: m a` 和函数 `f :: a -> m b`
2. 首先利用 `fmap` 把函数 `f` 应用到 `x` 内部。因为 `x` 本身有 `m`，`f` 又会产生一层 `m`，所以 `fmap f x` 的结果是嵌套的 `m (m b)` （即 $T^2$）
3. 接着利用单子乘法 $\mu$（即 `join`）把 `m (m b)` 转换为 `m b`（即 $T^2 \to T$）。

简而言之，`bind` 操作可以理解为先做了一次自函子映射，然后执行了一次乘法运算。也就是嵌套成两层后拍平。

---

意义：

如果直接定义 `m a -> m b` 会失去通用性。如果是 `a -> b`，函数只关心业务逻辑（如加减法）；如果是 `a -> m b`，函数只关心逻辑结果是否合法（如除法是否除以0）;如果是 `m a -> m b`，函数还需要处理“输入 `m a` 本身是否为空这种复杂任务。
