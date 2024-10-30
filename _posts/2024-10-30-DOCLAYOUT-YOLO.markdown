---
layout:     post
title:      "DOCLAYOUT-YOLO"
subtitle:   "文档布局分析"
date:       2024-10-30 12:35:00
author:     "JIzp"
header-img: "img/bg-little-universe.jpg"
tags:
    - paper
---

# DOCLAYOUT-YOLO: ENHANCING DOCUMENT LAY-OUT ANALYSIS THROUGH DIVERSE SYNTHETIC DATA AND GLOBAL-TO-LOCAL ADAPTIVE PERCEPTION（DOCLAYOUT-YOLO：通过多样化的数据和全局到局部的自适应感知增强文档布局分析）

------

## DIVERSE DOCSYNTH-300K DATASET CONSTRUCTION

### A部分

A1代表一个小的文本数据集，从这里面去分割文本，获取组件。

实际上作者是从图中的A2开始的，需要一个不同分类的文本组件数据集，比如说广告、二维码、文本、表格，所谓的组件实际上就是图片。

A3：去便利所有组件的文档，数据小于阈值的进行数据增加，从原有的组件取出图片进行随机加噪，随机对比度等等操作，再放回去充当新数据。

### B部分

算法描述：

1. 取99%的大组件和1%的小组件作为候选集。
2. 从候选组件中随机选取，生成组件在空白页中中心点的随机坐标（左不小于w，右不大于1-w）。
3. 根据中心坐标计算组件的左上、右下坐标[x1,x2],[y1,y2]，做两次笛卡尔积，随意组合准备添加的网格坐标[(x1,y1),(x2,y2)]。
4. 过滤掉无效的网格坐标(x1>x2,y1>y2)(原有的网格，准备添加的网格计算iou)。
5. 遍历候选组件，遍历网格，选取填充率最高的填充。

![image-20241029105341650](C:\Users\24531\AppData\Roaming\Typora\typora-user-images\image-20241029105341650.png)

## GLOBAL-TO-LOCAL MODEL ARCHITECTURE

### Baseline

YOLOV10

### 修改了Baseline哪些部分

<img src="D:\QQ聊天记录\Tencent Files\2453194789\nt_qq\nt_data\Pic\2024-10\Ori\ebeb74fb4a735875be57f1e40b864a0f.png" alt="ebeb74fb4a735875be57f1e40b864a0f" style="zoom: 50%;" /><img src="D:\QQ聊天记录\Tencent Files\2453194789\nt_qq\nt_data\Pic\2024-10\Ori\76eb7fd683779d4696302abfa91ef8a5.png" alt="76eb7fd683779d4696302abfa91ef8a5" style="zoom:50%;" />

### CONTROLLABLE RECEPTIVE MODULE

可控是指有一个超参数：use_dilated TRUE OR FALSE

流程：

3 x 3 卷积

会有一个d列表，代表膨胀率（不同膨胀率会对卷积核进行不同程度的插值）。会对传进来的特征进行dn次卷积。得到的F1,F2......Fn，存储到列表中，然后concat得到F^，F^经过1 x 1卷积和sigmoid得到不同特征的权重M，M * F^。

3 x 3 卷积

![image-20241030100545983](C:\Users\24531\AppData\Roaming\Typora\typora-user-images\image-20241030100545983.png)

### GLOBAL-TO-LOCAL DESIGN

- Global-level（第四层，参数：[256, True, True, [1,2,3], 5, "glu"]）

  浅层使用大核

- Block-level（第六层，参数：[512, True, True, [1,3,5], 3, "glu"]）

  中间阶段使用较小核

- Local-level（第八层，参数：[1024, True, False]）

  对于深层直接CSP

![image-20241030123940135](C:\Users\24531\AppData\Roaming\Typora\typora-user-images\image-20241030123940135.png)
