---
title: "相机标定与测距原理及 OpenCV 实现"
layout: post
date: 2020-9-14 10:06
category: blog
tag: 
- opencv
author: ingerchao
---



### 相机标定

<img src="https://picb.zhimg.com/80/v2-afff3b4901966569a5203751afb5e50f_1440w.jpg" style="zoom:33%;" />

三维真实世界到二维可以理解为通过一定的函数变换，那么二维到三维通过一个反函数可以实现；相机标定的目的是通过找到一个数学模型，求出这个模型的参数。「相机标定」就是**通过数学模型表达复杂的成像过程，并且可以用于表示成像的反过程**。

标定之后的相机可以进行三维场景的重建，通过感知到的深度构建。

将 ObjectPoint 称为物点（三维点），ImagePoint 称为像点（二维点）。

#### 针孔相机模型的描述

对相机成像过程进行简化和建模，得到如下的针孔相机模型，本质上就是把相机简化为小孔成像。

<img src="https://picb.zhimg.com/80/v2-0dc5b5ea8f626e1ac827d7aabcea8efc_1440w.jpg" alt="img" style="zoom:48%;" />

首先建立**相机坐标系**，以光心 O 为 坐标系原点，X 与 Y 方向是 CCD 像素排列的水平和竖直两个方向，Z 方向垂直于 [CCD 面](https://baike.baidu.com/item/CCD%E7%9B%B8%E6%9C%BA)，建立右手坐标系。其次，建立 CCD 标号坐标系，以 CCD 左上角像素标号为原点，CCD 像素排列的水平和竖直两个方向为 U 与 V 方向。

<img src="https://pic2.zhimg.com/80/v2-e4fdd182aa1bb94cd626fe71d4993a40_1440w.jpg" alt="img" style="zoom:48%;" />

- 由光心 O 沿着光轴出发，像平面在 $Z = f$ 上，$f$ 为相机的物理焦距（单位：$mm$）；
- 点 Q 在物理空间中，在相机坐标系下的位置为 $Q(X, Y, Z)$；
- 点 P 在像平面上
  - 在相机坐标系下的位置是 $P(x,y,z)$；
  - 在 CCD 标号坐标系下的位置是 $P(u_{ccd}, v_{ccd})$；

在无镜头畸变的条件下，光心 O、点 P 和点 Q 在一条直线上。

- $k, l$ 是 CCD 单个像素在水平和竖直两个方向上的尺寸（单位：$mm / pixel$ ），因此定义焦距为 $f_x = \frac{f}{k}, f_y=\frac{f}{l}$  （单位：像素）;
- CCD 标号坐标系原点到光轴的偏移量为 $c_x,\ c_y$ （单位：像素）;
- 根据相似三角形关系原理可以得出：

$$
\begin{cases}
u_{ccd} = f_x(\frac{X}{Z} + c_x) \\
v_{ccd} = f_y(\frac{Y}{Z} + c_y)
\end{cases}
$$

> 成像原理理论知识先不看了，看看 OpenCV 相机标定应该怎么用。

#### OpenCV 官方文档

> Finds the camera intrinsic and extrinsic parameters from several views of a calibration pattern.

从校准图案的多个视图中查找相机的固有参数和外部参数。

```c++
double calibrateCamera(InputArrayOfArrays objectPoints, InputArrayOfArrays imagePoints, Size imageSize, InputOutputArray cameraMatrix, InputOutputArray distCoeffs, OutputArrayOfArrays rvecs, OutputArrayOfArrays tvecs, int flags=0, TermCriteria criteria=TermCriteria( TermCriteria::COUNT+TermCriteria::EPS, 30, DBL_EPSILON) )¶
```

方法作用：估算每个视图下相机的固有参数和外部参数。在每个视图下，3D object points 和他们相关的 2D image Points 必须明确。

参数列表：

- objectPoints（3D 点）：
- imagePoints（2D 点）：

> 暂时没看懂，先放一放



### 单目相机测距

#### 原理

<img src="https://img2020.cnblogs.com/blog/1251718/202005/1251718-20200503161951749-1222809582.png" alt="img" style="zoom:75%;" />

根据相似三角形计算单目相机到物体的距离，必须已知一个确定的长度。

假设我们有一个宽度为 W 的目标或者物体。然后我们将这个目标放在距离我们的相机为 D 的位置。我们用相机对物体进行拍照并且测量物体的像素宽度 P 。这样我们就得出了相机焦距的公式：$F= \frac{P\times{D}}{W}$

当我们将摄像头远离或者靠近A4纸时，就可以用相似三角形得到相机距离物体的距离。
此时的距离：$D' = \frac{W'\times{F}}{P'}$。

#### 实现Demo



---

参考资料：

- [相机标定究竟在标定什么？](https://zhuanlan.zhihu.com/p/30813733)
- [OpenCV 2.4.6 官方文档](https://docs.opencv.org/2.4.6/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html#calibratecamera)
- [【立体视觉】世界坐标系、相机坐标系、图像坐标系、像素坐标系之间的关系](https://blog.csdn.net/u011574296/article/details/73658560)
- [摄像头单目测距原理及实现](https://www.cnblogs.com/wujianming-110117/p/12822331.html)
- [单目摄像机测距（python+opencv）](https://blog.csdn.net/m0_37811342/article/details/80394935)

