---
layout: post
title: 拆解 MaskRCNN-Benchmark（一）
categories: [Data science]
tags: [object detection, deep learning, computer vision]
description: 这篇文章简单介绍 FAIR 项目 maskrcnn-benchmark 安装配置和整个数据流的结构。
---
[maskrcnn-benchmark](https://github.com/facebookresearch/maskrcnn-benchmark) 是继 Detectron 之后 FAIR 用 Pytorch1.0 最新的实现，在效率上超越了 Detectron。通过阅读、实验代码，可以对 Mask RCNN 这一多任务模型的诸多细节有更深刻的认识。

项目的安装方法在[安装文档](https://github.com/facebookresearch/maskrcnn-benchmark/blob/master/INSTALL.md)写的很详细，这里不再赘述。

程序的训练/测试入口是```tools/train_net.py```和```tools/test_net.py```。运行的变量包括要使用的```config```文件，这里我们用```configs/e2e_mask_rcnn_R_50_FPN_1x.yaml```为例。

整个流程大概可以用这个框图表示，红色部分表示训练过程，蓝色部分表示测试过程，黑色表示共用部分。
<img src="/images/2019-02-17-maskrcnn-benchmark-1/maskrcnn-benchmark.png" width="600px"/>

在后序文章中将会结合代码，依次分析其中的细节。