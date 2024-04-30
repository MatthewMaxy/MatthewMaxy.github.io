---
layout: post
title:  "机器学习知识点总结 "
date:   2024-04-27 08:00:00 +0800
tags:   MachineLearning
color: rgb(66,118,91)
cover: '../assets/Blogs/MachineLearning/title.png'
subtitle: '准备夏令营和春招面试知识点总结'
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

{:toc}

### 归纳偏好

{:toc}

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

#### ROC和AUC

考虑：不知道如何划分阈值判断正负类时，对概率进行降序排列，按顺序逐个截断设为正类，其中 ROC 曲线纵轴是“真正例率”（TPR），横轴是“假正例率”（FPR）

$$\mathrm{TPR}=\frac{T P}{T P+F N} \quad \quad \mathrm{FPR}=\frac{F P}{T N+F P} $$

![ROC](/assets/Blogs/MachineLearning/3.png){:height="70%" width="70%"}

**AUC 即为 ROC 曲线下方面积大小:**

$$\begin{aligned} \text{AUC} &= 1 - \frac{1}{m^{+} m^{-}} \sum_{\boldsymbol{x}^{+} \in D^{+}} \sum_{\boldsymbol{x}^{-} \in D^{-}}\left(\mathbb{I}\left(f\left(\boldsymbol{x}^{+}\right)<f\left(\boldsymbol{x}^{-}\right)\right)+\frac{1}{2} \mathbb{I}\left(f\left(\boldsymbol{x}^{+}\right)=f\left(\boldsymbol{x}^{-}\right)\right)\right) \\ &= \frac{1}{2} \sum_{i=1}^{m-1}\left(x_{i+1}-x_{i}\right) \cdot\left(y_{i}+y_{i+1}\right) \end{aligned}
$$

### 代价敏感错误率和代价曲线

主要解决分类错误代价不同问题 $cost_{ij}$ 表示把第 $i$ 类错判为 $j$ 的代价

$$\begin{aligned} E(f ; D ; cost) &= \frac{1}{m}\left(\sum_{\boldsymbol{x}_{i} \in D^{+}} \mathbb{I}\left(f\left(\boldsymbol{x}_{i}\right) \neq y_{i}\right) \times \operatorname{cost}_{01}\right. \\ & \left.+\sum_{\boldsymbol{x}_{i} \in D^{-}} \mathbb{I}\left(f\left(\boldsymbol{x}_{i}\right) \neq y_{i}\right) \times \operatorname{cost}_{10}\right)  \end{aligned}$$

相应的应该使用代价曲线（cost curve）衡量学习器期望代价：

横坐标是取值 [0, 1] 的正例概率代价，其中 $p$ 是样例为正例的概率:

$$P(+)cost = \frac{p \times cost_{01}}{p \times cost_{01} + (1-p) \times cost_{10}}$$

纵轴是取值 [0, 1] 的归一化代价，其中 $FPR$ 是假正例率，$FNR = 1- TPR$ 是假反例率：

$$cost_{norm} = \frac{FNR \times p \times cost_{01} + FPR \times(1-p) \times cost_{10} }{p \times cost_{01} + (1-p) \times cost_{10}}$$

ROC 曲线上每个点对应了代价平面上一条线段：

+ 假设 ROC 曲线上点坐标 (FPR, TPR)，则在代价平面上有线段(0, FPR) 到 (1, TPR)
+ 线段下面积为该条件下期望总体代价
+ 所有线段下界围成面积为学习器的期望总体代价

![代价曲线](/assets/Blogs/MachineLearning/5.png){:height="60%" width="60%"}

### 偏差与方差

偏差-方差分解尝试对学习算法的期望泛化错误率进行拆解

+ 以回归任务为例
+ 对测试样本 $x$，$y_D$ 为 $x$ 在数据集中的标记，$y$ 为 $x$ 的真实标记
+ $f(x;D)$ 为训练集 $D$ 上学得模型 $f$ 在 $x$ 上的预测输出

学习算法的期望预测结果为（当前算法的期望输出）

\\[ \overline{f(x)} = \mathbb{E}_D[f(\boldsymbol{x};D)] \\]

不同训练集产生的方差（训练集不同导致不同输出的方差）

\\[ var(x) = \mathbb{E}_D[(f(\boldsymbol{x};D) - \overline{f(x)})^2] \\]

噪声（真实标签和数据集标签差异）

\\[ \epsilon^2 = \mathbb{E}_D[(y_D - y)^2] \\]

期望输出与真实标记的差别（偏差bias）

\\[ bias^2(x) = (\overline{f(x)} - y)^2 \\]

假定噪声期望为0，可以对算法期望误差进行分解如下：

$$\begin{aligned} E(f;D) &= \mathbb{E}_D[(f(\boldsymbol{x};D) - y_D)^2] \\ &= \dots \\ &= \mathbb{E}_D[(f(\boldsymbol{x};D) - \overline{f(x)})^2] + (\overline{f(x)} - y)^2 + \mathbb{E}_D[(y_D - y)^2] \\ &= bias^2(x) + var(x) + \epsilon^2
\end{aligned}$$

即泛化误差可以分解为偏差、方差和噪声之和

+ 偏差即为训练不到位导致的，训练前期占主导作用
+ 方差是由于训练过拟合导致的，后期占主导作用

![loss](/assets/Blogs/MachineLearning/4.png){:height="70%" width="70%"}

## 第三章 线性模型

### 基本形式

$$f(\boldsymbol{x}) = \boldsymbol{w}^T\boldsymbol{x} + b$$

+ 线性模型具有良好的可解释性
+ 许多非线性可以通过在线性基础上引入层级结构或高维映射实现

### 线性回归（Linear Regression）

给定数据集

$$D =\{(\boldsymbol{x}_1,y_1),(\boldsymbol{x}_2,y_2),...,(\boldsymbol{x}_m,y_m)\} \quad \boldsymbol{x}=(\boldsymbol{x}_{i1};\boldsymbol{x}_{i2}; ... ;\boldsymbol{x}_{id})，y_i \in \mathbb{R}$$

线性回归试图学得 $f(x_i) = w^T x_i + b$ 使得 $f(x_i) \simeq y_i$，即为学习

$$\begin{split}(w^*,b^*) &= \arg\min_{(w,b)} \sum_{i=1}^m (f(x_i)-y_i)^2 \\\ &= \arg\min_{(w,b)} \sum_{i=1}^m (y_i-wx_i-b)^2\end{split}$$

#### 最小二乘法

最小二乘法可以实现对 $w, b$ 分别求偏导，得到 $w, b$ 最优解的闭式解（此处假设x只有一维）

$$ w = \frac{\sum_{i=1}^{m}y_i(x_i - \bar{x})}{\sqrt{m\sum_{i=1}^{m}x_i^2 - \left(\sum_{i=1}^{m}x_i\right)^2}}$$

$$ b = \frac{1}{m} \sum_{i=1}^m (y_i - w_ix_i)$$

#### 多元线性回归（一般情况）

若对于数据集 $D$，样本由 $d$ 个属性描述，仍可以用最小二乘法思路求解。为方便作出如下规定：

假设最后的 b 是样本的一个特征产生的，但因为对于所有样本保持一致，因此定为 1

\\[X = \begin{pmatrix} x_{11} & x_{12} & \dots & x_{1d} & 1 \\\ x_{21} & x_{22} & \dots & x_{2d} & 1 \\\ \vdots & \vdots & \ddots & \vdots & \vdots \\\ x_{m1} & x_{m2} & \dots & x_{md} & 1 \end{pmatrix} = \begin{pmatrix} \mathbf{x}_1^T & 1 \\\ \mathbf{x}_2^T & 1 \\\ \vdots & \vdots \\\ \mathbf{x}_m^T & 1 \end{pmatrix} \\]

相应的将 $b$ 融入 $\boldsymbol{w}$ 权重矩阵构成 $\boldsymbol{\hat{w}} = (\boldsymbol{w};b)$，标记也表示为 $\boldsymbol{y} = (y_1;y_2;...;y_m)$

可得最小化目标，及求解结果如下：

$$ w^* = \arg \min_{\boldsymbol{\hat{w}}} (\boldsymbol{y} – \boldsymbol{X} \boldsymbol{\hat{w}})^T(\boldsymbol{y} – \boldsymbol{X} \boldsymbol{\hat{w}})$$

$$ \frac{\partial E(\boldsymbol{\hat{w}})}{\partial \boldsymbol{\hat{w}}} = 2 \boldsymbol{X}^T ( \boldsymbol{X} \boldsymbol{\hat{w}} – \boldsymbol{y}) $$

求解闭式解时涉及逆矩阵运算，因此当 $\boldsymbol{X}^T\boldsymbol{X}$ 满秩时可以求解得到

$$\boldsymbol{\hat{w}^*} = (\boldsymbol{X}^T\boldsymbol{X})^{-1}\boldsymbol{X}^T\boldsymbol{y}$$

为什么一般不用逆矩阵：
+ 样例数少于属性数目导致不满秩
+ 多个维度线性相关导致不满秩
+ 矩阵乘法后值过大，取逆后小于最小精度

#### 广义线性模型

考虑单调可微函数 $g(\cdot)$ 使得

$$y = g^{-1}(\boldsymbol{w}^T\boldsymbol{x} + b)$$

其中一个典型案例：对数线性回归，实际上让 $e^{\boldsymbol{w}^T\boldsymbol{x} + b}$ 逼近 y

$$\ln(y) = \boldsymbol{w}^T\boldsymbol{x} + b$$

![ln](/assets/Blogs/MachineLearning/6.png){:height="50%" width="50%"}

### 对数几率回归（Logistic Regression）

考虑利用线性模型解决分类问题，只需找一个单调可微函数将分类任务的真实标记 $y$ 与线性回归模型的预测值联系起来(可参考广义线性模型)

对数几率回归选用的单调可微函数是一种 Sigmoid 函数(有时会讲这个函数就叫 Sigmoid，但一般 Sigmoid 其实是指形似 S 的系列函数，和阶跃函数相比如下图)

![sigmoid](/assets/Blogs/MachineLearning/7.png){:height="50%" width="50%"}

$$
S(z) = \frac{1}{1+ e^{-z}} \quad \text{求导性质}：S'(z) = S(z)(1-S(z)) 
$$

参考广义线性模型，将 $S$ 作为 $g( \cdot)$ 带入可以得到

$$
y = \frac{1}{1+ e^{-(\boldsymbol{w}^T\boldsymbol{x} + b)}} \quad  \ln{\frac{y}{1-y} = \boldsymbol{w}^T\boldsymbol{x} + b}
$$

可以将$y$ 视为分类为正例概率，$1-y$ 为负例概率

#### 如何求解 $\boldsymbol{\beta}$

假设 $\boldsymbol{\beta} = (\boldsymbol{w};b)$，利用极大似然法得到最小化目标函数：

$$ \ell(\boldsymbol{\beta}) = \sum_{i=1}^{m} (-y_i \boldsymbol{\beta}^T \boldsymbol{\hat{x_i}} + \ln(1+e^{\boldsymbol{\beta}^T \boldsymbol{\hat{x_i}}}))$$

可以利用诸如梯度下降法、牛顿法等进行最优解的求解，即可得到

$$\boldsymbol{\beta}^* = \arg \min_{\boldsymbol{\beta}} \ell(\boldsymbol{\beta})$$

具体推导过程可见：[推导过程](https://datawhalechina.github.io/pumpkin-book/#/chapter3/chapter3?id=_327)

### 线性判别分析（LDA）

给定训练样例集，设法将样例投影到一条直线上，使得同类样例的投影点尽可能接近、异类样例的投影点尽可能远离；在对新样本进行分类时，将其投影到同样的这条直线上，再根据投影点的位置来确定新样本的类别

![LDA](/assets/Blogs/MachineLearning/8.png){:height="50%" width="50%"}

#### 二分类问题

给定数据集 $D = \lbrace (\boldsymbol{x_i}, y_i) \rbrace_{i=1}^m \quad y_i \in \lbrace 0,1 \rbrace$
+ $\boldsymbol{X_i}, \boldsymbol{\mu_i}, \boldsymbol{\Sigma_i}$ 分别表示第 $i \in \lbrace 0,1 \rbrace$ 类示例的集合、均值向量、协方差矩阵
+ 将数据投影到直线 $\boldsymbol{w}$ 上
  + 两类样本的中心在直线上的投影分别为 $\boldsymbol{w^T\mu_0}$ 和 $\boldsymbol{w^T\mu_1}$
  + 两类样本的协方差分别为 $\boldsymbol{w^T\Sigma_0w}$ 和 $\boldsymbol{w^T\Sigma_1w}$

最大化目标：

$$\begin{aligned} J &=\frac{\left\Vert \boldsymbol{w}^{T}\boldsymbol{\mu}_{0}-\boldsymbol{w}^{T}\boldsymbol{\mu}_{1}\right\Vert ^{2}}{\boldsymbol{w}^{T}\boldsymbol{\Sigma} _{0}\boldsymbol{w}+\boldsymbol{w}^{T}\boldsymbol{\Sigma}_{1}\boldsymbol{w}} \\ \\ &=\frac{\boldsymbol{w}^{T}\left(\boldsymbol{\mu} _{0}-\boldsymbol{\mu}_{1}\right)\left(\boldsymbol{\mu}_{0}-\boldsymbol{\mu}_{1}\right)^{T}\boldsymbol{w}}{\boldsymbol{w}^{T}\left(\boldsymbol{\Sigma} _{0}+\boldsymbol{\Sigma}_{1}\right)\boldsymbol{w}}
= \frac{\boldsymbol{w}^{T}\boldsymbol{S_b}\boldsymbol{w}}{\boldsymbol{w}^{T}\boldsymbol{S_w}\boldsymbol{w}}\end{aligned}$$

+ 同类投影点尽可能近：同类投影点的协方差 $\boldsymbol{w}^{T}\boldsymbol{\Sigma}_0 \boldsymbol{w} + \boldsymbol{w}^{T}\boldsymbol{\Sigma}_1 \boldsymbol{w}$ 尽可能小
+ 异类投影点尽可能远离：类中心之间的距离 $\left\Vert \boldsymbol{w}^{T}\boldsymbol{\mu}_0 - \boldsymbol{w}^{T}\boldsymbol{\mu}_1\right\Vert ^2$ 尽可能大
+ 即最大化 $\boldsymbol{S_b}$ 和 $\boldsymbol{S_w}$ 的广义瑞利商

经过化简后可以得到

$$\boldsymbol{w} = \boldsymbol{S_w}^{-1}(\boldsymbol{\mu_0} - \boldsymbol{\mu_1})$$

具体推导过程可见：[推导过程](https://datawhalechina.github.io/pumpkin-book/#/chapter3/chapter3?id=_332)

#### 多分类推广

多分类问题中LDA的基本流程：

+ 计算每个类别的均值 $\mu_i$，全局样本均值 $\mu$
+ 计算类内散度矩阵 $S_w，全局散度矩阵 $S_t$ ，类间散度矩阵 $S_b$ 
+ 对矩阵 $S_w^{-1}S_b$ 做特征值分解
+ 取最大的 $d' \leq N -1$ 个特征值所对应的特征向量
+ 计算投影矩阵 $W$

**注意：** LDA在降维时利用了类别信息，是一种有监督的降维方法

### 多分类学习

#### OvO 和 OvR

![OvOR](/assets/Blogs/MachineLearning/9.png){:height="70%" width="70%"}

+ **存储测试：** OvR 需训练 N 个分类器；OvO 需训练 N(N-1)/2 个分类器，OvO的存储、测试时间开销通常比 OvR 更大
+ **训练：** OvR 的每个分类器均使用全部训练样例，OvO 的每个分类器仅用到两个类的样例，因此OvO 的训练更快
+ **预测性能：** 取决于具体的数据分布，在多数情形下两者差不多

#### MvM：纠错输出码（ECOC）

+ **编码:** 对 N 个类别做 M 次划分,每次划分将一部分类别划为正类,一部分划为反类，从而形成一个二分类训练集;这样一共产生 M 个训练集,可训练出 M 个分类器
+ **解码:** M 个分类器分别对测试样本进行预测，这些预测标记组成一个编码，将这个预测编码与每个类别各自的编码进行比较，返回其中距离最小的类别作为最终预测结果

![ECOC](/assets/Blogs/MachineLearning/10.png){:height="70%" width="70%"}

### 类别不平衡问题

#### 阈值移动（再缩放）

一般情况下，分类器决策规则为:

$$\frac{y}{1-y} > 1 \qquad 则预测为正例$$

考虑到正负样本数量不同，假设正例数目为$m^+$，负例数目为$m^-$，则对决策策略进行修改（left）。但分类器决策还是基于1进行决策，因此可以对预测值进行缩放（right）

$$
\frac{y}{1-y} > \frac{m^+}{m^-} \quad 则预测为正例；\qquad \frac{y'}{1-y'} = \frac{y}{1-y} \times \frac{m^-}{m^+} 
$$

也是[代价敏感学习](#代价敏感错误率和代价曲线)的基础，即用 $cost^+/cost^-$ 代替 $m^-/m^+$

#### 欠采样

直接对训练集里的反类样例进行“欠采样”(undersampling)，即去除些反例使得正、反例数目接近，然后再进行学习

+ 损失数据可能导致部分性质丢失（欠拟合）

#### 过采样

对训练集里的正类样例进行“过采样”(oversampling)，即增加一些正例使得正、反例数目接近，然后再进行学习

+ 简单重复采样可能导致严重过拟合

## 第四章 决策树

### 学习基本算法

**伪代码：**

```python
输入: 训练集 D = {(X1,Y1), (X2,Y2), ..., (Xm, Ym)};
属性集 A = {a1, a2,..., ad}.
过程: 函数 TreeGenerate(D, A)
1:  生成结点 node;
2:  if D 中样本全属于同一类别 C then
3:      将 node 标记为C类叶结点; return     # (1)
4:  end if
5:  if A = Ø OR D中样本在A上取值相同 then
6:      将 node 标记为叶结点,其类别标记为D中样本数最多的类; return  # (2)
7:  end if
8:  从 A 中选择最优划分属性 a*;
9:  for a* 的每一个值 ai do
10:     为 node 生成一个分支; 令 Di 表示 D 中在 a* 上取值为 ai 的样本子集;
11:     if Di 为空 then
12:         将分支结点标记为叶结点,其类别标记为 D 中样本最多的类; return    # (3)
13:     else
14:         以TreeGenerate(Di, A \ {a*})为分支结点
15:     end if
16: end for

输出:以 node 为根结点的一棵决策树
```

递归返回的三种情况，分别对应伪代码中(1)~(3)
+ 当前结点包含的样本全属于同一类别，无需划分
+ 当前属性集为空，或是所有样本在所有属性上取值相同，无法划分
+ 当前结点包含的样本集合为空，不能划分

### 划分节点依据

解决如何选择节点使用特征问题，其中（1）ID3 决策树采用信息增益；（2）C4.5 决策树采用增益率；（3）CART决策树选用基尼系数。本质都是希望各个节点的“纯度”尽可能高。

#### 信息熵（前置知识）

$$\text{Ent}(D)=-\sum_{k=1}^{|\mathcal{Y}|}p_k\log_2 p_k$$

+ $\text{Ent}(D)$ 表示数据集 $D$ 的信息熵，越小意味着 $D$ 的纯度越高
+ 数据集 $D$ 中第 $k$ 类样本占比为 $p_k (k=1,2,\dots, | \mathcal{Y} |)$

#### 信息增益（ID3）

$$\text{Gain}(D,a)= \text{Ent}(D)-\sum_{v=1}^{V}\frac{|D^v|}{|D|}\text{Ent}(D^v)$$

+ 离散属性 $a$ 有 $V$ 个可能的取值 $\{a^1,a^2,...,a^V\}$
+ 若用 $a$ 划分，会产生 $V$ 个分支，其中第 $v$ 个结点包含所有取值为 $a^v$ 的样本,记为$D^v$ 
+ 依据各节点包含样本数目进行加权求和，计算本次划分能降低多少信息熵，即信息增益
+ 选择属性 $a_* = \arg \max_{a \in A} Gain(D,a)$ 进行划分

#### 增益率（C4.5）

信息增益偏好于可取值数目较多的属性，例如序号（每个序号分一个节点可以让信息增益最大，信息熵为0）。因此需要基于属性本身进行约束。

$$ Gain\_ratio(D, a) = \frac{Gain(D, a)}{IV(a)}$$

其中 $IV(a)$ 为属性固有值，一属性可能取值数目越多则 $IV(a)$ 通常会越大，计算公式如下：

$$ IV(a) = \sum_{v=1}^{V} \frac{|D^v|}{|D|} \log_2 \frac{|D^v|}{|D|} $$

注：**增益率划分可能对取值数目较少的属性有偏好**，因此一般采用启发式的操作：先从候选划分属性中找出信息增益高于平均水平的属性，再从中选择增益率最高的

#### 基尼指数（CART）

数据集 $D$ 的纯度可以用基尼值衡量：

$$\begin{aligned} \text{Gini}(D) &= \sum_{k=1}^{|\mathcal{Y}| } \sum_{k' \not= k} p_kp_{k'} \\ &= 1-\sum_{k=1}^{|\mathcal{Y}| }p_k^2 \end{aligned}$$

基尼值反映了从数据集中随机抽取两个样本，标记类别不一致的概率，因此基尼值越小纯度越高

基尼指数定义如下，选择属性 $a_* = \arg \min_{a \in A} Gini_index(D,a)$ 进行划分

$$Gini\_index(D, a) = \sum_{v=1}^{V}{\frac{|D^v|}{|D|}}Gini(D^v)$$

### 剪枝

#### 预剪枝

基本策略与优缺点：
+ 生成过程中，每个节点划分前进行评估：不能提升泛化性能则将当前节点标记为叶子
+ 优点：降低了过拟合的风险、减少了训练和测试的时间开销
+ 缺点：基于贪心限制节点分裂，带来欠拟合风险

#### 后剪枝

基本策略与优缺点：
+ 首先生成一棵完整决策树，自下而上考察非叶节点，若换成叶子可以提升性能则替换
+ 如果性能不变按照 Occam's razor 也应替换为叶子
+ 优点：保留了更多的分支，欠拟合风险更小
+ 缺点：先生成再剪枝，因此训练时间开销更大

### 连续值、缺失值处理

#### 连续值处理

基本思路：连续变量离散化进而采用二分策略，即对原连续特征的值排序后（假设 $n$ 个取值），对每两个相邻值的平均值 $t$ 做为候选划分点，共有 $n-1$ 个候选划分点

$$\begin{aligned} \text{Gain}(D,a) &= \max_{t \in T_a} \text{Gain}(D,a,t) \\ &=\max_{t \in T_a} \text{Ent}(D)-\sum_{\lambda \in \{+,-\}} \frac{|D_t^{\lambda}|}{|D|} \text{Ent}(D_t^{\lambda}) \end{aligned}$$

+ $Gain(D, a, t)$ 为基于划分点 $t$ 二分后的信息增益
+ $D_t^+$ 和 $D_t^-$ 分别表示那些在属性 $a$ 上不小于（大于）$t$ 的样本

#### 缺失值处理

核心问题：
+ 如何在属性值缺失的情况下进行划分属性选择
+ 给定划分属性,若样本在该属性上的值缺失,如何对样本进行划分?

符号定义如下：
+ 假定数据集 $D$，属性 $a$，特征 $a$ 有 $V$ 个取值
+ $\tilde{D}$ 为 $D$ 中在属性 $a$ 上没有缺失值的样本子集
+ $\tilde{D}^v$ 为 $\tilde{D}$ 中在属性 $a$ 上取值为 $v$ 的样本子集
+ $\tilde{D}_k$ 为 $\tilde{D}$ 中属于第 $k$ 类的样本子集
+ 每个样本 $x$ 赋予权重 $w_x$

$$
\begin{aligned} & \rho = \frac{\Sigma_{x\in \tilde{D}}  w_x}{\Sigma_{x\in D} w_x} \\

& \tilde{p}_k = \frac{\Sigma_{x\in \tilde{D}_k} w_x}{\Sigma_{x\in \tilde{D}}  w_x} \quad (1\le k\le |\mathcal{Y}|) \\

& \tilde{r}_v = \frac{\Sigma_{x\in \tilde{D}^v}  w_x}{\Sigma_{x\in \tilde{D}} w_x} \quad (1\le v\le V) \end{aligned}$$

+ $\rho$ 表示无缺失值样本所占的比例
+ $\tilde{p}_k$ 表示无缺失值样本中第 $k$ 类所占的比例
+ $\tilde{r}_v$ 表示无缺失值样本中在属性 $a$ 上取值 $a^v$ 的样本所占的比例

针对问题（1）即训练时，信息增益表达式可以得到推广：

$$\begin{aligned} Gain(D, a) &= \rho \times Gain(\tilde{D}, a) \\ 
&= \rho \times \left(Ent (\tilde{D}) - \sum_{v=1}^{V} \tilde{r}_{v} Ent (\tilde{D}^{v}) \right) \end{aligned}$$

$$\text{Ent}(\tilde{D}) = -\sum_{k=1}^{|\mathcal{Y}|} \tilde{p}_k \log_2 \tilde{p}_k$$

针对问题（2）即推理时:
+ 若样本 $x$ 在划分属性 $a$ 上的取值已知:
  + 将 $x$ 划入与其取值对应的子结点
  + 样本权值在子结点中保持为 $w_x$
+ 若样本 $x$ 在划分属性 $a$ 上的取值未知:
  + 将 $x$ 同时划入所有子结点
  + 样本权值在与属性值 $a^v$ 对应的子结点中调整为 $\tilde{r}_v \cdot w_x$

### 多变量决策树

首先考虑对于单变量决策树模型的决策边界：

![Tree](/assets/Blogs/MachineLearning/11.png){:height="30%" width="30%"}![line](/assets/Blogs/MachineLearning/12.png){:height="30%" width="30%"} 


若能使用斜线划分，数模型将进一步简化。因此尝试对每个节点采用多个属性进行线性组合，即每个非叶节点是一个形如 $\Sigma_{i=1}^dw_ia_i$ 的线性分类器，构建多变量决策树和相应的决策边界示意如下

![Tree](/assets/Blogs/MachineLearning/13.png){:height="30%" width="30%"}![line](/assets/Blogs/MachineLearning/14.png){:height="30%" width="30%"}

## 神经网络

### 单层感知机

**输出：**

$$y = f(\sum_{i=1}^nw_ix_i-\theta)$$

+ $f$ 为激活函数，通常为 Sigmoid 函数
+ $w_i$ 为各输入权重， $\theta$ 为阈值
+ 训练时可以将 $\theta$ 看为一个输入固定为 -1.0，权重为 $w_{n+1}$ 的哑节点

**权重调整策略：**

$$w_i \leftarrow w_i + \Delta w_i \qquad \Delta w_i = \eta (y - \hat{y})x_i$$

单层感知机只能解决线性可分问题。面对非线性可分问题，如“异或”，则需要使用多层感知机，也可以叫多层前馈神经网络

### 误差逆传播算法（BP算法）

#### 符号说明

用来解决多层感知机的训练问题，符号说明如下

+ 数据集：$D = \lbrace (x_1, y_1), (x_2, y_2), \dots, (x_m, y_m) \rbrace$ ,  $x_i \in R^d$ , $y_i \in R^l$
+ 网络结构：$d$ 个输入神经元、$l$ 个输出神经元、$q$ 个隐层神经元
+ 神经元阈值：
  + $\theta_j$：输出层第 $j$ 个神经元的阈值
  + $\gamma_h$：隐层第 $h$ 个神经元的阈值
+ 神经元连接权重：
  + $v_{ih}$：输入层第 $i$ 个神经元与隐层第 $h$ 个神经元之间的连接权
  + $w_{hj}$：隐层第 $h$ 个神经元与输出层第 $j$ 个神经元之间的连接权 
+ 神经元输入输出：
  + $\alpha_h = \sum_{i=1}^d v_{ih}x_i$：隐层第 $h$ 个神经元接收到的输入
  + $b_h$：隐层第 $h$ 个神经元的输出
  + $\beta_j = \sum_{h=1}^q w_{hj}b_h$：输出层第 $j$ 个神经元接收到的输入
+ 对训练样本 $(x_k, y_k)$
  + 神经网络输出： $\hat{y}_k = (\hat{y}_1^k,\hat{y}_2^k,\dots \hat{y}_l^k)$
  + $\hat{y}_j^k = f(\beta_j - \theta_j)$
  + 均方误差为：$E_k = \frac{1}{2}\Sigma_{j=1}^l(\hat{y}_j^k-y_j^k)^2$

共有 $(d + l + 1)q + l$ 个参数需要学习：
+ 连接权重：
  + 输入层到隐藏层：$d \times q$
  + 隐藏层到输出层：$q \times l$
+ 阈值：
  + 隐藏层：$q$
  + 输出层：$l$

#### 梯度更新公式
  
BP算法基于随机梯度下降，对于学习率 $\eta$，需要对两个权重、两个阈值做更新，更新如下：

$$ \Delta w_{hj} = \eta g_j b_h$$
$$ \Delta \theta_{j} = -\eta g_j $$
$$ \Delta v_{ih} = \eta e_h x_i$$
$$ \Delta \gamma_{h} = - \eta e_h$$

其中 $g_j, b_h, e_h$ 含义如下：

$$ b_h = \frac{\partial\beta_j}{\partial w_{hj}}$$
$$ g_j = \hat{y}_j^k(1-\hat{y}_j^k)(y_j^k-\hat{y}_j^k)$$
$$ e_h = b_h(1-b_h)\sum_{j=1}^l w_{hj}g_j$$

具体推导过程可见：[推导过程](https://datawhalechina.github.io/pumpkin-book/#/chapter5/chapter5?id=_510)

#### 误差逆传播算法

```python
输入：训练集 D = {(x_k, y_k)}^m_{k=1};
     学习率 η
过程：
1. 在(0,1)范围内随机初始化网络中所有连接权和阈值
2. repeat
3. for all (x_k, y_k) ∈ D do
4.     根据当前参数计算当前样本的输出 \hat{y}_k;
5.     计算输出层神经元的梯度项 g_j;
6.     计算隐层神经元的梯度项 e_h;
7.     更新连接权 w_{hj}, v_{ih} 与阈值 θ_j, γ_h
8. end for
9. until 达到停止条件

输出：连接权与阈值确定的多层前馈神经网络
```

#### 注意点

BP算法目标是要最小化 $D$ 上的累积误差，但是上述“标准BP算法”更新的是单个样本的误差
+ 整个数据集全部计算一遍误差后再进行梯度更新
+ 设置 batch-size，每个 batch 更新一次

BP神经网络经常会遭遇过拟合，可以尝试以下方法缓解
+ “早停”：若验证集准确度下降则停止更新完成训练
+ 引入正则化：在损失函数中引入描述网络复杂度的部分，如所有权重阈值平方和，修正后的目标函数如下
  + $E = \lambda \frac{1}{m}\sum_{k=1}^m E_k + (1-\lambda)\sum_iw_i^2$
  + 其中 $\lambda \in (0, 1)$，用于对经验误差和网络复杂度进行折中，通常利用交叉验证选择

### 全局最小和局部最小

![min](/assets/Blogs/MachineLearning/15.png){:height="50%" width="50%"}

尝试跳出局部最小：

+ 不同参数初始化并训练多个网络，再选择最优的
+ 模拟退火：每一步都有一定概率接受次优解，但这个概率应该逐渐变小以保证训练稳定
+ 随机梯度下降
+ ...
  
## 第六章 支持向量机

### SVM基本思想

目标：对数据集 $D$，假设为二分类任务，期望在样本空间中找到一个划分超平面将不同样本分开，且泛化性能最后（即后面所说的间隔最大）

划分超平面定义如下，其中 $w$ 为法向量决定面的方向，$b$ 决定截距

$$w^Tx + b = 0$$

样本空间中点 $x$ 到超平面距离为

$$r = \frac{|w^Tx + b|}{||w||}$$

若能正确分类，则有如下不等式成立，其中最近的几个训练样本点使等号成立，他们就是 **“支持向量”**
$$\begin{cases} w^T x_i + b > +1, & y_i = +1 ; \\\ w^T x_i + b \le -1, & y_i = -1. \end{cases}$$

两个异类支持向量到超平面距离和即为间隔

$$\gamma = \frac{2}{||w||}$$

![svm](/assets/Blogs/MachineLearning/16.png){:height="50%" width="50%"}

欲找到最大间隔的划分超平面，就是要满足如下最优化条件的 $w，b$

$$\begin{aligned} & \min\limits_{w,b} \quad \frac{1}{2}\|w\|^2 \\ & s.t. \quad y_i(w^Tx_i + b) \ge 1, \quad i = 1, 2, \ldots, m \end{aligned} $$

**模型结果求解过程见：**[求解过程](https://datawhalechina.github.io/pumpkin-book/#/chapter6/chapter6?id=_69)。对 $w, b$ 求偏导后得到表达式如下,其中 $\alpha_i$ 是拉格朗日乘子,对应样本 $(x_i，y_i)$ ：
$$\begin{aligned} &w = \sum_{i =1}^m\alpha_iy_ix_i, \\ & 0 = \sum_{i=1}^m \alpha_i y_i\end{aligned}$$

**最优化目标转化过程见：**[转化过程](https://datawhalechina.github.io/pumpkin-book/#/chapter6/chapter6?id=_611)。转化后可以得到最优化目标如下（对偶问题）：

$$\begin{aligned} &\max\limits_{\alpha} \quad\frac{1}{2}\sum_{i=1}^m\alpha_i-\sum_{i=1}^m\sum_{j=1}^ma_{i}a_jy_iy_jx_i^Tx_j \\ &\text{s.t.} \quad\sum_{i=1}^ma_{i}y_i=0; \quad a_i\ge 0,\quad i=1,2,...,m.
\end{aligned}$$

同时上述过程要满足 KKT 条件，

$$\begin{cases} \alpha_i \ge 0;\\\ y_if(x_i)-1\ge 0;\\\ \alpha_i(y_if(x_i)-1)=0..\end{cases}$$

考察KKT条件可以发现，模型结果只可能与 $y_if(x_i) = 1$ 的样本（即支持向量）有关，这也是为什么被称为支持向量机。

**优化问题求解方法：** SMO 方法
+ 选取一对需要更新的 $\alpha_i, \alpha_j$
+ 固定其他参数，求解获得更新后的 $\alpha_i, \alpha_j$

### 核函数

#### 提出背景
在基本SVM模型中假设特征空间线性可分，但一般情况下需要将特征映射到高维空间，才可以实现线性可分。

假设映射函数 $\phi(x)$，则映射后在特征空间中划分平面可以表示为

$$f(x) = w^T\phi(x) + b$$

求解最优化问题转化为

$$\begin{aligned} &\max\limits_{\alpha} \quad\frac{1}{2}\sum_{i=1}^m\alpha_i-\sum_{i=1}^m\sum_{j=1}^m\alpha_{i}\alpha_jy_iy_j\phi(x_i)^T\phi(x_j) \\ &\text{s.t.} \quad\sum_{i=1}^m\alpha_{i}y_i=0; \quad \alpha_i\ge 0,\quad i=1,2,...,m.
\end{aligned}$$

核函数的提出就是为了解决最优化问题中出现 $\phi(x_i)^T\phi(x_j)$ 空间维度很高甚至无穷难以计算的问题。即假设存在 $\kappa(x_i,x_j) = \phi(x_i)^T\phi(x_j)$，带入求解后可以得到模型结果如下

$$\begin{aligned} f(x) &= w^T\phi(x) + b \\ &= \sum_{i=1}^m\alpha_iy_i\phi(x_i)^T\phi(x)+b \\ &= \sum_{i=1}^m\alpha_iy_i\kappa(x_i,x)+b\end{aligned}$$

#### 核函数和特征空间选择

正常思路：（1）先找到一个$\phi(\cdot)$ 将数据映射到高维线性可分空间；（2）找到相应的 $\kappa(\cdot, \cdot)$。但这两步都难以完成，因此我们尝试先选择一个 $\kappa(\cdot, \cdot)$，但有两个小问题：
+ （1）他对应的 $\phi(\cdot)$ 映射空间是线性可分的吗？
+ （2）什么样的函数可以作为核函数，即可以找到相应的$\phi(\cdot)$？

对于（1）其实是模型好坏的问题，因为即使不可分也只是影响模型效果，后文也有软间隔等策略进行补偿。问题（2）的解答如下：

![kappa](/assets/Blogs/MachineLearning/17.png)

简单来说，我们不知道怎么映射是最好的，因此我们选择一个核函数对原数据进行操作，本质上就是将样本映射到相应的高维空间。

因此**核函数隐式定义了整个高维样本空间**，在这样的意义下对**SVM 模型分类结果的好坏**起着至关重要的影响

#### 常用的核函数

名称 | 表达式 | 参数
---|---|---
线性核 |  $\kappa(x_i, x_j) = x_ix_j^T$  | 
多项式核 |  $\kappa(x_i, x_j) = (x_i^Tx_j)^d$  | $d \ge 1$ 为多项式的次数
高斯核 | $\kappa(x_i, x_j) = \text{exp}(-\frac{\lvert\lvert x_i-x_j \rvert\rvert^2}{2\sigma^2})$ | $\sigma > 0$ 为高斯核的带宽(width)
拉普拉斯核 |  $\kappa(x_i, x_j) = \text{exp}(- \frac{\lvert\lvert x_i-x_j \rvert\rvert}{\sigma})$  |  $\sigma> 0$ 
Sigmoid 核 |  $\kappa(x_i, x_j) = \text{tanh}(\beta_0x_i^Tx_j+\theta)$  | tanh为双曲正切函数,  $\beta > 0, \theta < 0$ 

此外可以通过函数组合得到
+ 若 $\kappa_1, \kappa_2$ 为核函数，则对任意正数 $\gamma_1, \gamma_2$，其线性组合也是核函数

$$\gamma_1 \kappa_1 + \gamma_2\kappa_2$$

+ 若 $\kappa_1, \kappa_2$ 为核函数，则核函数的直积也是核函数
  
$$\kappa_1 \otimes\kappa_2 (x,z) = \kappa_1(x,z)\kappa_2(x,z)$$

+ 若 $\kappa_1$ 为核函数，则对任意函数 $g(x)$，下述函数也为核函数

$$\kappa(x,z) = g(x)\kappa_1(x,z)g(z)$$

### 软间隔与正则化

#### 1. 软间隔的引入

如前文所说不一定线性可分，同时即使线性可分也可能过拟合，因此允许某些样本不满足 $y_if(x_i)-1\ge 0$ 的约束条件，即为软间隔

![soft](/assets/Blogs/MachineLearning/18.png){:height="50%" width="50%"}

基于软间隔，目标函数可以调整如下：

$$\min\limits_{w,b}\frac{1}{2}\|w\|^2 + C \sum_{i=1}^{m}\ell_{0/1}(y_i(w^Tx_i+b)-1)$$
$$l_{0/1}(z) = \begin{cases} 1, & \text{if } z < 0; \\ 0, & \text{otherwise}. \end{cases}$$

其中 $C>0$ 为惩罚系数，设置为 ∞ 时转化为硬间隔。$l_{0/1}(z)$是“0/1损失函数”，但考虑到数学性质太差，通常采用别的“替代损失”函数，通常是凸的连续函数且是“0/1损失函数”的上界。


$$\begin{aligned} &hinge损失: l_{hinge}(z) = max(0,1 - z) ; \\

&指数损失(exponential\ loss): l_{exp}(z) = exp(-z) ; \\

&对率损失(logistic\ loss): l_{log}(z) = log(1+exp(-z)) \end{aligned}$$

![loss](/assets/Blogs/MachineLearning/19.png){:height="70%" width="70%"}

#### 2. 软间隔支持向量机

引入软间隔和松弛变量 $\xi_i = \max(0, 1 - y_i(w^Tx_i+b)) \ge 0$ 后，可以重写优化目标，并得到软间隔支持向量机

$$\begin{aligned} &\min\limits_{w,b,\xi_i} \quad \frac{1}{2}\|w\|^2+C\sum_{i=1}^m\xi_i \\ & s.t. \quad y_i(w^Tx_i+b) \ge 1-\xi_i \\ & \qquad \xi_i \ge 0, \quad i=1,2,...,m.\end{aligned}$$

**模型结果求解过程见：**[求解过程](https://datawhalechina.github.io/pumpkin-book/#/chapter6/chapter6?id=_635)。对 $w, b, \xi_i$ 求偏导后得到表达式如下,其中 $\alpha_i, \mu_i$ 是拉格朗日乘子，对应样本 $(x_i，y_i)$：

$$\begin{aligned} &w = \sum_{i =1}^m\alpha_iy_ix_i, \\ & 0 = \sum_{i=1}^m \alpha_i y_i \\ & C= \alpha_i + \mu_i \end{aligned}$$

**最优化目标转化过程见：**[转化过程](https://datawhalechina.github.io/pumpkin-book/#/chapter6/chapter6?id=_640)。转化后可以得到最优化目标如下（对偶问题）：

$$\begin{aligned} &\max\limits_{\alpha} \quad\frac{1}{2}\sum_{i=1}^m\alpha_i-\sum_{i=1}^m\sum_{j=1}^m\alpha_{i}\alpha_jy_iy_jx_i^Tx_j \\ &\text{s.t.} \quad\sum_{i=1}^m\alpha_{i}y_i=0; \quad 0 \le \alpha_i \le C,\quad i=1,2,...,m.
\end{aligned}$$

同时上述过程要满足 KKT 条件，

$$\begin{cases} \alpha_i \ge 0，\quad \mu_i \ge 0，\\\ y_if(x_i)-1 + \xi_i \ge 0，\\\ \alpha_i(y_if(x_i)-1+\xi_i)=0， \\ \xi_i \ge 0， \mu_i\xi_i = 0\end{cases}$$

考察KKT条件可以发现，模型结果只可能与支持向量有关：
+ $\alpha_i = 0$ 或 $y_if(x_i) = 1 - \xi_i$ 与 SVM 分析一致
+ 若 $\alpha_i < C$，则 $\mu_i >0，\xi_i = 0$，即样本在间隔线上
+ 若 $\alpha_i = C$，则 $\mu_i = 0$
  + 若 $\xi_i \le 1$ 则在最大间隔内
  + 若 $\xi_i > 1$ 则为分类错误

如果用对率损失函数替代，会发现与Logist回归模型几乎一致，两者性能也相当，但存在如下差异：
+ Logistic 回归模型可以输出概率，而 SVM 输出不具备概率含义
+ Logistic 回归可以直接用于多分类，而 SVM 需要进一步加工

hinge损失得出的解具有稀疏性，而对率损失是光滑的单调递减函数，不能导出支持向量的概念，因此求解依赖更多的训练样本，预测开销也更大

#### 3. 正则化


