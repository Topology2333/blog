---
title: "随机积分小记：几个概念之间的关系"
date: "2026-03-31"
categories: ["math"]
tags: []
---

## 前言

本文的目标是理清SDE（随机微分方程）、generator（生成元）和 Fokker-Planck 方程之间的关系，并说明 generator 或许才是**最本质**的对象，而 SDE 只是 generator 的一种具体实现方式。

## 基本对象：Markov 过程和 Semigroup

考虑一个 Markov 过程 $X_t$，由它出发定义半群（semigroup）：

$$P_t f(x) = E[f(X_t) \mid X_0 = x]$$

这个对象完全不涉及 SDE，仅依赖于 Markov 过程的本质性质。它表达了一个简单而深刻的想法：给定一个初值 $x$，一个可测函数 $f$ 在随机演化时间 $t$ 后的期望值。

$$P_0 f(x) = E[f(X_0) \mid X_0 = x] = f(x)$$

所以 $P_0$ 即为恒等算子。

$$
\begin{align}
P_{t+s} f(x) &= E[f(X_{t+s}) \mid X_0 = x]\\
&= E[E[f(X_{t+s}) \mid X_t] \mid X_0 = x]\\
&= E[P_s f(X_t) \mid X_0 = x]\\
&= P_t(P_s f)(x)
\end{align}
$$

这正是半群的组合性。Markov 过程的无记忆性保证了算子的这个结构。

## Generator：Semigroup 的微分

一旦我们有了 semigroup $P_t$，generator 就自然产生了——它就是这个 semigroup 在 $t=0$ 处的导数：

$$\mathcal{L}f(x) = \lim_{t\to0} \frac{P_t f(x)-f(x)}{t}$$

其描述了函数在随机演化下的瞬时变化率。换言之，若函数 $f$ 沿着随机过程漂移，generator 告诉我们它的期望值以多快的速度变化。至此仍不需要 SDE。只要有 Markov 过程，就自动具有 generator。

## SDE：Generator 的一种实现

现在引入 SDE。考虑一个由以下随机微分方程驱动的过程：

$$dX_t = a(X_t)dt + b(X_t)dW_t$$

其中 $a(x)$ 是漂移系数，$b(x)$ 是扩散系数，$W_t$ 是标准 Brownian 运动。

利用 Itô 引理，可以证明这个 SDE 对应的 generator 具有特殊形式：

$$\mathcal{L}f(x) = a(x)f'(x) + \frac{1}{2}b^2(x)f''(x)$$

### 证明：从 Itô 引理到 Generator

对于光滑函数 $f$，应用 Itô 引理：

$$df(X_t) = f'(X_t)dX_t + \frac{1}{2}f''(X_t)(dX_t)^2$$

将 $dX_t = a(X_t)dt + b(X_t)dW_t$ 代入。注意二次变分规则：

- $(dt)^2 = 0$
- $dt \cdot dW_t = 0$
- $(dW_t)^2 = dt$

因此：
$$(dX_t)^2 = [a(X_t)dt + b(X_t)dW_t]^2 = b^2(X_t)dt$$

代入 Itô 公式：

$$df(X_t) = f'(X_t)[a(X_t)dt + b(X_t)dW_t] + \frac{1}{2}f''(X_t)b^2(X_t)dt$$

$$= \left[a(X_t)f'(X_t) + \frac{1}{2}b^2(X_t)f''(X_t)\right]dt + f'(X_t)b(X_t)dW_t$$

对两边求期望，由于 $dW_t$ 项的期望为零：

$$E[df(X_t) \mid X_0 = x] = E\left[\left[a(X_t)f'(X_t) + \frac{1}{2}b^2(X_t)f''(X_t)\right]dt \mid X_0 = x\right]$$

在微小时间 $dt$ 内，$X_t$ 接近初值 $x$，所以：

$$E[f(X_{t+dt}) - f(X_t) \mid X_0 = x] \approx \left[a(x)f'(x) + \frac{1}{2}b^2(x)f''(x)\right]dt$$

两边除以 $dt$ 并取极限 $dt \to 0$：

$$\lim_{dt \to 0} \frac{E[f(X_{dt}) \mid X_0 = x] - f(x)}{dt} = a(x)f'(x) + \frac{1}{2}b^2(x)f''(x)$$

根据 generator 的定义 $\mathcal{L}f(x) = \lim_{t \to 0} \frac{P_tf(x) - f(x)}{t}$，我们得到：

$$\mathcal{L}f(x) = a(x)f'(x) + \frac{1}{2}b^2(x)f''(x)$$

这表明 SDE 的 generator 形式是由 Itô 引理唯一确定的。

这是一个重要的反演（inversion）：给定 SDE，我们可以计算出对应的 generator。  
反过来，给定一个这种特殊形式的 generator，我们可以构造一个相应的 SDE。  
理论上，generator 可以不对应任何 SDE——特别是对于跳过程、Lévy 过程或一般的 Markov 过程，它们有 generator 但没有 SDE 表示。

## 密度的演化：Fokker-Planck 方程

现在考虑随机过程的概率密度 $p(x,t)$ 的演化。这涉及到 generator 的一个重要伴侣——**对偶算子 $\mathcal{L}^*$**。

### 从 Backward 方程到 Forward 方程

由 generator 定义出发，对任意光滑函数 $f$ 和概率密度 $p(x,t)$，有：

$$\frac{d}{dt}E[f(X_t)] = E[\mathcal{L}f(X_t)]$$

展开期望积分形式：

$$\frac{d}{dt}\int_{\mathbb{R}} f(x)p(x,t)dx = \int_{\mathbb{R}} \mathcal{L}f(x) \cdot p(x,t)dx$$

左边改写为：

$$\int_{\mathbb{R}} f(x)\partial_t p(x,t)dx = \int_{\mathbb{R}} \mathcal{L}f(x) \cdot p(x,t)dx$$

这必须对所有光滑函数 $f$ 成立。现在对右边使用**分部积分**。对于 $\mathcal{L}f = af' + \frac{1}{2}b^2f''$：

$$\int_{\mathbb{R}} \mathcal{L}f(x) \cdot p(x,t)dx = \int_{\mathbb{R}} \left[af' + \frac{1}{2}b^2f''\right]p dx$$

**第一项分部积分**（假设边界项消失）：

$$\int_{\mathbb{R}} af' \cdot p \, dx = -\int_{\mathbb{R}} f \cdot \partial_x(ap) dx$$

**第二项分部积分两次**：

$$\int_{\mathbb{R}} \frac{1}{2}b^2f'' \cdot p \, dx = \int_{\mathbb{R}} f \cdot \frac{1}{2}\partial_{xx}(b^2p) dx$$

代入回原式：

$$\int_{\mathbb{R}} f(x)\partial_t p(x,t)dx = -\int_{\mathbb{R}} f \cdot \partial_x(ap) dx + \int_{\mathbb{R}} f \cdot \frac{1}{2}\partial_{xx}(b^2p) dx$$

$$= \int_{\mathbb{R}} f \left[-\partial_x(ap) + \frac{1}{2}\partial_{xx}(b^2p)\right] dx$$

由于这对所有 $f$ 成立，必有：

$$\partial_t p = -\partial_x(ap) + \frac{1}{2}\partial_{xx}(b^2p)$$

这就是著名的 **Fokker-Planck 方程**（也称为 **Kolmogorov 前向方程**），其描述了概率密度如何随时间演化。

### 对偶性的意义

注意到：

- **Backward operator**（向后方程的 generator）：$\mathcal{L}f = af' + \frac{1}{2}b^2f''$
- **Forward operator**（向前方程的 generator）：$\mathcal{L}^* p = -\partial_x(ap) + \frac{1}{2}\partial_{xx}(b^2p)$

满足如下对偶性：

$$\int_{\mathbb{R}} \mathcal{L}f \cdot p \, dx = \int_{\mathbb{R}} f \cdot \mathcal{L}^* p \, dx$$

虽然 backward 和 forward 方程表面上不同，但它们是对偶的，描述的是同一个过程的两个互补视角。

## 三个视角，一个动力学

现在我们可以总结一下：一个随机系统本质上可以从三个不同的角度观察：

- **路径视角（SDE）**：单条轨迹怎样演化，$dX_t = a(X_t)dt + b(X_t)dW_t$
- **函数视角（Generator）**：可测函数的期望怎样变化，$d E[f(X_t)] = E[\mathcal{L}f(X_t)]dt$
- **密度视角（Fokker-Planck）**：概率密度函数怎样演化，$\partial_t p = -\partial_x(ap) + \frac{1}{2}\partial_{xx}(b^2 p)$

这三个方程在本质上描述的是同一个动力学系统。

---

简而言之概括性来看，更清晰的分层为：

1. **Markov 过程** → 最一般的随机过程，满足 Markov 性质
2. **Semigroup** → 由 Markov 过程导出的算子家族 $P_t$
3. **Generator** → Semigroup 的微分，是最本质的对象
4. **Backward 方程** → Generator 诱导的 PDE
5. **Forward 方程** → 密度演化的 PDE

只有当 generator 具有特殊的二阶微分形式时，我们才能构造对应的 SDE。很多重要的随机过程（如跳跃过程、稳定过程等）根本没有 SDE 表示，但它们有明确定义的 generator。
