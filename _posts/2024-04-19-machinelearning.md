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

给定数据集$D =\{(\boldsymbol{x}_1,y_1),(\boldsymbol{x}_2,y_2),...,(\boldsymbol{x}_m,y_m)\}$，其中 $\boldsymbol{x}=(\boldsymbol{x}_{i1};\boldsymbol{x}_{i2}; ... ;\boldsymbol{x}_{id})，y_i \in \mathbb{R}$

线性回归试图学得 $f(x_i) = w^T x_i + b$ 使得 $f(x_i) \simeq y_i$，即为学习

$$\begin{split}(w^*,b^*) &= \arg\min_{(w,b)} \sum_{i=1}^m (f(x_i)-y_i)^2 \\\ &= \arg\min_{(w,b)} \sum_{i=1}^m (y_i-wx_i-b)^2\end{split}$$

#### 最小二乘法

最小二乘法可以实现对 $w, b$ 分别求偏导，得到 $w, b$ 最优解的闭式解（此处假设x只有一维）

$$ w = \frac{\sum_{i=1}^{m}y_i(x_i - \bar{x})}{\sqrt{m\sum_{i=1}^{m}x_i^2 - \left(\sum_{i=1}^{m}x_i\right)^2}}$$

$$ b = \frac{1}{m} \sum_{i=1}^m (y_i - w_ix_i)$$

#### 多元线性回归（一般情况）

若对于数据集 $D$，样本由 $d$ 个属性描述，仍可以利用最小二乘法思路求解。为方便，作出如下规定：

1. 假设最后的 b 是样本的一个特征产生的，但因为对于所有样本保持一致，因此定为 1

\\[X = \begin{pmatrix} x_{11} & x_{12} & \dots & x_{1d} & 1 \\\ x_{21} & x_{22} & \dots & x_{2d} & 1 \\\ \vdots & \vdots & \ddots & \vdots & \vdots \\\ x_{m1} & x_{m2} & \dots & x_{md} & 1 \end{pmatrix} = \begin{pmatrix} \mathbf{x}_1^T & 1 \\\ \mathbf{x}_2^T & 1 \\\ \vdots & \vdots \\\ \mathbf{x}_m^T & 1 \end{pmatrix} \\]

2. 相应的将 $b$ 融入 $\boldsymbol{w}$ 权重矩阵构成 $\boldsymbol{\hat{w}} = (\boldsymbol{w};b)$，标记也表示为 $\boldsymbol{y} = (y_1;y_2;...;y_m)$

3. 可得最小化目标，及求解结果如下：

$$ w^* = \arg \min_{\boldsymbol{\hat{w}}} (\boldsymbol{y} – \boldsymbol{X} \boldsymbol{\hat{w}})^T(\boldsymbol{y} – \boldsymbol{X} \boldsymbol{\hat{w}})$$

$$ \frac{\partial E(\boldsymbol{\hat{w}})}{\partial \boldsymbol{\hat{w}}} = 2 \boldsymbol{X}^T ( \boldsymbol{X} \boldsymbol{\hat{w}} – \boldsymbol{y}) $$

4. 求解闭式解时涉及逆矩阵运算，因此当 $\boldsymbol{X}^T\boldsymbol{X}$ 满秩时可以求解得到

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

+ 同类投影点尽可能近：同类投影点的协方差尽可能小（ $\boldsymbol{w}^{T}\boldsymbol{\Sigma} _{0}\boldsymbol{w}+\boldsymbol{w}^{T}\boldsymbol{\Sigma}_{1}\boldsymbol{w}$ ）
+ 异类投影点尽可能远离：类中心之间的距离尽可能大（ $\left\Vert \boldsymbol{w}^{T}\boldsymbol{\mu}_{0}-\boldsymbol{w}^{T}\boldsymbol{\mu}_{1}\right\Vert ^{2}$ ）
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

