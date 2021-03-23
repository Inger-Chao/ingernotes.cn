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
- 使用 OpenCV 提供的工具生成正服数据的 xml 文件。[参考博客](https://suimingyang.github.io/2019/07/03/opencv%E7%BA%A7%E8%81%94%E8%AE%AD%E7%BB%83%E5%9B%BE%E5%83%8F%E5%88%86%E7%B1%BB%E5%99%A8/)

**⚠️注意：** OpenCV 需要切换到 3.4 版本，opencv4 编译 opencv_traincascade 会出现 Error。OpenCV4 更推荐用 RCNN 或其他神经网络来进行目标识别任务。

```bash
➜  bin git:(3.4) pwd
/Users/inger/opencv/opencv/build/bin

➜  bin git:(3.4)./opencv_createsamples -vec positive.vec -info ~/underground/datasets/positive.txt 18 -w 64 -h 64

➜  bin git:(3.4) cd ~/underground/datasets
➜  datasets ls
cascades            deform-collapse     negative.txt        obstacle            pipe                positive.vec
corrosion-damage    labelme             non-pipe            opencv_traincascade positive.txt
➜  datasets ./opencv_traincascade -data cascades -vec positive.vec -bg ~/underground/datasets/negative.txt -numPos 18 -numNeg 89 -numStages 5 -w 64 -h 64 -minHitRate 0.9999 -maxFalseAlarmRate 0.5 -mem 2048 -mode ALL
PARAMETERS:
cascadeDirName: cascades
vecFileName: positive.vec
bgFileName: /Users/inger/underground/datasets/negative.txt
numPos: 18
numNeg: 89
numStages: 5
precalcValBufSize[Mb] : 1024
precalcIdxBufSize[Mb] : 1024
acceptanceRatioBreakValue : -1
stageType: BOOST
featureType: HAAR
sampleWidth: 64
sampleHeight: 64
boostType: GAB
minHitRate: 0.9999
maxFalseAlarmRate: 0.5
weightTrimRate: 0.95
maxDepth: 1
maxWeakCount: 100
mode: ALL
Number of unique features given windowSize [64,64] : 13481422
```

| 参数               | 数值         | 说明                           |
| ------------------ | ------------ | ------------------------------ |
| -data              | cascades     | 训练后生成xml的文件夹          |
| -vec               | positive.vec | 正向数据                       |
| -bg                | negative.txt | 负向数据                       |
| -numPos            | 18           | 正向样本数                     |
| -numNeg            | 89           | 负向样本数                     |
| -numStages         | 5            | 训练迭代数                     |
| -w                 | 64           | 图像宽                         |
| -h                 | 64           | 图像高                         |
| -minHitRate        | 0.9999       | 期望最小检测率                 |
| -maxFalseAlarmRate | 0.5          | 期望最大误检率                 |
| -mem               | 2048         | 使用的计算内存数（M）          |
| -mode              | ALL          | 选择用来训练的haar特征集的种类 |

