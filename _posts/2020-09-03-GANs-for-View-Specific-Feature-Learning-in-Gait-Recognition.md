---
title: "论文阅读笔记：GANs for View-Specific Feature Learning in Gait Recognition"
layout: post
date: 2020-09-03
tag: 
- Paper
Category: blog
Author: ingerchao
Description: 利用 GAN 完成特定角度下的步态识别任务
---

### Abstract

步态识别中现存问题

- 现有的交叉视角步态识别方法针对**从不同视角转换步态模版**再进行识别，因此会随着视角差异的变大而积累误差；
- 常用的 GEI 模版会丢失步态序列的时间信息。

本文提出了一种 **多任务生成对抗网络**，用于学习特定试图下的特征表示，利用对抗训练从步态序列中提取更多判别特征。同时提出了周期能量图像（Period Energy Image）。

实验数据集：OU-ISIR, CASIA-B, USF

### Introduction

<img src="./../assets/images/paper/903-gan-gait-recognition.png" alt="gan-gait-recognition" style="zoom:40%;" />

1. PEI：经过卷积神经网络提取特征；
2. 视角分类器：交叉熵损失训练视角分类器，判断输入序列属于哪个视角；
3. View-Transform Layer: 从一个视图转换到另一个视图；
4. 对改进的GANs结构进行像素级损失和多任务对抗性损失训练。

