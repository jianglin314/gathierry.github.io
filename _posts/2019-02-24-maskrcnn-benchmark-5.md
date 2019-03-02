---
layout: post
title: 拆解 MaskRCNN-Benchmark 之五 —— Mask
categories: [data science]
tags: [object detection, instance segmentation, deep learning, computer vision]
description: 这篇文章介绍 FAIR 项目 maskrcnn-benchmark 模型中的 mask head 部分。
---
RCNN 的检测结果，会作为 proposals 输入到 mask head 中。

## Mask head
如果是训练过程，输出的检测结果同时包含了正/负样本，需要先滤去负样本，只保留正样本。

对于 mask 分支，可以选择是否与 rcnn 共享 feature extractor。如果共享，就是直接使用 rcnn feature extractor 的输出，也就是 RoI align 后两次全连接的特征。不过在官方实现中，是不 share 的，mask 分支经过 RoI align 后再通过 4 个```kernel=256```的```conv3x3 + Relu```。提取的特征经过一个转置卷积上采样到原来边长的两倍，再经过```conv1x1```输出```num_class```个 map 作为 mask。

## 计算 Loss

训练阶段，要将 target 中的 mask 转化成与输出一样的形式比对求交叉熵 loss。

annotation 中给出的 mask 是多边形的形式```[x0,y0,x1,y1,x2,y2,...]```，以图片左上角为参考系原点。先将多边形的坐标转换成以 box 左上角为参照系原点的坐标，之后```resize```到 mask 输出的大小（代码中是```28x28```），最后通过 coco 提供的解码工具把多边形转换成```28x28```的```tensor```。

## Mask 后处理

在推断阶段，mask 要同样要进行后处理。
与求 loss 时相反，这一次是要将输出的 mask 恢复到原图里。先将 mask bilinear resize 到 box 长宽的比例，再根据 box 左上角的点还原出 mask 在原图中的位置，最后通过阈值将 mask prob 中低于阈值的部分过滤掉。