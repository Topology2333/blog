---
title: "扩散模型，蒸馏，以及量化"
date: "2026-03-23"
categories: ["AI"]
tags: ["AI","diffusion","distill","quantization"]
---

## 前因

读到朋友写的一篇[博客](<https://aucannot.github.io/posts/distilled-diffusion-quantization/>)，又想起来一两年前我的随机过程数学老师建议我去学习一下 diffusion model。现在我打算写一篇博客。由于没有正经阅读 AI 文献，以下部分有一些是 Gemini-generated，注意甄别。

## 几个概念

### 文生图模型

以扩散模型（Diffusion Models） 为核心，并结合 Transformer 增强特征提取能力的复合架构。可以拆解为三个核心组件：Autoencoder（空间压缩）、Text Encoder（语义理解）和 Denoising Backbone（去噪骨架）。

---

Autoencoder (VAE) 空间压缩的目标是学习两个映射函数：编码器 (Encoder) $\mathcal{E}$ 和 解码器 (Decoder) $\mathcal{D}$。

具体来说，给定一张图像 $x \in \mathbb{R}^{H \times W \times 3}$，通过编码器 $\mathcal{E}$ 可以将其转换为一个低维的潜在表示 $z = \mathcal{E}(x)$，其中 $z \in \mathbb{R}^{h \times w \times c}$，且通常 $h \ll H, w \ll W$。随后，解码器 $\mathcal{D}$ 可以用来将 $z$ 恢复回原始图像空间，即 $\mathcal{D}(z) \approx x$。

这样一来，扩散操作就不再直接作用于像素空间 $x$，而是在潜在特征空间 $z$ 上进行，从而大幅减少了计算开销。

---

Text Encoder 文本编码器是一个将离散文本转为连续向量空间的映射 $c = \tau_\theta(y)$。定义：给定提示词（Prompt）$y$，编码器 $\tau_\theta$（如 CLIP 或 T5）将其转换为条件向量 $c$。数学意义：$c$ 通常是一个序列向量 $\in \mathbb{R}^{L \times d}$，其中 $L$ 是 Token 长度，$d$ 是特征维度。为生图提供语义指导。

---

去噪骨架是扩散模型的核心，数学上表现为一个噪声预测算子 $\epsilon_\theta$。在训练阶段，我们要最小化以下目标函数：

$$\mathcal{L} = \mathbb{E}_{z, \epsilon \sim \mathcal{N}(0,1), t, c} \left[ \| \epsilon - \epsilon_\theta(z_t, t, c) \|^2_2 \right]$$
</br>
变量含义：$z_t$：在 $t$ 时刻加噪后的潜变量。$c$：来自文本编码器的条件向量。$\epsilon_\theta$：神经网络（U-Net 或 Transformer），它学习预测注入到 $z$ 中的噪声 $\epsilon$。

这里 Gemini 给出了 “$t$ 代表噪声强度”的论述。在我的追问下，它给出如下说明：通常定义一系列随时间 $t$ 变化的参数，通常记作 $\alpha_t$。$\alpha_0\sim 1,\alpha_T\sim 0$。参数化：  
$$z_t = \sqrt{\bar{\alpha}_t} z_0 + \sqrt{1 - \bar{\alpha}_t} \epsilon$$
其中 $\epsilon \sim \mathcal{N}(0, \mathbf{I})$，开根号是为了保持新分布的方差不变。

我的理解方式是，噪声是随机的，但去噪声的过程和 $c$ 有关，因而不是纯随机的，相当于带有一个 Guidance 的随机演化。

---

### 模型蒸馏
