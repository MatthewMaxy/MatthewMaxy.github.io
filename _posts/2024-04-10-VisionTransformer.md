---
layout: post
title:  "计算机视觉 - Transformer"
date:   2024-04-10 15:05:21 +0800
tags:   CV Transformer
color: rgb(255,90,90)
cover: '../assets/Blogs/2024-04-10/Transformers.jpeg'
subtitle: '计算机视觉里的Transformer模型(更新中)'
---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

## Vision  Transfomer（ViT）

原文链接： [An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale](https://arxiv.org/pdf/2010.11929.pdf)

### 模型框架

![ViT Model](/assets/Blogs/2024-04-10/Vit.jpg)

### Multi-head Self-Attention（MSA）模块计算量

#### 1. 通过 $W_q, W_k ,W_v$ 生成对应的 $Q, K, V$

这里假设 $Q, K, V$ 的向量长度与 feature map 的深度 $C$ 保持一致（ 即 $d_{model} = C$ ）。那么对应所有像素生成 $Q$ 的过程如下式：

$$ A^{hw \times C} \cdot W_q^{C \times C} = Q^{hw \times C} $$

+ $A^{hw \times C}$ ：将所有像素（token）拼接在一起得到的矩阵（一共有hw个像素，每个像素的深度为C）

+ $W_q^{C \times C}$：生成query的变换矩阵
+ $Q^{hw \times C}$：所有像素通过 $W^{C \times C}_q$ 得到的query拼接后的矩阵

根据矩阵运算的计算量公式可以得到生成 $Q$ 的计算量为 $hw \times C \times C$，生成 $K$ 和 $V$ 同理都是 $hwC^2$， __总共是 $3hwC^2$__。

#### 2. 相似度计算与查询

计算相似度， 对应计算量为 $(hw)^2C$

$$Q^{hw \times C} \cdot (K^{hw \times C})^T = X^{hw \times hw} $$

忽略除以 $\sqrt {d}$ 与$softmax(·)$ 的复杂度，假设得到$P^{hw \times hw}$，和 $V^{hw \times C}$ 相乘计算量为 $(hw)^2C$

$$ P^{hw \times hw} \cdot V^{hw \times C} = B ^{hw \times hw} $$

即该部分**总计算量为$2(hw)^2C$**

#### 3. 多头注意力计算

若为单头注意力模块，计算量计算到此为止，共为：$3hwC^2 + 2(hw)^2C$

但多头注意力模块相比单头注意力模块的计算量多了最后一个融合矩阵 $W_O$  的计算量 $ hwC^2$ ：

$$B^{hw \times C} \cdot W^{C \times C}_O = O^{hw \times C}$$

因此MSA模块 __总计算量为$4hwC^2 + 2(hw)^2C$__。

## Swin Transformer

原文链接： [Swin Transformer: Hierarchical Vision Transformer using Shifted Windows](https://arxiv.org/pdf/2103.14030.pdf)

### 模型框架

![Swin Model](/assets/Blogs/2024-04-10/Swin.png)

### Window based Multi-head Self-Attention（W-MSA）模块计算量

由于自注意力机制在 $ M \times M $ 的窗口内计算， 因此对于每个窗口可以套用MSA的计算量公式即：

$$ 4M^2C^2 + 2(M^2)^2C$$

由于共有$\frac{h}{M} \times \frac{w}{M}$ 个窗口，因此总的计算复杂度为

$$\Omega(W-MSA) = 4hwC^2 + 2M^2hwC $$
