---
layout: post
title:  "机器学习知识点总结 "
date:   2024-04-19 08:00:00 +0800
tags:   MachineLearning
color: rgb(66,118,91)
cover: '../assets/Blogs/MachineLearning/title.png'
subtitle: '准备夏令营和春招面试知识点总结 &#124;'
---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']],
            splayMath: [ ['$$','$$'], ["\\[","\\]"] ]
            }
        });
    </script>
</head>

## 绪论

### 归纳偏好

#### NFL 定理（没有免费的午餐）

首先考虑 $\mathfrak{L}{a}$ 的训练集外误差

$$E_{o t e}\left(\mathfrak{L}{a} | X, f\right)=\sum{h} \sum_{\boldsymbol{x} \in \mathcal{X}-X} P(\boldsymbol{x}) \mathbb{I}(h(\boldsymbol{x}) \neq f(\boldsymbol{x})) P\left(h | X, \mathfrak{L}_{a}\right)$$

在二分类问题下考虑 NFL 定理：

$$\begin{aligned} \sum_{f}E_{ote}(\mathfrak{L}a\vert X,f) &= \sum_f\sum_h\sum{\boldsymbol{x}\in\mathcal{X}-X}P(\boldsymbol{x})\mathbb{I}(h(\boldsymbol{x})\neq f(\boldsymbol{x}))P(h\vert X,\mathfrak{L}a) \\ &=... \\ &=2^{\vert \mathcal{X} \vert-1}\sum{\boldsymbol{x}\in\mathcal{X}-X}P(\boldsymbol{x}) \cdot 1\ \end{aligned}$$ 

即总误差与学习算法无关，理解如下：
此处考虑我们希望学习的真实目标函数 $f$ 均匀分布，即所有问题出现的机会等同且所有任务一样重要。因此**没有一个学习算法可以在全体上优于其他，必须结合具体任务进行选择与设计**

#### Occam's razor 奥卡姆剃刀

若有多个假设与观察一致，则选择最简单的那个

## 模型评估与选择

### 经验误差与过拟合

#### 精度与错误率

$m$ 个样本中 $a$ 个样本分类错误:

+ 错误率（error rate）：$E = a/m$
+ 精度（accuray）：$1- a/m$

#### 数据划分

完整数据集 $D$，希望划分产生训练集 $S$ 和测试集 $T$

**留出法(hold out)：**

直接将数据集 $D$ 划分产生 $S$ 和 $T$，满足$D = S \cup T, S \cap T = \emptyset$

注意要保持数据分布一致性，例如分类问题中至少保持样本类别比例类似，即**分层采样**

**$p$ 次 $k$ 折交叉验证(cross validation)：**

将数据集 $D$ 划分产生 $k$ 个大小相似的互斥子集，满足$D = D_1 \cup D_2 \cup ... \cup D_k, \quad D_i \cap D_j = \emptyset(i \not ={j})$，每次用 $k-1$ 个做训练集，余下的做测试集，进行 $k$ 次训练和测试