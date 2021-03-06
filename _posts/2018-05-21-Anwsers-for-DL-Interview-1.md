---
layout: post
title: 深度学习面试你必须知道这些答案（一）
categories: [data science]
tags: [deep learning]
description: 本文节选了网上分享的一部分深度学习相关的面试题，并将搜集到的答案随之附上。
---
本文节选了网上分享的一部分深度学习相关的面试题，并将搜集到的答案随之附上。  
参考了来自 _https://cloud.tencent.com/developer/article/1064505_ 的内容。

#### Q1. 列举常见的一些范数及其应用场景，如 $$L^0$$，$$L^1$$，$$L^2$$，$$L^\infty$$，Frobenius 范数
- 范数用于测量一个向量。比如 $$\boldsymbol{x}$$ 的 $$L^p$$ 范数记为
$$||\boldsymbol{x}||_p=(\sum_i|x_i|^p)^{\frac{1}{p}}$$ 其中 $$p\in\mathbb{R}$$, $$p\geq1$$
- $$L^\infty$$范数，
\\[||\boldsymbol{x}||_{\infty}=max_i |x_i|\\]
- $$L^0$$范数表示向量中非零元素的个数  
- Frobenius范数用于测量一个矩阵
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
<img src="/images/2018-05-21-Anwsers-for-DL-Interview-1/softplus.png" width="400px"/>

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
$$
H(P,Q)=H(P)+D_{KL}(P||Q)=\mathbb{E}_{x \sim P}[\log{P(x)}] + D_{KL}(P||Q) = -\mathbb{E}_{x \sim P}[\log{Q(x)}]
$$
- 深度学习里的 Cross entropy loss 中 Q 表示预测值，P 表示真实值，Cross entropy 与 KL 散度相差一个常数。

#### Q7. 数值计算中的计算上溢与下溢问题，如 softmax 中的处理方式
- 当接近零的数被四舍五入为零时发生**下溢**。许多函数在其参数为零而不是一个很小的正数时才会表现出质的不同。
- 当大量级的数被近似为 $$\infty$$ 或 $$-\infty$$ 时发生**上溢**。
- 对于softmax函数$$
softmax(\boldsymbol{x})_i=\frac{\exp(x_i)}{\sum_{j=1}^n\exp(x_j)}
$$
当所有$$x_i$$都等于一个很小的负数时，分母为0；当所有$$x_i$$都是非常大的正数时，会发生上溢。解决方法，是令$$
softmax(\boldsymbol{x})_i=\frac{\exp(x_i-\max_k x_k)}{\sum_{j=1}^n\exp(x_j-\max_k x_k)}
$$

#### Q8. 与矩阵的特征值相关联的条件数（病态条件）指什么，与梯度爆炸与梯度弥散的关系
考虑函数$$f(\boldsymbol{x})=\boldsymbol{A}^{-1}\boldsymbol{x}$$当$$\boldsymbol{A}$$具有特征值分解时，其条件数为$$max_{i,j}|\frac{\lambda_i}{\lambda_j}|$$，即分子是最大的特征值，分母是最小的特征值，当该数很大时，矩阵求逆对输入的误差特别敏感，输入中的舍入误差可能导致输出的巨大变化。

#### Q9. 在基于梯度的优化问题中，如何判断一个梯度为 0 的零界点为局部极大值、全局极小值还是鞍点，Hessian 矩阵的条件数与梯度下降法的关系
- 梯度为 0 时，当 Hessian 矩阵是正定的，是一个局部极小点。当 Hessian 矩阵是负定的 时，是一个局部极大点。这就是所谓的二阶导数测试。如果 Hessian 的特征值中至少一个是正的且至少一个是负的，那么是某个横截面 的局部极大点，却是另一个横截面的局部极小点。
- 可以使用 Hessian 矩阵的信息来指导搜索。牛顿法基于一个二阶泰勒展开来近似 $$x_0$$ 附近的 $$f(x)$$
\\[
f(x) = f(x_0) + (x - x_0)^T\nabla_xf(x_0)+\frac{1}{2}(x - x_0)^TH(f)(x - x_0)
\\]
计算 $$f'(x)=0$$ 可得到函数的临界点
\\[
x^*=x_0 - H(f)(x_0)^{-1} \nabla_xf(x_0)
\\]
当 f 是一个正定二次函数时，牛顿法只要应用上面的一次式就能直接跳到函数的最小点。如果 f 不是一个真正二次但能在局部近似为正定二次，牛顿法则需要多次迭代上式。迭代地更新近似函数和跳到近似函数的最小点可以比梯度下降更快地到达临界点。这在接近局部极小点时是一个特别有用的性质，但是在鞍点附近 是有害的。当附近的临界点是最小点(Hessian 正定)时牛顿法才适用，这时梯度下降不会被吸引到鞍点(除非梯度指向鞍点)。

#### Q10. KTT 方法与约束优化问题，活跃约束的定义
部分参考自 (http://www.cnblogs.com/mo-wang/p/4775548.html)  
对于一个优化问题
\\[
\min{f(x)}
\\]

- 如果约束是等式
\\[
h_k(x)=0 \quad k=1,2,...,l
\\]
可以使用拉格朗日乘子法，定义拉格朗日函数
\\[
F(x,\lambda)=f(x)+\sum_{k=1}^{l}\lambda_kh_k(x)
\\]
通过解方程组
\\[
\frac{\partial F}{\partial x_i}=0 \\
...\\
\frac{\partial F}{\partial \lambda_k}=0
\\]
计算出最优解。

- 如果约束是不等式
\\[
h_j(x)=0 \quad j=1,2,...,p \\
g_k(x)\leq 0 \quad k=1,2,...,q
\\]
此时的广义拉格朗日函数可以定义为
\\[
L(x,\lambda, \mu)=f(x) + \sum_{j=1}^{p}\lambda_jh_j(x) + \sum_{k=1}^{q}\mu_kg_k(x)
\\]
KTT 条件是最优点的必要条件，不一定是充分条件：
  - L 梯度为 0
  - h(x)=0
  -  不等式约束显示的“互补松弛性”：$$\sum_{k=1}^{q}\mu_kg_k(x)=0$$
如果 $$g_k(x^*)=0$$ 那么称这个约束是**活跃的**。

