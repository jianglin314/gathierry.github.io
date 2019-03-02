---
layout: post
title: 拆解 MaskRCNN-Benchmark 之一 —— 数据的整理与加载
categories: [data science]
tags: [object detection, instance segmentation, deep learning, computer vision]
description: 这篇文章介绍 FAIR 项目 maskrcnn-benchmark 模型的输入，也就是数据准备与预处理。
---
既然是监督学习，网络模型首先就要定义输入。对于目标检测和实例分割任务而言，输入的除了图像，还需要每幅图中检测框的坐标，以及物体分割的 mask。这样的输入数据格式显然比单纯的分类问题要复杂很多，同时还需要考虑到输入图片的分辨率、长宽比不一致。在实现时，因为深度学习框架的输入必须是张量的形式，这就要求开发者对数据进行足够的预处理，才能放入网络。

## 检测、分割标签数据的整理
一张图中可能会存在多个检测框以及分割实例，而且这个数量是随不同图像变化的。

maskrcnn-benchmark 中，在```maskrcnn_benchmark/structures/bounding_box.py```中定义了一个```BoxList```类。每一个```BoxList```对象即是一幅图像的标签。这个类内部包含了```Tensor```类型的成员变量```bbox```用来存储一张图中检测框的信息，同时还有一个字典类型的成员变量```extra_fields```用来存储分割以及其他可能会用到的数据。

如果我们看一下整套代码，就会发现```extra_fields```所存过的变量，包括

- 分割标签```mask```
- 物体类别```labels```
- 匹配检测框与 ground truth 框的```matched_idx```
- RPN 预测的置信度```objectness```
- RCNN 预测的置信度```score```
- anchor 是否超出图片边界```visibility```

除了```mask```以外，其他变量都比较简单，对于每个框都都可以用一个标量表示。

分割标签的数据更加不规则，每个数据表示一个多边形，是由长度不等的 list 表示的。每张图片会有多个 mask，而每个 mask 也可能分成多个 polygons（比如物体被遮挡分成两部分）。代码中通过```maskrcnn_benchmark/structures/segmentation_mask.py```整理分割的标签数据，并且作为一个```extra_field```存在一个```BoxList```对象中。

这样一层包装，有效的将标签信息传入到网络模型中。

## DataLoader
PyTorch 的数据加载，是基于 [```torch.utils.data.DataLoader```](https://pytorch.org/docs/stable/data.html#torch.utils.data.DataLoader)。```DataLoder```的主要参数有

- ```dataset```
- ```batch_sampler```
- ```collate_fn```

```dataset```提供每次迭代产生的单个数据，```collate_fn```将产生的数据组合成 batch，而每个 batch 中数据的取样规则，通过```batch_sampler```来控制。

### Dataset

有了图片和标签，就可以构造[```torch.utils.data.Dataset```](https://pytorch.org/docs/stable/data.html#torch.utils.data.Dataset)。```Dataset```主要的输出是```(image, boxlist)```。一些预处理以及数据增强，都在这里实现。PyTorch 提供了图像预处理以及数据增强的接口[```torchvision.transforms```](https://pytorch.org/docs/stable/torchvision/transforms.html)。但是这里当我们对图片进行 resize，flip 之类的操作时，需要对检测框、分割框进行相应的调整，所以在项目目录```maskrcnn_benchmark\data\transforms\transforms.py```中重写了定制化的接口。

### Batch Collator
```collate_fn```函数在```dataset```取值之后调用。假设```batch_size=2```，那么```collate_fn```的输入就是长度为 2 的 list，每个元素是一组```dataset```的输出，比如```(image, boxlist, idx)```。

当 mini-batch > 1 时，同一个 batch 中的图片大小必须是一致的。然而输入图像有时无法满足这一要求。这时我们就需要在```collate_fn```中将 batch 中的所有图片 padding 到该 batch 最大图片的尺寸。代码中在```maskrcnn_benchmark\structrues\image_list.py```构造了```ImageList```类，将 padding 后的图像组成一个 tensor，同时储存了原本的图像尺寸信息。

### Batch Sampler
PyTorch 中提供了[```torch.utils.data.BatchSampler```](https://pytorch.org/docs/stable/data.html#torch.utils.data.BatchSampler)的接口，包含基本的实现。然而在```maskrcnn_benchmark\data\samplers\```进行了扩展，这里只说```grouped_batch_sampler.py```。

```GroupedBatchSampler```通过参数```group_id```的指导，将```group_id```相同的数据放入同一个 batch。maskrcnn 中这个```group_id```是由图片长宽比大于/小于 1 来区分的。这样做的结果就是避免横版图片和竖版图片放进同一个 batch，从而导致 ```collate_fn```时为了补齐尺寸差异而大量填零。

<img src="/images/2019-02-18-maskrcnn-benchmark-1/dataloader.png" width="600px"/>

**经过以上步骤，数据就以```ImageList```和```BoxList```的形式传入了模型。然而它们的本质仍然是```tensor```类型，这也是为什么经过包装的数据可以被运算框架接受并且进行高效运算。**