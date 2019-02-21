---
layout: post
title: 拆解 MaskRCNN-Benchmark（三）
categories: [data science]
tags: [object detection, instance segmentation, deep learning, computer vision]
description: 这篇文章简单介绍 FAIR 项目 maskrcnn-benchmark 模型的 backbone 部分。
---
项目中的 backbone 用的是经过 ImageNet 预训练的 ResNet 模型。但是存在两个区别：

## 1. 固定 Batch Normalization  
在代码中，有一个重新实现的 BN 层```maskrcnn_benchmark/layers/batch_norm.py```，这里的参数全部是固定的初始值，不会随着训练而更新。固定 BN 的原因，在[这里](https://github.com/facebookresearch/maskrcnn-benchmark/issues/267)也给出了解释，是因为在 batch size per GPU 比较小的情况下，如果不采用多 GPU 同步，BN 达不到应有的效果，所以干脆使用固定的原 pretrained 模型的值。

## 2. 参数初始化
另一个和 torchvision 的 resnet 实现不同的地方是参数初始化的方法。在 maskrcnn-benchmark 的 ResNet 和 FPN 中，所有的初始化都使用的 [```nn.init.kaiming_uniform_(conv.weight, a=1)```](https://pytorch.org/docs/0.3.1/nn.html#torch.nn.init.kaiming_uniform)