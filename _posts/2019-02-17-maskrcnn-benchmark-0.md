---
layout: post
title: 拆解 MaskRCNN-Benchmark 之〇 —— 写在前面
categories: [data science]
tags: [object detection, instance segmentation, deep learning, computer vision]
description: 这篇文章介绍 FAIR 项目 maskrcnn-benchmark 安装配置和整个模型的结构。
---
[maskrcnn-benchmark](https://github.com/facebookresearch/maskrcnn-benchmark) 是继 Detectron 之后 FAIR 用 Pytorch1.0 最新的实现，在效率上超越了 Detectron。通过阅读、实验代码，可以对 Mask RCNN 这一多任务模型的诸多细节有更深刻的认识。

**这个系列的文章是我自己阅读代码时做的分析，便于理解工程的结构和思路，并不会去解释 Mask RCNN 的论文，也不会逐行分析每句代码的作用，如果你已经读过论文，但是在工程实现上对一些细节还有疑问，那么希望这篇文章能帮到你。**

## 安装方法

项目的安装方法在[安装文档](https://github.com/facebookresearch/maskrcnn-benchmark/blob/master/INSTALL.md)写的很详细，这里就不多啰嗦了。

文档中要求使用 nighlty release 版本的 PyTorch 1.0，事实上 stable 版也是可以的，不过一些运算的速度可能会降低。

在最后用 ```setup.py``` 安装项目的时候，Mac 用户要多往下看一行，使用最后一行被注释掉的安装命令

	MACOSX_DEPLOYMENT_TARGET=10.9 CC=clang CXX=clang++ python setup.py build develop

## 代码结构

程序的训练/测试入口是```tools/train_net.py```和```tools/test_net.py```。

```configs```文件夹中包含了不同网络配置的参数，这里我们用```configs/e2e_mask_rcnn_R_50_FPN_1x.yaml```为例。

```maskrcnn_benchmark```目录下包含了数据处理、网络运算等核心代码，也是接下来的重点所在。

## 网络流程

粗略梳理一下网络结构，可以发现，整个流程大概可以用这个框图表示，红色部分表示训练过程，蓝色部分表示测试过程，黑色表示共用部分。
<img src="/images/2019-02-17-maskrcnn-benchmark-0/maskrcnn-benchmark.png" width="1000px"/>

在后序文章中将会结合代码，依次分析其中的细节。