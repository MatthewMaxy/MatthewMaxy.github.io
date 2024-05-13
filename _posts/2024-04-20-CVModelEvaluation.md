---
layout: post
title:  "模型评价指标总结（一）"
date:   2024-04-20 16:13:00 +0800
tags:   CV Basic
color: rgb(255,102,94)
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

## 目标检测领域

### AP 和 mAP

#### 关于 top1 和 top5 准确度

top1：预测的概率向量里面最大的类别作为预测结果，该类必须是正确类别才算预测正确

top5：预测的概率向量最大的前五名中出现了正确的类别，即为预测正确。

#### PR 曲线（注意区分于roc曲线）

k 个样本按照置信度分数降序排列，分别将每个假设为正类然后计算当前情况下的 Recall 和 Precious。以这 k 组（R，P）数据绘制 Recall 值为横轴，Precision 值为纵轴的 PR 曲线

![pr](/assets/Blogs/CVModelEvaluation/1.png){:height="50%" width="50%"} ![pr](/assets/Blogs/CVModelEvaluation/2.png){:height="50%" width="50%"}

#### AP（Average Precision）和 mAP

AP为平均精度，使用积分的方式来计算PR曲线与坐标轴围成的面积。但是在实际应用中，对其平滑操作来简化计算，对PR曲线上的每个点，Precision的值取该点右侧最大的Precision的值

对所有类别的 **AP** 值取平均即可得出整个数据集的 **mAP**

#### Interplolated AP（Pascal Voc 2008 的AP计算方式）

Pascal VOC 2008 中设置IoU的阈值为 0.5，即对于一个样本若被预测为正类有以下两种情况为FP：（1）IOU 小于阈值（2）此目标被重复检测且该样本IOU不是最大的

对平滑后的PR曲线进行间隔 0.1 的均匀采样，取这11个点的 Precision 值（平滑后），计算其平均值为最终AP的值

#### MS COCO mAP

对于 COCO 数据集来说，使用的也是 Interplolated AP 的计算方式，但为了提高精度，在PR曲线上采样了 100 个点进行计算。而且Iou的阈值从 0.5 - 0.95 的区间上每隔 0.05 计算一次 AP 的值，取所有结果的平均值作为最终的结果。除了AP，还有其他衡量指标值，含义如下：

+ $AP_{50}$：IoU 阈值为 0.5 时的 AP 测量值
+ $AP_{75}$：IoU阈值为0.75时的 AP 测量值
+ $AP_{S}$：: 像素面积小于 $32^2$ 的目标框的 AP 测量值（小尺度）
+ $AP_{M}$：像素面积在 $32^2 - 96^2$ 之间目标框的 AP 测量值（中等尺度）
+ $AP_{L}$：像素面积大于 $96^2$ 的目标框的 AP 测量值（大尺度）

注意：COCO 语境下的 AP 已经对所有类别平均，即为一般意义的 mAP

## 图像生成领域

### Inception Score（IS）

评价一个生成模型的好坏，需要从两方面考量：

+ 生成的图像是否清晰，清晰度高的表示生成图像的质量高；
+ 生成的图像是否具有多样性，即每个类别生成图像的数目尽可能相等。

#### 图片质量评价

引入 Inception 模型，对生成图像 $x$ 进行分类判断，得到一个 1000 维的分类向量 $y$。如果图像 $x$ 清晰（质量高），则该向量 $y$ 在某一个维度上的取值较大（表示属于某一类的概率较大），而其他维度的取值较小

即一张高质量图片在类别上的熵应该很小，即 $p(y ｜ x)$ 应该较小

#### 图片多样性

在图像的多样性方面，假设我们生成20000张图像，总共有1000个类别，那么最理想的情况是每一类的个数是平均的

即对于生成图像在所有类别的边缘分布 $p(y)$ 的熵应该越大（最理想是均匀分布）。

#### IS 计算公式

综合以上两个方面，IS公式如下

$$\text{IS(G)} = \text{exp}(\text{E}_{x~\text{P}_g} \text{KL}(p(y|x)||p(y)))$$

+ 图像 $x$ 是从生成数据的分布 $P_g$ 中采样的
+ KL 表示 KL 散度
+ 对于 $p(y \vert x)$ 期望接近one-hot，而 $p(y)$ 接近均匀分布，因此 KL 散度值越大模型效果越好

#### 缺点

+ Inception分类器在ImageNet1k上训练，因此生成模型也必须在此数据集训练，否则在分类上存在问题
+ 神经网络中权值的细节改变可能很大的影响IS分数
+ Inception Score高的图片不一定真实；且如果给一个不在1000类中的真实图，IS值可能很高
+ 检验多样性时，无法检测每个类别内图片一样存在的问题
+ 过拟合拷贝源数据集无法检测

### Frechet Inception Distance（FID）

FID 指标用于度量生成的图像与真实图像之间的差异，本质在于表示生成图像的特征向量与真实图像的特征向量之间的距离。该距离越小，表明生成模型的效果越好，即图像的清晰度高，且多样性丰富

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

+ 使用预训练的Inception来提取高维特征向量，捕捉了图像的语义信息（也避免了IS中依赖类别问题）
+ 敏感度较高：FID对生成图像与真实图像之间的差异具有较高的敏感度
  
**缺点：**

+ 需要预训练的模型：可能增加了计算成本和实施的复杂性
+ 受限于特征提取器的限制：使用Inception来捕捉图像的语义信息，但这个特征提取器可能不适用于所有类型的图像生成任务
+ FID是衡量多元正态分布直接按的距离，但提取的图片特征不一定是符合多元正态分布的
+ 过拟合拷贝源数据集无法检测

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

### CLIP Score

原文链接：[CLIP Score](https://aclanthology.org/2021.emnlp-main.595v2.pdf)

CLIP score是一种用于评估 text2img 或者 img2img，模型生成的图像与原文本（prompt text）或者原图关联度大小的指标。

将文本和图像对 (如为 text2img ) 输入到 OpenAI 的CLIP（Contrastive Language-Image Pre-training）模型后分别转换为特征向量，然后计算它们之间的余弦相似度。
