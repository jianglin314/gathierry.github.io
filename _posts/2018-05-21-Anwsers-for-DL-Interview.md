---
layout: post
title: 深度学习面试你必须知道这些答案
categories: [data science]
tags: [deep learning]
description: 本文节选了网上分享的一部分深度学习相关的面试题，并将搜集到的答案随之附上。
---
本文节选了网上分享的一部分深度学习相关的面试题，并将搜集到的答案随之附上。  
参考了来自 _https://cloud.tencent.com/developer/article/1064505_ 的内容。

#### Q1. 列举常见的一些范数及其应用场景，如 $$L^0$$，$$L^1$$，$$L^2$$，$$L^\infty$$，Frobenius 范数
- 范数用于测量一个向量。比如 $$\boldsymbol{x}$$ 的 $$L^p$$ 范数记为
\\[
||\boldsymbol{x}||_p=(\sum_i|x_i|^p)^{\frac{1}{p}} \quad p\in\mathbb{R}, p\geq1
\\]
\\[
||\boldsymbol{x}||_\infty=max_i |x_i|
\\]

- $$L^0$$范数表示向量中非零元素的个数  
Frobenius范数用于测量一个矩阵
\\[
||\boldsymbol{A_F}||=\sqrt{\sum_{i,j}A^2_{i,j}}
\\]  
- $$L^1$$，$$L^2$$范数常用在正则化中，加入正则项会改变目标函数的优化路径。$$L^2$$也被成为“岭回归”，能让学习算法“感知”到具有较高方差的输入 $$\boldsymbol{x}$$ 因此相对增加输出方差的特征的权重将会收缩。$$L^1$$也被成为“LASSO回归”，会产生更稀疏的解，使得部分特征参数为0，表明相应特征可以被安全忽略。

#### Q2. 简单介绍一下贝叶斯概率与频率派概率，以及在统计中对于真实参数的假设
直接与事件发生的频率相联系（如抽取扑克牌），被称为频率派概率；而涉及到确定性水平（如病人诊断为阳性），被称为贝叶斯概率。

#### Q3. 概率密度的万能近似器
- 高斯混合模型是概率密度的万能近似器，任何平滑的概率密度都可以用具有足够多组件的高
斯混合模型以任意精度来逼近。  
- 一个高斯混合模型包含了一组参数不同的高斯分布，参数包括均值 $$\mu^(i)$$，协方差矩阵 $$\Sigma^(i)$$，以及每个组件 $$i$$ 的先验概率 $$\alpha_i=P(c=i)$$。

#### Q4. 简单介绍一下 sigmoid，relu，softplus，tanh，RBF 及其应用场景
- sigmoid
\\[
\sigma(x)=\frac{1}{1+\exp(-x)}
\\]
\\[
\sigma(x)’=\sigma(x)(1-\sigma(x))
\\]

- softplus
\\[
\zeta(x)=\log(1+\exp(x))
\\]
<img src="/images/2018-05-21-Anwsers-for-DL-Interview/softplus.png" width="400px"/>

#### Q5. Jacobian，Hessian 矩阵及其在深度学习中的重要性
仅使用梯度信息的优化算法被称为一阶优化算法，如梯度下降。使用 Hessian 矩阵的优化算法被称为二阶最优化算法，如牛顿法。

#### Q6. KL 散度在信息论中度量的是那个直观量
- 如果我们对于同一个随机变量 有两个单独的概率分布 $$P(x)$$ 和 $$Q(x)$$，我们可以使用 KL 散度来衡量这两个分布的差异。
\\[
D_{KL}(P||Q)=\mathbb{E}_{x \sim P}[\log{P(x)} - \log{Q(x)}]
\\]
- KL 散度有很多有用的性质，最重要的是它是非负的。KL 散度为 0 当且仅当 P 和 Q 在离散型变量的情况下是相同的分布，或者在连续型变量的情况下是“几乎处处”相同的。  
它并不是真的距离因为它不是对称的。$$D_{KL}(P||Q)\neq D_{KL}(Q||P)$$  
- 一个和 KL 散度密切联系的量是交叉熵
\\[
H(P,Q)=H(P)+D_{KL}(P||Q)=\mathbb{E}_{x \sim P}[\log{P(x)}] + D_{KL}(P||Q) = -\mathbb{E}_{x \sim P}[\log{Q(x)}]
\\]
深度学习里的 Cross entropy loss 中 Q 表示预测值，P 表示真实值，Cross entropy 与 KL 散度相差一个常数。