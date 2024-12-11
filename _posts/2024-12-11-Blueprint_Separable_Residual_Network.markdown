---
layout:     post
title:      "Blueprint Separable Residual Network"
subtitle:   "Blueprint Separable Residual Network for Efficient Image Super-Resolution"
date:       2024-12-11 15:03:00
author:     "JIzp"
header-img: "img/bg-material.jpg"
tags:
    - paper
---

# Blueprint Separable Residual Network for Efficient Image Super-Resolution

------

这篇文章由深圳市计算机视觉与模式识别重点实验室撰写，发表在2022CVPRW。

标题：蓝图可分离残差网络的高效图像超分辨率

链接：[CVPR 2022 Open Access Repository (thecvf.com)](https://openaccess.thecvf.com/content/CVPR2022W/NTIRE/html/Li_Blueprint_Separable_Residual_Network_for_Efficient_Image_Super-Resolution_CVPRW_2022_paper.html)

代码链接：https://github.com/xiaom233/BSRN

这篇文章介绍了一种名为Blueprint Separable Residual Network (BSRN)的轻量级单图像超分辨率模型，旨在实现高效的超分辨率重建，特别适用于计算资源有限的边缘设备。

## Network Architecture

### 前言：

### 深度可分离卷积

![image-20241209183541164](/img/Blueprint01.png)

正常卷积如果输入是三通道输出需要三通道，那么我们需要三个3 * 3 * 3 * 3的卷积核。

深度可分离卷积是一种逐通道的卷积，每次不进行相加操作（不同通道之间相同位置的值不进行相加），这种操作通道之间的信息没有进行交互。如果输入是三通道输出需要三通道，那么我们需要三个3 * 3 * 3 * 1的卷积核。然后再进行逐点卷积，使用1 * 1 * 3 * 3的卷积核利用这个卷积核相加。

### 正文：

![image-20241209182043483](/img/Blueprint02.png)

在预处理阶段，输入图像首先被复制n次，然后将这些图像沿通道维度拼接在一起。接下来是浅层特征提取部分将输入图像映射到一个高维特征空间。浅层特征随后通过一组堆叠的ESDB进行深层特征提取。

## Efficient Separable Distillation Block

![image-20241209190854640](/img/Blueprint03.png)![image-20241209191014066](/img/Blueprint04.png)![image-20241209191046127](/img/Blueprint05.png)

这个步骤是这样一个过程，左边是1 * 1卷积降维（精细特征），右边是深度可分离卷积的反向版本（正常深度可分离卷积是1 * 1卷积在后的，这里在前）加激活函数（得到的是粗略特征），将得到的四个精细特征concat，再用1 * 1卷积降维，然后依次通过ESA、CCA。

## Attention modules of ESA and CCA

![image-20241211145000381](/img/Blueprint06.png)![image-20241211145018921](/img/Blueprint07.png)

### ESA

空间维度的注意力：通过1 * 1的卷积降维，通过步幅卷积和步幅最大池化减小空间尺寸，然后通过卷积提取特征，使用插值上采样恢复空间，进行残差连接，使用1 * 1卷积恢复通道，通过sigmod函数生成注意力矩阵。

### CCA

通道维度的注意力：计算对比信息（均值和标准差总和），经过两个1 * 1卷积，经过sigmod函数生成注意力矩阵。
