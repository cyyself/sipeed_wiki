---
title: YOLOv2 目标检测模型
---


YOLO(You Only Look Once) v2 成功让目标检测达到实时的同时，有着较高检测准确率，正如其名字一样，你只需要看一眼，就能知道结果。

YOLO v2 因其检测效率高，可以在性能不是很强的边缘设备运行，不过也有缺点，就是对于小物体的检测能力不够。

## 目标

输入一张图片，输出图片中的目标类别和位置，能同时检测多个物体。


## 原理

论文原文：[YOLO9000: Better, Faster, Stronger](https://arxiv.org/abs/1612.08242)

YOLOV2 主干网络也是卷积网络，输出层则使用了 S x S x (Bx5+C) 这样一个三维结构，其中 S 是网格的大小，B 是每个网格预测的边界框数量，C 是类别数量。
也就是说，在 S x S 个网格中，每个网格预测 B 个框，每个框有 5 个参数，分别是 x, y, w, h, p，其中 x, y 是框的中心点相对于网格左上角的偏移，w, h 是框的宽高，p 是框的置信度，最后 C 个参数分别是对应类别的概率。
对于这 B 个框，预先我们手动确定了 B 个 anchor（预选框），每个 anchor 有 w, h 两个参数，这里 B 个预测框的 w, h 是相对于这 B 个 anchor 的缩放系数，这样预测框的 w, h 就可以是任意值了。
> 这 B 个 Anchor 是使用 K-Means 算法对训练的数据所有标注框的宽高聚类得到的，聚类的目标是让每个 anchor 的宽高比接近真实框的宽高比，这样预测框的宽高比就会更接近真实框的宽高比，从而提高检测精度。所以代码里面会有一个 anchors 参数，这个参数就是我们训练的时候统计训练数据得到的 B 个 anchor，实际在跑模型时一定要保证这个参数和训练时的一致。

在获得所有预测框后，还需要对预测框进行过滤，去除一些置信度低的框，即上面说的 p，这也就是我们在代码里看到的阈值（threshold）; 以及需要进行 nms（non maximum suppression/非极大值抑制） 计算去除预测出相同分类并且有太多重叠部分的框，保留概率大的一个即可，所以代码里面我们看到有一个 nms_threshold 参数，这个参数就是用来过滤 IOU（两个框的相交区域（交集）面积/并集面积）大于这个阈值的框的。


看起来有点复杂的样子，要想详细学习可以参考更多三方教程和官方论文，实际我们训练的 YOLO v2 模型纯粹的模型到输出 S x S x (Bx5+C) 这样一个三维张量就行了，然后代码库提供了一个解码的函数，调用函数输入这个三维张量就可以得到合法的预测框输出了，就算你对原理不了解，也不妨碍你使用这个模型。

## YOLO v3

YOLO v3 相比于 YOLO v2， 做了多输出，更利于检测小物体，不过因为多输出，在计算量上也会大很多，帧率自然也比 YOLO v2 更低。

## 试试训练一个模型

在 [MaixHub](https://maixhub.com) 创建一个检测训练项目，然后采集数据并标注（可以在线标注），创建一个训练任务，参数选择你有的硬件平台，如果没有开发板可以使用手机，选择 yolov2 进行训练即可，训练完成后可以一键部署看看效果。


