---
title: "OpenCV: 测量物体大小"
date: 2020-10-27 
category: blog
tag: 
- opencv
author: ingerchao
---



要测量目标图片中的物体大小，必须要使用一个参考物体的尺寸，通过计算 **参考物体/参考物体的像素数** 得到一个比值。其他的物体只需要通过 **物体像素数 $\times$ 这个比值 **就可以得到物体的尺寸。将这个比值可以定义为 $R_{PM}$.

### The Pixels per Metric Ratio

为了测量图片中的物体尺寸，我们首先需要标定一个参考物体，定义参考物体中的以下两个关键属性：

- 一个可测量物体及其尺寸
- 根据对象的位置（例如始终将参考对象放置在图像的左上角）或者通过外观（例如一种独特的颜色或形状，与图像中的所有其他对象都不同），无论哪种方式，参考对象必须以某种方式唯一地进行标识。

在本次实验的参考图像中是一枚美分硬币。确定了参考物体是最左侧的物体，就可以从左到右遍历每一个目标并计算每个物体的尺寸。通过参考物体计算 $R_{PM} = D_{img} / D_{obj}$ ，其中，分母为物体在图片中的长度，分子为物体的实际长度。

则物体的实际长度 $D_{obj} = D_{img} / R_{PM}$ 。

### Code

对图像进行预处理，灰度化、边缘提取等操作。

```python
# 读取输入图片
image = cv2.imread(args["image"])
# 输入图片灰度化
gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
# 对灰度图片执行高斯滤波
gray = cv2.GaussianBlur(gray, (7, 7), 0)
showImage("gaussian blur", gray)

# 对滤波结果做边缘检测获取目标
edged = cv2.Canny(gray, 50, 100)
# 使用膨胀和腐蚀操作进行闭合对象边缘之间的间隙
edged = cv2.dilate(edged, None, iterations=1)
edged = cv2.erode(edged, None, iterations=1)
showImage("edged", edged)
```

在边缘计算过后的图像中寻找物体轮廓：

```python
# 在边缘图像中寻找物体轮廓（即物体）
cnts = cv2.findContours(edged.copy(), cv2.RETR_EXTERNAL,
                        cv2.CHAIN_APPROX_SIMPLE)
cnts = imutils.grab_contours(cnts)

# 对轮廓按照从左到右进行排序处理
(cnts, _) = contours.sort_contours(cnts)
```

循环遍历每一个轮廓，并根据物体的轮廓计算出外切矩形框并在图形中进行绘制。通过中点函数计算出各个中点的坐标，再通过计算中心店的欧式距离得到物体在图片中的距离$D_{img}$, 包括长度和宽度，由于我们的参考物体为正方形，所以根据长度或宽度计算均可。

```python
    # 根据物体轮廓计算出外切矩形框
    orig = image.copy()
    box = cv2.minAreaRect(c)
    box = cv2.cv.BoxPoints(box) if imutils.is_cv2() else cv2.boxPoints(box)
    box = np.array(box, dtype="int")

    # 按照top-left, top-right, bottom-right, bottom-left的顺序对轮廓点进行排序，并绘制外切的BB，用绿色的线来表示
    box = perspective.order_points(box)
    cv2.drawContours(orig, [box.astype("int")], -1, (0, 255, 0), 2)
```

若当前物体为第一个物体，即参考物体，计算 $R_{PM}$ 的值：

```python
    if pixelsPerMetric is None:
        pixelsPerMetric = dB / args["width"]
```

通过 $R_{PM}$ 计算每个物体的实际长度：

```python
    # 计算目标的实际大小（宽和高），用英尺来表示
    dimA = dA / pixelsPerMetric
    dimB = dB / pixelsPerMetric
```

### Results

```bash
python3 measure_oj_size.py --image ./pic/size_of_objects_reference.jpg --width 0.955
```

<img src="./../assets/images/opencv/LQyNul.jpg" alt="results" style="zoom:50%;" />

<img src="./../assets/images/opencv/ojU5P1.jpg" alt="results" style="zoom:50%;" />

----

[source code](https://github.com/Inger-Chao/opencv_demo/blob/main/measure_oj_size.py)

[Reference post](https://www.pyimagesearch.com/2016/03/28/measuring-size-of-objects-in-an-image-with-opencv/)

