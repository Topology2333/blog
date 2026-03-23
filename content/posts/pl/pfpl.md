---
title: "Practical Foundations For Programming Languages"
date: "2026-03-22"
categories: ["pl"]
tags: ["Programming Language"]
---

## 简介

编程语言是由 Judgments 和 Inference Rules 定义的。Judgments 例如 “$e$ 是一个表达式（$e \text{ exp}$）” 或者 “$\tau$ 是一个类型（$\tau \text{ type}$）”。Inference Rules 的标记方法是：横线上方写前提（Premises）横线下方写结论（Conclusion）：

$$\frac{\Gamma \vdash e_1 : \text{nat} \quad \Gamma \vdash e_2 : \text{nat}}{\Gamma \vdash e_1 + e_2 : \text{nat}}$$

</br>

这种嵌套结构最后会成为证明树（Proof Tree）。

此外还有 Concrete Syntax 和 Abstract Syntax 的区别。前者是字符串，比如 1 + 1 或者 +(1, 1)。后者是抽象结构，可以使用抽象绑定树 (Abstract Binding Trees, ABTs) 来表示。比如表达式 1 + 2 在抽象语法中可能被表示为 plus(num[1]; num[2])。这种消歧表示直接体现了语言的逻辑结构。

---

$$\frac{\Gamma \vdash e_1 : \text{bool} \quad \Gamma \vdash e_2 : \tau \quad \Gamma \vdash e_3 : \tau}{\Gamma \vdash \text{if } e_1 \text{ then } e_2 \text{ else } e_3 : \tau}$$

中间的 $e_1$ 必须是 bool 类型；两个分支 $e_2$ 和 $e_3$ 必须具有相同的类型 $\tau$。结果类型就是那个共同的类型 $\tau$。
