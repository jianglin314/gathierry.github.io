---
layout: post
title: 拆解 MaskRCNN-Benchmark（六）—— Mask
categories: [data science]
tags: [object detection, instance segmentation, deep learning, computer vision]
description: 这篇文章介绍 FAIR 项目 maskrcnn-benchmark 模型中的 mask head 部分。
---
RCNN 的检测结果，会作为 proposals 输入到 mask head 中。如果是训练过程，输出的检测结果同时包含了正/负样本，需要先滤去负样本，只保留正样本。

对于 mask 分支，可以选择是否与 rcnn 共享 feature extractor。如果共享，就是直接使用 rcnn 中 RoI align 后两次全连接的特征。在官方实现中，是不 share 的，mask 分支经过 RoI align 后再通过四个```kernel=256```的```conv3x3 + Relu```。提取的特征经过一个转置卷积上采样到原来边长的两倍，再经过```conv1x1```输出```num_class```个 map 作为 mask。

训练阶段，要将 target 中的 mask 转化成与输出一样的形式比对求交叉熵 loss。先根据 box 对 mask 进行修剪，之后```resize```到 mask 输出的大小。（代码中是```28x28```）。

在推断阶段，mask 要同样要进行后处理。与求 loss 时相反，这一次是要将输出的 mask 恢复到原图里。先将 mask bilinear resize 到 box 长宽的比例，在平移到 box 所在坐标。通过阈值将 mask prob 中低于阈值的部分过滤掉。