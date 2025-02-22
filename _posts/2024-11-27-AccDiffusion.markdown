---
layout:     post
title:      "AccDiffusion"
subtitle:   "An Accurate Method for Higher-Resolution Image Generation"
date:       2024-11-27 13:18:00
author:     "JIzp"
header-img: "img/bg-little-universe.jpg"
tags:
    - paper
---

# AccDiffusion: An Accurate Method for Higher-Resolution Image Generation

------

这篇文章由厦门大学多媒体可信感知与高效计算教育部重点实验室撰写，目前还在arxiv上。

标题：AccDiffusion：一种高分辨率图像生成的精确方法

链接：**https://arxiv.org/html/2407.10738v1**

代码链接：**https://github.com/lzhxmu/AccDiffusion**

本文希望解决补丁式高分辨率图像生成过程中出现的对象重复问题。作者指出由于所有的补丁都使用了相同的文本提示，这导致了生成重复对象问题的产生。作者希望通过解耦文本提示的方法来解决上述问题。作者还提出了基于位置的双射函数，通过该方法使不同扩展样本的噪声之间进行交互来解决全局不一致的问题。

## 小对象重复的深入分析

对DemoFusion进行如下操作：

1. 生成过程中移除文本提示词，重复物体消失了，但细节出现了更多的退化。
2. 移除DemoFusion中的残差连接和膨胀采样操作，出现了严重的大物体重复现象。

## 基于内容感知的局部提示

使用低分辨率图像生成过程中的交叉注意力图
$$
M \in R^{N \times M}
$$
其中N是像素数量，M是提示词数量。因此，矩阵的列表示潜在噪声对第j个单词标记的注意程度。如果某个图像区域被第j个单词标记关注，其注意值通常会高于其他区域的值。

![image-20241126102203049](/img/AccDiffusion01.png)

将M转换成二值掩码，遍历每一个值，如果当前的值高于第j列的平均注意力，那么在矩阵B中当前值为1，表示该像素区域显著地被第j个单词标记所关注。这个时候的B是扁平情况下的，我们需要对B进行Reshape（把第j列的全部值输入，转成h * w），这样我们就得到了第 j个单词标记在空间上的显著区域。

![image-20241126102240266](/img/AccDiffusion02.png)

对于小的不连通的区域需要进行开运算（先腐蚀后膨胀）完成后掩码图进行插值，使掩码图的尺寸和高分辨率图像的尺寸一致。之后作者使用移位窗口采样，把掩码图分开。

**Tips：**腐蚀和膨胀

- 腐蚀：以结构元素为模板（有一个结构元），将结构元移动到图像的每个像素点的位置，如果模板内全是1结构元位置就是1，否则结构元位置置为0。

![这里写图片描述](/img/AccDiffusion04.png)

- 膨胀（这个操作貌似不止一个方法）：以结构元素为模板（有一个结构元），将结构元移动到图像的每个像素点的位置，将结构元素位置全部置为1。

![img](/img/AccDiffusion05.png)

现在每一个补丁都对应了一套掩码图（通道维度上的每一张图片对应一个词），现在数每张掩码图中的1，如果大于阈值，就保留该词，否则不保留。至此，实现了提示词解耦。

## 具有窗口交互作用的扩张采样

**扩张采样**：对Zt先进行膨胀采样（跳步的切片）,挨个对采样出的潜在表示去噪，然后再拼回去得到一个完整的潜在表示，和分块去噪得到的Zt-1求加权和得到处理后的去噪（加权值是从1渐变到0，刚开始通过膨胀采样的）。为什么会进行这样一个操作，因为我们的Zt-1是分块去噪得到的，我们需要把全局信息融合到Zt-1里面。

**双射函数**：双射函数（每个位置时间步长不一样，函数不一样）就是每个补丁里面的元素换位置，去噪，再换回来。现在变成了这样的步骤：先进行膨胀采样，然后经过双射函数，去噪，双射函数的逆变换，再拼回去，再和Zt-1加权和。

![image-20241127125243265](/img/AccDiffusion03.png)
