---
title: "训练一个管道识别模型"
date: 2020-11-16
category: blog
tag: 
- OpenCV
author: ingerchao
---



### 一. 准备数据集

- 使用 iMovie 将真实场景下CCTV录制的地下水管道视频剪辑，使用 `videos_to_image` 脚本将视频批量转化为图像；
- 准备正副样本集；
  - Positive Sets：想让分类器检测出来的管道特征；
  - Negative Sets：与任务相关但不想被检测出来的东西，可以将图片中的其他信息裁剪为 64*64 的多个小图片。[参考博客](https://blog.csdn.net/u010429424/article/details/74377617)

