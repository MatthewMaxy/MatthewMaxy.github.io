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

## 第一章 绪论

### 归纳偏好

#### NFL 定理（没有免费的午餐）

首先考虑 $\mathfrak{L}{a}$ 的训练集外误差

$$E_{o t e}\left(\mathfrak{L}{a} | X, f\right)=\sum_{h} \sum_{\boldsymbol{x} \in \mathcal{X}-X} P(\boldsymbol{x}) \mathbb{I} (h(\boldsymbol{x}) \neq f(\boldsymbol{x})) P\left(h | X, \mathfrak{L}_{a}\right)$$

在二分类问题下考虑 NFL 定理：

$$\begin{aligned} \sum_{f}E_{ote}(\mathfrak{L}a\vert X,f) &= \sum_f\sum_h\sum_{\boldsymbol{x}\in\mathcal{X}-X}P(\boldsymbol{x})\mathbb{I}(h(\boldsymbol{x})\neq f(\boldsymbol{x}))P(h\vert X,\mathfrak{L}a) \\ &=... \\ &=2^{\vert \mathcal{X} \vert-1}\sum_{\boldsymbol{x}\in\mathcal{X}-X}P(\boldsymbol{x}) \cdot 1\ \end{aligned}$$ 

即总误差与学习算法无关，理解如下：
此处考虑我们希望学习的真实目标函数 $f$ 均匀分布，即所有问题出现的机会等同且所有任务一样重要。因此**没有一个学习算法可以在全体上优于其他，必须结合具体任务进行选择与设计**

#### Occam's razor 奥卡姆剃刀

若有多个假设与观察一致，则选择最简单的那个

## 第二章 模型评估与选择

### 数据划分

完整数据集 $D$，希望划分产生训练集 $S$ 和测试集 $T$

#### 留出法(hold out)

直接将数据集 $D$ 划分产生 $S$ 和 $T$，满足$D = S \cup T, S \cap T = \emptyset$

注意要保持数据分布一致性，例如分类问题中至少保持样本类别比例类似，即**分层采样**

#### $p$ 次 $k$ 折交叉验证(cross validation)

将数据集 $D$ 划分产生 $k$ 个大小相似的互斥子集，满足$D = D_1 \cup D_2 \cup ... \cup D_k, \quad D_i \cap D_j = \emptyset(i \not ={j})$，每次用 $k-1$ 个做训练集，余下的做测试集，进行 $k$ 次训练和测试

划分 $k$ 个子集也有不确定性，因此可以重复 $p$ 次，最后评估的就是 $p \times k$ 次的平均结果

留一法（Leave-one-out）：即令 $k = m$，每次只取一个作为测试集样本

#### 自助法（bootstrapping）

每次从 $D$ 中随机挑选一个样本放入 $D'$ 中，重复 $m$ 次，获得与数据集等大的训练集，此时约有 $1/e$ 的元素未被采样，因此用 $D'$ 做训练集，$D \backslash D'$ 为测试集，这样的结构称为 **“包外估计”**

### 性能度量

回归问题（均方误差）：

$$E(f ; D)=\frac{1}{m} \sum_{i=1}^{m}\left(f\left(\boldsymbol{x}_{i}\right)-y_{i}\right)^{2}$$

$$E(f ; \mathcal{D})=\int_{\boldsymbol{x} \sim \mathcal{D}}(f(\boldsymbol{x})-y)^{2} p(\boldsymbol{x}) \mathrm{d} \boldsymbol{x} .$$

重点性能度量均以二分类为例

#### 错误率与精度

错误率：分类错误样本占样本总数比例

$$E(f ; D)=\frac{1}{m} \sum_{i=1}^{m} \mathbb{I}\left(f\left(\boldsymbol{x}_{i}\right) \neq y_{i}\right)$$

$$E(f ; \mathcal{D})=\int_{\boldsymbol{x} \sim \mathcal{D}} \mathbb{I}(f(\boldsymbol{x}) \neq y) p(\boldsymbol{x}) \mathrm{d} \boldsymbol{x}$$

精度：分类正确样本总数占样本总数比例

$$\operatorname{acc}(f ; D) =\frac{1}{m} \sum_{i=1}^{m} \mathbb{I}\left(f\left(\boldsymbol{x}_{i}\right)=y_{i}\right)  =1-E(f ; D)$$

$$\operatorname{acc}(f ; \mathcal{D})  =\int_{\boldsymbol{x} \sim \mathcal{D}} \mathbb{I}(f(\boldsymbol{x})=y) p(\boldsymbol{x}) \mathrm{d} \boldsymbol{x}  =1-E(f ; \mathcal{D})$$

#### 查全率、查准率、F1

真正例(TP)、假正例(FP)、真反例(TN)、假反例(FN)

混淆矩阵：

![matrix](/assets/Blogs/MachineLearning/1.png){:height="50%" width="50%"}

查准率（P）和查全率（R）：

$$P = \frac{TP}{TP+FP} \quad \qquad Q = \frac{TP}{TP + FN}$$

$P-R$ 曲线和平衡点（查准率 = 查全率）：

![PR](/assets/Blogs/MachineLearning/2.png){:height="50%" width="50%"}

$F1$ 度量（基于查准和查全的调和平均）：

$$F1 = \frac{2}{\frac{1}{P} + \frac{1}{R}} = \frac{2 \times P \times R}{P + R} = \frac{2 \times TP}{N + TP - TN}$$

$F1$度量的一般形式 $F_\beta$，旨在表达对查准/查全的不同偏好:

$$F_\beta =  \frac{(1 + \beta ^ 2) \times P \times R}{(\beta ^ 2 \times P) + R}$$

+ $\beta > 1$ 表明查全率更重要

多次测试（多个混淆矩阵）时关于$P, R, F1$的计算

+ 宏查全率、宏查准率、宏$F1$
  
$$\begin{array}{c}\text { macro- } P=\frac{1}{n} \sum_{i=1}^{n} P_{i} \\ \\ \text { macro- } R=\frac{1}{n} \sum_{i=1}^{n} R_{i} \\ \\ \text { macro- } F 1=\frac{2 \times \text { macro } P \times \text { macro }-R}{\text { macro- } P+\text { macro }-R}\end{array}$$

+ 微查全率、微查准率、微$F1$

$$\begin{array}{c}\text { micro- } P=\frac{\overline{T P}}{\overline{T P}+\overline{F P}} \\ \\ \text { micro- } R=\frac{\overline{T P}}{\overline{T P}+\overline{F N}}, \\ \\ \text { micro- } F 1=\frac{2 \times \text { micro- } P \times \text { micro- } R}{\text { micro- } P+\text { micro- } R}\end{array}$$



