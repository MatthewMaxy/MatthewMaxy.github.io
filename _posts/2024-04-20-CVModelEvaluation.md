---
layout: post
title:  "模型评价指标总结（一）"
date:   2024-04-20 16:13:00 +0800
tags:   ComputerVision Evaluation
color: rgb(66,118,91)
cover: '../assets/Blogs/CVModelEvaluation/title.png'
subtitle: '计算机视觉领域各任务模型评价指标'
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

## 图像生成领域

### Frechet Inception Distance（FID）

FID 指标用于度量生成的图像与真实图像之间的差异，本质在于表示生成图像的特征向量与真实图像的特征向量之间的距离。该距离越小，表明生成模型的效果越好，即图像的清晰度高，且多样性丰富。

#### 特征提取

使用预训练的 Inception 模型，去除全连接层后用于特征提取。

假设有真实图像和生成图像各 $N$ 张，经过特征提取各自得到 $N \times 2048$ 维的特征向量，分别记为 $g, r$

#### Frechet 距离计算

利用特征向量 $g, r$ 的相关指标计算 Frechet 距离，得出 FID 值

$$ \text{FID}(g,r) = \lVert \mu _g - \mu_r \rVert_2 ^2 + \text{Tr}(\Sigma_g + \Sigma_r - 2(\Sigma_g \Sigma_r)^{\frac{1}{2}}) $$

+ $\mu_r, \mu_g$ 表示各自特征向量的均值
+ $\Sigma_g, \Sigma_r$ 表示各自特征向量的协方差矩阵
+ Tr表示矩阵的迹
+ 矩阵开根如果为复数，则只取实部

#### 优缺点分析

**优点：**

+ 考虑了图像的语义信息：使用预训练的Inception来提取高维特征向量，捕捉了图像的语义信息
+ 敏感度较高：FID对生成图像与真实图像之间的差异具有较高的敏感度
+ 可解释性较强：基于统计的特征分析，提供了对生成图像与真实图像之间差异的可解释性
  
**缺点：**

+ 需要预训练的模型：可能增加了计算成本和实施的复杂性。
+ 受限于特征提取器的限制：使用Inception来捕捉图像的语义信息，但这个特征提取器可能不适用于所有类型的图像生成任务
+ 不考虑多模态分布：基于单个协方差矩阵来比较两个数据集的分布，假设数据集的分布是单一的。然而，真实图像集和生成图像集可能具有多个模态或不同的分布特征，FID可能无法很好地捕捉这种多样性

#### 具体实现

```python
# act1(2).size = N * 2048 from inception feature extractor 
def calculate_fid(act1, act2):
    mu1, sigma1 = act1.mean(axis=0), np.cov(act1, rowvar=False)
    mu2, sigma2 = act2.mean(axis=0), np.cov(act2, rowvar=False)

    ssdiff = np.sum((mu1 - mu2)**2.0)
    covmean = linalg.sqrtm(sigma1.dot(sigma2))

    # check and correct imaginary numbers from sqrt
    if np.iscomplexobj(covmean):
        covmean = covmean.real

    fid = ssdiff + np.trace(sigma1 + sigma2 - 2.0 * covmean)

    return fid

```