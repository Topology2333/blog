---
title: "矩阵微积分简明教程笔记"
date: "2026-03-19"
categories: ["math"]
---

## 前言

朋友发来了一个名为 `gentle.pdf` 的神秘文件。 中文翻译可以是《矩阵微积分简明教程》。遂写一份笔记。不同的是与原文章不同，这篇笔记会更偏张量语言。

---

## 笔记

视矩阵为二阶张量 $X^i_j$。

那么 $\text {tr}(A) = A_i ^i$，于是迹的转置不变性显然。$\operatorname {tr}(AB) = \operatorname {tr}(A_i ^k B_k^j) = A_i^k B_k^i$。同理 $\operatorname {tr}(ABC) = \operatorname {tr}(A_i ^k B_k^j C_j^l) = A_i^k B_k^j C_j^i$。于是一列矩阵的积的迹在循环置换下的不变性显然。

<!-- 迹也可以看成 Frobenius Inner Product。$\operatorname{tr}(A) = \langle A, I \rangle = A_{ij} \delta_{ij} = A_{ii}$。 -->

线性型 $y^j = A_i^j x^i$。二次型 $f = A_{ij} x^i x^j$，注意到 $f = A'_{ji}x^jx^i = A'_{ij}x^ix^j$，因此 $f = (A_{ij}+A'_{ij}) x^ix^j/2$。也就是说，实际的有效输入只有 $(A+A')/2$。

Kronecker 积 $C^{ik}_{jl} = (A \otimes B)^{ik}_{jl} = A^i _j B^k_l$，得到的应该是一个 $4-$dim 的张量。只不过原文按照矩阵的表示方法。自然地，$(A \otimes B)^{ik}_{jl} (C \otimes D)^{jl}_{mn} = (A^i_j B^k_l) (C^j_m D^l_n) = (A^i_j C^j_m) (B^k_l D^l_n) = (AC)^i_{\phantom{i}m} (BD)^k_{\phantom{k}n}$，即 $(A\otimes B)(C\otimes D) = (AC)\otimes (BD)$。

vec 把一个指标对映射成一个单指标，根据原文的约定，对于 $\phi(i,j)$ 而言，前面的因子是慢指标，后面的因子是快指标（因为按列堆叠）。因此实际上对于矩阵 $A^i_j$ 而言，$\operatorname{vec} A$ 的行指标 $i$ 是快指标而列指标 $j$ 是慢指标，于是 $(\operatorname{vec} A)^{\phi (j,i)} = A^i_j$。而对于张量积的约定，很自然地有前面的指标是慢指标而后面的指标是快指标，于是很自然地有 $A^i_j\otimes B^k_l = C^{ik}_{jl} :\sim C^{\phi(i,k)}_{\phi(j,l)}$

我们定义 $Y_l^i = A_j^iB_k^jC_l^k$，则 $(\text{vec}Y)^{\phi(l,i)} = Y_l^i = C_l^k A_j^i (\text{vec} B)^{\phi(k,j)} = (C'\otimes A)_{\phi(k,j)}^{\phi(l,i)} (\text{vec} B)^{\phi(k,j)}$。即：在实际矩阵运算中 $C'\otimes A$ 这个 $(2,2)$ 型张量的上下指标分别被线性化展平，形成一个矩阵；而 $B$ 和 $Y$ 也分别被展平成向量。因此 $\operatorname{vec}(ABC) = (C' \otimes A) \operatorname{vec}(B)$。

对于原文中的 $K$，原定义为 $K \text{vec} A = \text{vec} A'$。这里采用张量的形式。按照上文约定，设 $B = A', x = \text{vec} A, y = \text{vec} B$，则 $y^{\phi(k,l)} = K^{\phi(k,l)}_{\phi(i,j)}x^{\phi(i,j)}$，且 $B^l_k = \delta_{jk}\delta^{il}A_i^j$。又 $B_k^l = K_{jk}^{il}A_i^j$，所以 $K_{jk}^{il} = \delta_{jk}\delta^{il}$。可以注意到这里的指标展平方式和前文两个矩阵的张量积的展平方式不同，一个简单的理解方式是，置换张量改变了指标的快慢次序。

于是很自然地，当 $m = n$，对于输入坐标 $\phi(i,j)$ 和输出坐标 $\phi(k,l)$，迹要求二者相等，求解得 $(i,j)=(k,l)$，于是 $\text{tr}(K) = \delta_{ji}\delta^{ij} = \delta_i^i = n$。

$K_{\phi(i,j)}^{\phi(k,l)} :\sim \delta_{jk}\delta^{il}, {K'} _{\phi(i,j)}^{\phi(k,l)} :\sim K_{\phi(k,l)}^{\phi(i,j)} :\sim \delta_{li}\delta^{kj}$，即得 $K$ 的对称性。又有 $(K^2)_{\phi(u,v)}^{\phi(k,l)} = \delta_{jk} \delta^{il}\delta_{vi}\delta^{uj} = \delta_k^u\delta_v^l$，即单位阵。

下面证明交换性定理 The Commuting Property，即 $K(A\otimes B) = (B\otimes A)K$。设等式左边为 $L$ 右边为 $R$。则 $L_{jl}^{uv} = K_{ik}^{uv}A^i_jB^k_l = \delta^v_i\delta ^u_kA^i_jB^k_l = A^v_j B_l^u$，$R_{st}^{ki} = A^i_jB^k_lK_{st}^{lj} = A^i_jB^k_l\delta^l_t\delta ^j_s = A^i_s B_t^k$，$(u,v)\mapsto(k,i),(j,l)\mapsto(s,t)$，因此 $L=R$，得证。

对于 $N_n = \frac{1}{2}(I+K)$，有 $N_n^2 = \frac{1}{4} (I+2K+K^2) = \frac{1}{4} (I+2K+I) =  \frac{1}{2}(I+K) = N_n$。显然 $N_n$ 也满足 $N_n(A\otimes B) = (B\otimes A)N_n$。

定义一个还原矩阵 $D_n$，把一个 $n(n+1)/2$ 维的向量填充在矩阵的下三角位置还原并对称还原原矩阵对应的拉直的向量。即 $D_n \text{vech}(A) = \text{vec}(A)$，此处 $A = A'$。于是 $K_nD_n \text{vech} X= K_n \text{vec} X = \text{vec} X' = \text{vec} X = D_n \text{vech} X$，即 $K_nD_n = D_n$。
