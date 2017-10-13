**OpenCV3编程入门6-2非线性滤波**

[TOC]

# 中值滤波

中值滤波（Median filter）是一种典型的非线性滤波技术，基本思想是用像素点邻域灰度值的中值来代替该像素点的灰度值，该方法在去除脉冲噪声、椒盐噪声的同时又能保留图像的边缘细节。

中值滤波是基于排序统计理论的一种能有效抑制噪声的非线性信号处理技术，其基本原理是把数字图像或数字序列中一点的值用该点的一个邻域中各点值的中值代替，让周围的像素值接近的真实值，从而消除孤立的噪声点，对于斑点噪声（speckle noise）和椒盐噪声（salt-and-pepper noise）来说尤其有用，因为它不依赖于邻域内那些与典型值差别很大的值。中值滤波器在处理连续图像窗函数时与线性滤波器的工作方式类似，但滤波过程却不再是加权运算。

中值滤波在一定的条件下可以克服常见线性滤波器如最小均方滤波、方框滤波器、均值滤波等带来的图像细节模糊，而且对滤除脉冲干扰及图像扫描噪声非常有效，也常用于保护边缘信息, 保存边缘的特性使它在不希望出现边缘模糊的场合也很有用，是非常经典的平滑噪声处理方法。

中值滤波器与均值滤波器比较的**优势**：在均值滤波器中，由于噪声成分被放入平均计算中，所以输出受到了噪声的影响，但是在中值滤波器中，由于噪声成分很难选上，所以几乎不会影响到输出。因此同样用3x3区域进行处理，中值滤波消除的噪声能力更胜一筹。中值滤波无论是在消除噪声还是保存边缘方面都是一个不错的方法。 

中值滤波器与均值滤波器比较的**劣势**：中值滤波花费的时间是均值滤波的5倍以上。

顾名思义，中值滤波选择每个像素的邻域像素中的中值作为输出，或者说中值滤波将每一像素点的灰度值设置为该点某邻域窗口内的所有像素点灰度值的中值。

例如，取3 x 3的函数窗，计算以点[i,j]为中心的函数窗像素中值步骤如下：

1. 按强度值大小排列像素点．
2. 选择排序像素集的中间值作为点[i,j]的新值．

这一过程如图下图所示．

![](http://img.blog.csdn.net/20140408150815000?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

一般采用奇数点的邻域来计算中值，但如果像素点数为偶数
时，中值就取排序像素中间两点的平均值．采用大小不同邻域的中值滤波器的结果如图。

![](http://img.blog.csdn.net/20140408150921718?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>中值滤波在一定条件下，可以克服线性滤波器（如均值滤波等）所带来的图像细节模糊，而且对滤除脉冲干扰即图像扫描噪声最为有效。在实际运算过程中并不需要图像的统计特性，也给计算带来不少方便。但是对一些细节多，特别是线、尖顶等细节多的图像不宜采用中值滤波。

## medianBlur函数原型

medianBlur函数使用中值滤波器来平滑（模糊）处理一张图片，从src输入，而结果从dst输出。
且对于多通道图片，每一个通道都单独进行处理，并且支持就地操作（In-placeoperation）。

>void medianBlur(InputArray src,OutputArray dst, int ksize)

参数详解：

* 第一个参数，InputArray类型的src，函数的输入参数，填1、3或者4通道的Mat类型的图像；当ksize为3或者5的时候，图像深度需为CV_8U，CV_16U，或CV_32F其中之一，而对于较大孔径尺寸的图片，它只能是CV_8U。
* 第二个参数，OutputArray类型的dst，即目标图像，函数的输出参数，需要和源图片有一样的尺寸和类型。我们可以用Mat::Clone，以源图片为模板，来初始化得到如假包换的目标图。
* 第三个参数，int类型的ksize，孔径的线性尺寸（aperture linear size），注意这个参数必须是大于1的奇数，比如：3，5，7，9 ...

## 示例程序

```cpp
//
// Descriptions: 中值滤波
//
// Created by liuguoquan on 2017/10/11.
//

#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>

using namespace cv;

int main() {

    Mat src = imread("../lena.png");

    imshow("原始", src);

    Mat out;
    //中值滤波
    medianBlur(src,out,7);

    imshow("效果图",out);

    imwrite("../median_blur.png",out);

    waitKey(0);

    return 0;
}
```

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
*
* Description: 中值滤波
*
* Created by liuguoquan on 2017/10/13 14:08
*
"""


import cv2

src = cv2.imread("../lena.png")

cv2.imshow("src", src)

dst = cv2.medianBlur(src, 5)

cv2.imshow("median blur dst", dst)

cv2.waitKey(0)

```

![median_blu](http://onke0yoit.bkt.clouddn.com/median_blur.png)

# 双边滤波


双边滤波（Bilateral filter）是一种非线性的滤波方法，是结合图像的空间邻近度和像素值相似度的一种折衷处理，同时考虑空域信息和灰度相似性，达到保边去噪的目的。具有简单、非迭代、局部的特点。

双边滤波器的好处是可以做边缘保存（edge preserving），一般过去用的维纳滤波或者高斯滤波去降噪，都会较明显地模糊边缘，对于高频细节的保护效果并不明显。双边滤波器顾名思义比高斯滤波多了一个高斯方差sigma－d，它是基于空间分布的高斯滤波函数，所以在边缘附近，离的较远的像素不会太多影响到边缘上的像素值，这样就保证了边缘附近像素值的保存。但是由于保存了过多的高频信息，对于彩色图像里的高频噪声，双边滤波器不能够干净的滤掉，只能够对于低频信息进行较好的滤波。

在双边滤波器中，输出像素的值依赖于邻域像素值的加权值组合：

![](http://img.blog.csdn.net/20140408151217500?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

而加权系数w(i,j,k,l)取决于定义域核和值域核的乘积。
其中定义域核表示如下（如图）：

![](http://img.blog.csdn.net/20140408151117718?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

定义域滤波对应图示：

![](http://img.blog.csdn.net/20140408151428437?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

值域核表示为：

![](http://img.blog.csdn.net/20140408151120703?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

值域滤波：

![](http://img.blog.csdn.net/20140408151524984?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

两者相乘后，就会产生依赖于数据的双边滤波权重函数：

![](http://img.blog.csdn.net/20140408151619015?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


## bilateralFilter函数原型

>void bilateralFilter(InputArray src, OutputArraydst, int d, double sigmaColor, double sigmaSpace, int borderType=BORDER_DEFAULT)  

* 第一个参数，InputArray类型的src，输入图像，即源图像，需要为8位或者浮点型单通道、三通道的图像。
* 第二个参数，OutputArray类型的dst，即目标图像，需要和源图片有一样的尺寸和类型。
* 第三个参数，int类型的d，表示在过滤过程中每个像素邻域的直径。如果这个值我们设其为非正数，那么OpenCV会从第五个参数sigmaSpace来计算出它来。
* 第四个参数，double类型的sigmaColor，颜色空间滤波器的sigma值。这个参数的值越大，就表明该像素邻域内有更宽广的颜色会被混合到一起，产生较大的半相等颜色区域。
* 第五个参数，double类型的sigmaSpace坐标空间中滤波器的sigma值，坐标空间的标注方差。他的数值越大，意味着越远的像素会相互影响，从而使更大的区域足够相似的颜色获取相同的颜色。当d>0，d指定了邻域大小且与sigmaSpace无关。否则，d正比于sigmaSpace。
* 第六个参数，int类型的borderType，用于推断图像外部像素的某种边界模式。注意它有默认值BORDER_DEFAULT。

## 示例程序

```cpp
//
// Descriptions: 双边滤波
//
// Created by liuguoquan on 2017/10/11.
//

#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>

using namespace cv;

int main() {

    Mat src = imread("../lena.png");

    imshow("原始", src);

    Mat out;
    //双边滤波
    bilateralFilter(src,out,25,25 * 2, 25 / 2);

    imshow("效果图",out);

    imwrite("../img/bilateral_filter.png",out);

    waitKey(0);

    return 0;
}
```

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
*
* Description: 双边滤波
*
* Created by liuguoquan on 2017/10/13 14:08
*
"""


import cv2

src = cv2.imread("../lena.png")

cv2.imshow("src", src)

dst = cv2.bilateralFilter(src, 25, 25 * 2, 25 / 2)

cv2.imshow("bilateral dst", dst)

cv2.waitKey(0)


```

![bilateral_filte](http://onke0yoit.bkt.clouddn.com/bilateral_filter.png)


