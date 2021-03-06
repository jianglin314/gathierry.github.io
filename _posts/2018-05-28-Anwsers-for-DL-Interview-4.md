---
layout: post
title: 深度学习面试你必须知道这些答案（四）
categories: [data science]
tags: [deep learning]
description: 本文节选了网上分享的一部分深度学习相关的面试题，并将搜集到的答案随之附上。
---
本文节选了网上分享的一部分深度学习相关的面试题，并将搜集到的答案随之附上。  
参考了来自 _https://cloud.tencent.com/developer/article/1064505_ 的内容。

#### Q31. 共享参数的概念及在深度学习中的广泛影响 

- 参数范数惩罚是正则化参数使其彼此接近的一种方式，而更流行的方法是使用约束：强迫某些参数相等，这种正则化方法通常被称为**参数共享**。
- 参数共享的一个显著优点是，只有参数的子集需要被存储在内存中，可以显著减少模型所占用的内存。
- 对于卷积神经网络，参数共享的特殊形式使得神经网络层具有对平移**等变**的性质。

#### Q32. Dropout 与 Bagging 集成方法的关系，以及 Dropout 带来的意义与其强大的原因

- Bagging（bootstrap aggregating）是通过结合几个模型降低泛化误差的技术 (Breiman, 1994)。主要想法是分别训练几个不同的模型，然后让所有模型表决测试样例的输出。其奏效的原因是不同的模型通常不会在测试集上产生完全相同的误差。在误差完全相关的情况下，平方误差的期望不会减少，但在误差完全不相关的情况下，平方误差期望减少为原来的  1/k，k是模型的数量。
- Dropout 可以被看做是集成了大型神经网络的子网络。
- 只有极少的训练样本可用时，Dropout不会很有效。
- 在推断时，每一个输出都要乘以权值被保留的概率。通过掩码 $$\mu$$ 定义每个子模型的概率分布
\\[p(y|x, \mu)\\]
输出取算术平均值 \\[\sum_\mu p(\mu)p(y|x, \mu)\\]

#### Q33. 批量梯度下降法更新过程中，批量的大小与各种更新的稳定性关系  

- 更大的批量会计算更精确的梯度估计，但是回报却是小于线性的。n 个样本均值的标准差是 $$\sigma/\sqrt(n)$$ ，$$\sigma$$ 是样本值真实标准差。
- 可能是由于小批量在学习过程中加入了噪声，它们会有一些正则化效果(Wilson and Martinez, 2003)。泛化误差通常在批量大小为1 时最好。因为梯度估计的高方差，小批量训练需要较小的学习率以保持稳定性，会进一步增加训练时间。
- 仅基于梯度 $$g$$ 的更新方法通常相对鲁棒，并能使用较小的批量获得成功。使用 Hessian 矩阵的二阶方法通常需要更大的批量。这些大批量需要最小化估计二阶项 $$H^{-1}g$$ 的波动。
- 小批量是随机抽取的这点也很重要。从一组样本中计算出梯度期望的无偏估计要求这些样本是独立的。


#### Q34. 如何避免深度学习中的病态，鞍点，梯度爆炸，梯度弥散 

- 牛顿法在解决带有病态条件的Hessian 矩阵的凸优化问题时，是一个非常优秀的工具，但是牛顿法运用到神经网络时需要很大的改动。
- 克服鞍点同样需要修改牛顿法（加动量）
- 关于梯度爆炸和梯度弥散，参考 Q20

#### Q35. SGD 以及学习率的选择方法，带动量的 SGD 对于 Hessian 矩阵病态条件及随机梯度方差的影响 

- 通常最好是检测最早的几轮迭代，选择一个比在效果上表现最佳的学习率更大的学习率，但又不能太大导致严重的震荡。
- 随机梯度下降学习过程有时会很慢。动量方法 (Polyak, 1964) 旨在加速学习，特别是处理高曲率、小但一致的梯度，或是带噪声的梯度。动量算法积累了之前梯度指数级衰减的移动平均，并且继续沿该方向移动。
- 一个病态条件的二次目标函数看起来像一个长而窄的山谷或具有陡峭边的峡谷。动量正确地纵向穿过峡谷，而普通的梯度步骤则会浪费时间在峡谷的窄轴上来回移动。
- 动量算法引入了变量 $$\boldsymbol{v}$$ 充当速度角色，相比于 SGD 中直接 $$\boldsymbol{\theta} = \boldsymbol{\theta} - \epsilon \boldsymbol{g}$$，带动量的 SGD 先更新速度 $$\boldsymbol{v}=\alpha \boldsymbol{v} - \epsilon \boldsymbol{g}$$，再更新参数 $$\boldsymbol{\theta}=\boldsymbol{\theta}+\boldsymbol{v}$$。当许多连续的梯度指向相同的方向时，步长最大。如果动量算法总是观测到梯度 $$\boldsymbol{g}$$，那么它会在方向$$-\boldsymbol{g}$$ 上不停加速，直到达到最终速度，其中步长大小为 
\\[\frac{\epsilon ||\boldsymbol{g}||}{1-\alpha}\\]
因此将动量的超参数视为 $$\frac{1}{1-\alpha}$$ 有助于理解。例如，$$\alpha=0.9$$ 对应着最大速度10 倍于梯度下降算法。在实践中，$$\alpha$$ 的一般取值为0.5，0.9 和0.99。之前梯度对现在方向产生影响，抑制了随机梯度的方差。
- Nesterov 动量，在计算梯度 $$\frac{1}{m}\nabla _\theta \sum_iL(f(x, \theta), y)​$$ 之前更新参数，变成 $$\frac{1}{m}\nabla _\theta \sum_iL(f(x, \theta+\alpha v), y)​$$。在凸批量梯度的情况下，Nesterov 动量将额外误差收敛率从O(1/k)（k 步后）改进到O(1/k2)。可惜，在随机梯度的情况下，Nesterov 动量没有改进收敛率。

#### Q36. 初始化权重过程中，权重大小在各种网络结构中的影响，以及一些初始化的方法；偏置的初始化

- 更大的初始权重具有更强的破坏对称性的作用，有助于避免冗余的单元，也有助于避免在每层线性成分的前向或反向传播中丢失信号——矩阵中更大的值在矩阵乘法中有更大的输出。如果初始权重太大，那么会在前向传播或反向传播中产生爆炸的值。在循环网络中，很大的权重也可能导致混沌 (chaos)(对于输入中很小的扰动非常敏感，导致确定性前向传播过程表现随机)。较大的权重也会产生使得激活函数饱和的值，导致饱和单元的梯度完全丢失。这些竞争因素决定了权重的理想初始大小。
- Xavier 均匀初始化 $$W_{i,j}=\mathcal{U}(-\sqrt{\frac{6}{m+n}}, \sqrt{\frac{6}{m+n}})$$。 m, n 分别是输入和输出的单元数。Xavier 初始化目的是使每一层输出的方差尽量相等，但推导时只考虑了激活函数是线性函数的情况。
- MSRA 初始化  $$W_{i,j}=\mathcal{N}(0, \sqrt{\frac{2}{m}})$$
- 设置偏置为零通常在大多数权重初始化方案中是可行的。存在一些我们可能设置偏置为非零值的情况:
  - 如果偏置是作为输出单元，那么初始化偏置以获取正确的输出边缘统计通常是有利的。
  - 有时，我们可能想要选择偏置以避免初始化引起太大饱和。
  - 有时，一个单元会控制其他单元能否参与到等式中。例如 LSTM

#### Q37. 自适应学习率算法: AdaGrad，RMSProp，Adam 等算法的做法

#### Q38. 二阶近似方法: 牛顿法，共轭梯度，BFGS 等的做法

#### Q39. Hessian 的标准化对于高阶优化算法的意义

#### Q40. 卷积网络中的平移等变性的原因，常见的一些卷积形式
对于卷积，参数共享的特殊形式使得神经网络层具有对平移等变的性质。卷积包含的参数有：kernel size, stride, padding, dilation