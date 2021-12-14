## HoughCircles in OpenCV

关于 OpenCV 中的 HoughCircles 函数，简单来说是基于改进的 Hough 变换在灰度图中寻找圆。

官方文档中关于 hough circles 的备注：通常 HoughCircles 能基于图像中的特征点拟合出圆形方程，然而它无法为每个候选圆心确定一个正确的半径。

>通常该函数可以很好地检测圆心。但是，它可能无法找到正确的半径。如果您知道，您可以通过指定半径范围（ minRadius 和 maxRadius ）来协助该功能。或者，在 #HOUGH_GRADIENT 方法的情况下，您可以将 maxRadius 设置为负数以仅返回中心而不进行半径搜索，并使用附加程序找到正确的半径。
>
>它也有助于平滑图像。例如，具有 7x7 内核和 1.5x1.5 sigma 或类似模糊的 GaussianBlur() 可能会有所帮助。
>
>@note Usually the function detects the centers of circles well. However, it may fail to find correct radii. You can assist to the function by specifying the radius range ( minRadius and maxRadius ) if you know it. Or, in the case of #HOUGH_GRADIENT method you may set maxRadius to a negative number to return centers only without radius search, and find the correct radius using an additional procedure.
>
>It also helps to smooth image a bit unless it's already soft. For example, GaussianBlur() with 7x7 kernel and 1.5x1.5 sigma or similar blurring may help.



