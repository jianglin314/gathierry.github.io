---
layout: post
title: 拆解 MaskRCNN-Benchmark 之三 —— RPN
categories: [data science]
tags: [object detection, instance segmentation, deep learning, computer vision]
description: 这篇文章介绍 FAIR 项目 maskrcnn-benchmark 模型中的 RPN 部分。
---
在 Mask RCNN 中，预设 anchor 数量从 Faster RCNN 的 9 个， 提升到了 15 个。这里```ratio=[0.5,1,2]```，而```scale```在每一层 feature map 上（```stride=4,8,16,32,64```）的尺度都是 8，映射到原图尺度上就是```scale=[32,64,128,256,512]```。

假设图像尺寸为 $$(H, W)$$，特征图 $$stride=\{S1, S2, S3, S4, S5\}$$，则特征图尺寸为 $$(H/S_i, W/S_i)$$。在每层特征上，$$scale=s$$, $$ratio=\{r1, r2, r3\}$$。Batch size 记为 $$B$$，anchor 数量记为 $$A$$。

## RPN Head
RPN 由一个简单的 head 紧接 FPN 每一个 level 的特征之后，它们**共享** head 的权重。设 FPN 输出的特征图为 P2, P3, P4, P5, P6, 每个特征图都经过同一个 3x3 卷积，然后两个 1x1 卷积分支得到两个输出，$$(B, A, H/S_i, W/S_i)$$ 和 $$(B, 4\times A, H/S_i, W/S_i)$$，表示前景的 score 和四个坐标的 regression 结果。

这里的卷积层初始化使用的并不是 xavier 或者 kaiming，而是
	
	torch.nn.init.normal_(l.weight, std=0.01)
	torch.nn.init.constant_(l.bias, 0)
	
详细代码在```maskrcnn_benchmark/modeling/rpn/rpn.py```

## Anchor Generator
RPN head 可以输出 box score 和 box regression，然而如果要想转化成原图尺度下的 bounding box，还需要根据相应的 anchor 进行转化。即

$$BoundingBoxes = BoxCoder(regression, anchor)$$

对于一个 $$(H/S_i, W/S_i)$$ 的特征图，需要生成 $$A \times H/S_i \times W/S_i$$ 个 anchor，这些 anchor 可以储存成```BoxList```对象。

### 1. 生成 anchor cells
每个 anchor cell 都是一个组合 $$(\Delta x_1, \Delta y_1, \Delta x_2, \Delta y_2)$$。feature map 上的一个像素，对应原图上 $$S_i \times S_i$$ 个像素。以 $$(0, 0, S_i-1, S_i-1)$$ 为基础进行放缩，可以生成 $$A$$ 个 anchor cells。每个特征图对应各自的 anchor cells。

### 2. 原图上取网格点
对每一个特征图，在原图尺度上以 $$S_i$$ 等间隔的取点 $$(x_a^i, y_a^i)$$ 经过变换可以得到一系列 anchors $$(x_a^i + \Delta x_1, y_a^i + \Delta y_1, x_a^i + \Delta x_2, y_a^i + \Delta y_2)$$。因为 anchor cell 假设以 $$(0, 0, S_i-1, S_i-1)$$ 为基础，所以在特征图上取点可以从 0 开始

	torch.arange(0, grid_width * stride, step=stride, dtype=torch.float32, device=device)
	
最终每个特征图都在原图上生成了 $$H/S_i \times W/S_i$$ 个等距且距离为 $$S_i$$ 的格点。

### 3. 生成 anchors
每一个格点与相应特征图上所有的 anchor cell 组合，生成一个 anchor，最终生成形状为 $$(A \sum_i (H/S_i \times W/S_i), 4)$$ 的 anchor boxes。

### 4. 包装进 BoxList
这些 anchor boxes 被包装成```BoxList```的对象。这里对每一个 anchor 进行了检查，如果存在边界超出图片尺寸的 anchor，会通过```add_field("visibility", 0)```来标记，在后续操作中进行忽略。

最终输出的 anchor 形式为 $$(B, 5, A \times H/S_i \times W/S_i, 4)$$。前两个维度以嵌套```list```形式储存，后两个维度以```BoxList```形式储存。

## RPN 后处理
RPN 生成了大量的 box，每个 anchor 都对应一个 box，显然不能全部输出到后面的网络中去，这就需要一些的后处理步骤选取最有意义的 proposals。这一步涉及到三个常数，```PRE_NMS_TOP_N```，```POST_NMS_TOP_N```，```FPN_POST_NMS_TOP_N```。后面会挨个涉及到。

### Box Coder
有了 anchor，我们已经可以复原出 RPN head 的输出所对应的 box 坐标了。这个转换的公式通过```maskrcnn_benchmark/modeling/box_coder.py```的```BoxCoder```实现。

Encode 过程是从真实 box 转化成输出格式。

- $$t_x = \frac{x-x_a}{w_a} \times W_x$$
- $$t_y = \frac{y-y_a}{h_a} \times W_y$$
- $$t_w = \log{\frac{w}{w_a}} \times W_w$$
- $$t_h = \log{\frac{h}{h_a}} \times W_h$$

Decode 则是相反的过程。

- $$x = \frac{t_x w_a}{W_x} + x_a$$
- $$y = \frac{t_y h_a}{W_y} + y_a$$
- $$w = w_a \exp{\frac{t_w}{W_w}}$$
- $$h = h_a \exp{\frac{t_h}{W_h}}$$

与论文中稍有不同的是加入了一项权重项。代码中 $$W=(1,1,1,1)$$，另外通过 ```bbox_xform_clip=math.log(1000. / 16)``` 限制了 decode 时 ```exp``` 指数的上限，避免出现过大的值。

**对回归值的编码，是为了计算 loss 能有效收敛。而其他对框的操作，如计算 IoU 则要解码后进行。**

### 过滤冗余的输出

<img src="/images/2019-02-17-maskrcnn-benchmark-3/proposal.png" width="1000px"/>

对于每一个 feature map 的输出，根据 score 排序，保留至多前```PRE_NMS_TOP_N```个 box。需要注意的是这里是对整个 batch 内部进行排序，而不是单张图片内部进行排序。之后通过```clip_to_image```去除超出图片边界的框，通过```remove_small_boxes```去掉过小的框（这里阈值为 0，即不会删框）。然后进行 NMS，保留至多```POST_NMS_TOP_N```个框。

因为有五个特征输出，所以经过上一步会得到至多```5*POST_NMS_TOP_N```个 proposals。通过 score 统一排序，保留前```FPN_POST_NMS_TOP_N```个。这一步训练和测试稍有区别，训练时是整个 batch 排序，而测试时是每张图片分别排序。不过从源代码的注释上来看，这两个阶段应该会统一成后一种。

最后，如果是训练，则把 ground truth 也加入到 proposal 中，丰富正样本。

## RPN Loss
RPN 的 loss 分为两部分，分类的交叉熵 loss 和回归的 smoothL1 loss。然而并不是所有的预测结果都被计入 loss 当中。首先要区分出预测结果中哪些是正确的，哪些是错误的，哪些是忽略不计的。之后在正确分类和误分类中按固定比例采样，计算 loss，而不是一股脑全部算进去。计算 loss 除了需要 RPN 的两个输出，还需要 anchors 和 targets。

### Matcher
在```maskrcnn_benchmark/modeling/matcher.py```里实现的```Matcher```，就是通过对预测结果和真实结果进行两两对比，划分正确样本、错误样本和忽略的样本。```Matcher```在初始化时需要定两个阈值，```RPN.FG_IOU_THRESHOLD```和```RPN.BG_IOU_THRESHOLD```。

使用时，```Matcher```的输入是一个尺寸为```(target#, prediction#)```的矩阵。矩阵里的值是两两配对的IoU。通过求最大值找到对于每一个 prediction 最靠谱的 target，得到两个```(prediction#, )```的向量——最大值和相应的 target index。如果最大值小于```RPN.BG_IOU_THRESHOLD```，index 设成 -1，如果最大值在```RPN.BG_IOU_THRESHOLD```和```RPN.FG_IOU_THRESHOLD```之间，index 设成 -2，其余保持不变，仍然是原来的 target index，将这个结果记为```matches```。

代码中还设定了```allow_low_quality_matches=True```。就是将与每个 target IoU 最大（一个或多个，可能并列）的 prediction 也设为正样本。先求出每个 target 对应的最大的 overlap，一个长度为```(target#, )```的向量，之后在原始的输入的```(target#, prediction#)```矩阵中在每一行找到等于最大 overlap 的 prediction 的 index，在```matches```中将响应的 prediction 对应的 target index 恢复成 -1，-2 以外有效的 target。

### FG/BG Sampler
通过```Matcher```，每一个 prediction 都可以被归入负样本（0），正样本（1），忽略样本（-1）中的一个。为了均衡样本，可以通过预设参数```BATCH_SIZE_PER_IMAGE```，```POSITIVE_FRACTION```强制调整正负样本的比例。当正样本小于```BATCH_SIZE_PER_IMAGE * POSITIVE_FRACTION```时，负样本数量就是```BATCH_SIZE_PER_IMAGE - num_pos```。当正样本数量大于这个限制时，随机选取，剩下的则在负样本中随机选取。

### 求 loss
最后就可以整理采样的样本计算 loss 了。这里 target 需要先通过```BoxCoder``` encode 转化成网络输出的形式，再计算 loss。

代码在```maskrcnn_benchmark/layers/smooth_l1_loss.py```重新实现并扩展了 smoothL1 loss，加入了参数```beta```来控制 smoothL1 中二阶曲线的范围。

## Enfin...
至此，RPN 就完成了。RPN 的最终输出是一个 BoxList 类型的 proposals，以及两个 loss。