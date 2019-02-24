---
layout: post
title: 拆解 MaskRCNN-Benchmark（五）—— RCNN
categories: [data science]
tags: [object detection, instance segmentation, deep learning, computer vision]
description: 这篇文章介绍 FAIR 项目 maskrcnn-benchmark 模型中的 RCNN head 部分。
---
通过 FPN 和 RPN head，我们已经得到了从原图中提取的五组 feature 以及 proposals。通过RoI align 参考 proposal 从 feature map 中提取 RoI 的特征。

项目中实现了三种 feature extractor，这里以```FPN2MLPFeatureExtractor```为例。RoI align 的输出结果经过 FC 转化成```ROI_BOX_HEAD.MLP_HEAD_DIM=1024```维，之后再经过一个 FC 保持维度不变。这个输出连接出两个全连接分支，一个输出```num_class```个神经元，一个输出```4*num_class```个神经元，分别作为分类和回归的结果。

如果是推断过程，还需要进行后处理。后处理包含三个任务，一是将 score 小于固定阈值```ROI_HEADS.SCORE_THRESH=0.05```的框删掉。二是对每一类进行 NMS，阈值取```ROI_HEADS.NMS=0.5```。三是将输出框的数量限制在```ROI_HEADS.DETECTIONS_PER_IMG=100```以内，这一步是将所有类合在一起通过 score 排序得到的。可以看出这里的后处理和 RPN 的后处理相比，只有 

而如果是训练过程，就不需要后处理，但是需要另外两个步骤。

1. 是在 RoI align 之前，先对 proposal 进行采样。这里的采样过程和 RPN 求 loss 时的采样过程有些不同。这里同样用到了```Matcher```和```BalancedPositiveNegativeSampler```，但是参数上有很大不同。

     | RPN  | RCNN
------------------ | ---- | ----
FG\_IOU\_THRESHOLD | 0.7  | 0.5
BG\_IOU\_THRESHOLD | 0.3  | 0.5
allow\_low\_quality\_matches | T | F
BATCH\_SIZE\_PER\_IMAGE | 256 | 512
POSITIVE\_FRACTION | 0.5 | 0.25

	proposals 经过```Matcher```匹配和```Sampler```采样，输出新的 proposals 进行 RoI align。
	
2. 求 loss。和 RPN 一样仍然是交叉熵 loss 和 target 类的 smoothL1 loss。这里的 ```beta=1```，与 RPN 中不同。