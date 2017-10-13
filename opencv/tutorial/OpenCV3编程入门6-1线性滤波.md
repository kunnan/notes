**OpenCV3⑫线性滤波**

[TOC]

# 图像滤波

**平滑处理（smoothing）**也称为模糊处理（bluring），是一种简单且使用频率很高的图像处理方法，它最常用的是用来减少图像上的噪点或者失真。

**图像滤波**是指在尽量保留图像细节特征的条件下对目标图像的噪声进行抑制，是图像预处理中不可缺少的操作，其处理效果的好坏将直接影响到后续图像处理和分析的有效性和可靠性。消除图像中的噪声成分称为图像的平滑化或者滤波操作。

图像滤波的目的有两个：一是抽出对象的特征作为图像识别的特征模式；二是为适应图像处理的要求，消除图像数字化所混入的噪声。而对滤波处理的要求也有两条：一是不能损坏图像的轮廓及边缘等重要信息；二是使图像清晰视觉效果好。

# 线性滤波器

## 简介

>线性滤波器经常用于剔除输入信号中不想要的频率或者从许多频率中选择一个想要的频率。几种常见的线性滤波器如下：

* 低通滤波器：允许低频率通过；
* 高通滤波器：允许高频率通过；
* 带通滤波器：允许一定范围频率通过；
* 带阻滤波器：阻止一定范围频率通过并且允许其他频率通过；
* 全通滤波器：允许所有频率通过，仅仅改变相位关系；
* 陷波滤波器：阻止一个狭窄频率范围通过，是一种特殊的带阻滤波器；

## 滤波与模糊

滤波可分为低通滤波和高通滤波，**低通滤波就是模糊，高通滤波就是锐化。**高斯模糊就是高斯低通滤波。

## 邻域算子和线性邻域滤波

邻域算子是利用给定像素周围的像素值决定此像素的最终输出值的一种算子。而线性邻域滤波就是一种常用的邻域算子，像素的输出值取决于输入像素的加权和。邻域算子除了用于局部色调调整以外，还可以用于图像滤波，以实现图像的平滑和锐化，图像边缘增强或者图像噪声的去除。

![邻域卷积](http://opj5rbb2n.bkt.clouddn.com/2017-06-28-邻域卷积.jpg)


上图所示为邻域滤波（卷积）-左边图像与中间图像的卷积产生右边图像。目标图像中标记的像素是利用原图像中标记的像素计算得到的。

线性滤波处理的输出像素值`g(i,j)`是输入像素值`f(i + k,j)`的加权和，如下

![](http://opj5rbb2n.bkt.clouddn.com/邻域卷积公式-1.jpg)

# OpenCV 线性滤波函数

## 方框滤波

方框滤波（box Filter）被封装在一个名为boxblur的函数中，即boxblur函数的作用是使用方框滤波器（box filter）来模糊一张图片，从src输入，从dst输出。

### boxFilter函数原型

>void boxFilter(InputArray src,OutputArray dst, int ddepth, Size ksize, Point anchor=Point(-1,-1), boolnormalize=true, int borderType=BORDER_DEFAULT )

参数详解：

* 第一个参数，InputArray类型的src，输入图像，即源图像，填Mat类的对象即可。该函数对通道是独立处理的，且可以处理任意通道数的图片，但需要注意，待处理的图片深度应该为CV_8U, CV_16U, CV_16S, CV_32F 以及 CV_64F之一。
* 第二个参数，OutputArray类型的dst，即目标图像，需要和源图片有一样的尺寸和类型。
* 第三个参数，int类型的ddepth，输出图像的深度，-1代表使用原图深度，即src.depth()。
* 第四个参数，Size类型（对Size类型稍后有讲解）的ksize，内核的大小。一般这样写Size( w,h )来表示内核的大小( 其中，w 为像素宽度， h为像素高度)。Size（3,3）就表示3x3的核大小，Size（5,5）就表示5x5的核大小
* 第五个参数，Point类型的anchor，表示锚点（即被平滑的那个点），注意他有默认值Point(-1,-1)。如果这个点坐标是负值的话，就表示取核的中心为锚点，所以默认值Point(-1,-1)表示这个锚点在核的中心。
* 第六个参数，bool类型的normalize，默认值为true，一个标识符，表示内核是否被其区域归一化（normalized）了。
* 第七个参数，int类型的borderType，用于推断图像外部像素的某种边界模式。有默认值BORDER_DEFAULT，我们一般不去管它。

boxFilter（）函数方框滤波所用的核为：

![](http://img.blog.csdn.net/20140401183028203?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其中：

![](http://img.blog.csdn.net/20140401183051640?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 示例代码

```cpp
//
// Descriptions: 方框滤波
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
    //方框滤波
    boxFilter(src,out,-1,Size(5,5));

    imshow("效果图",out);

    imwrite("../box_blur.png",out);

    waitKey(0);

    return 0;
}

```

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
*
* Description: 方框滤波
*
* Created by liuguoquan on 2017/10/11 16:47
*
"""

import cv2

src = cv2.imread("../lena.png")

cv2.imshow("src", src)

dst = cv2.boxFilter(src, -1, (5, 5))

cv2.imshow("dst", dst)

cv2.waitKey(0)
```

![box_blu](http://onke0yoit.bkt.clouddn.com/box_blur.png)


## 均值滤波

均值滤波是典型的线性滤波算法，主要方法为邻域平均法，即用一片图像区域的各个像素的均值来代替原图像中的各个像素值。一般需要在图像上对目标像素给出一个模板（内核），该模板包括了其周围的临近像素（比如以目标像素为中心的周围8（3x3-1）个像素，构成一个滤波模板，即去掉目标像素本身）。再用模板中的全体像素的平均值来代替原来像素值。即对待处理的当前像素点（x，y），选择一个模板，该模板由其近邻的若干像素组成，求模板中所有像素的均值，再把该均值赋予当前像素点（x，y），作为处理后图像在该点上的灰度个g（x，y），即个g（x，y）=1/m ∑f（x，y） ，其中m为该模板中包含当前像素在内的像素总个数。

均值滤波本身存在着固有的缺陷，即它不能很好地保护图像细节，在图像去噪的同时也破坏了图像的细节部分，从而使图像变得模糊，不能很好地去除噪声点。

OpenCV中使用均值滤波——blur函数,blur函数文档中，给出的其核是这样的：

![](http://img.blog.csdn.net/20140401183602687?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这个内核一看就明了，就是在求均值，即blur函数封装的就是均值滤波。

### blur函数原型

>void blur(InputArray src, OutputArraydst, Size ksize, Point anchor=Point(-1,-1), int borderType=BORDER_DEFAULT )

* 第一个参数，InputArray类型的src，输入图像，即源图像，填Mat类的对象即可。该函数对通道是独立处理的，且可以处理任意通道数的图片，但需要注意，待处理的图片深度应该为CV_8U, CV_16U, CV_16S, CV_32F 以及 CV_64F之一。
* 第二个参数，OutputArray类型的dst，即目标图像，需要和源图片有一样的尺寸和类型。比如可以用Mat::Clone，以源图片为模板，来初始化得到如假包换的目标图。
* 第三个参数，Size类型（对Size类型稍后有讲解）的ksize，内核的大小。一般这样写Size( w,h )来表示内核的大小( 其中，w 为像素宽度， h为像素高度)。Size（3,3）就表示3x3的核大小，Size（5,5）就表示5x5的核大小
* 第四个参数，Point类型的anchor，表示锚点（即被平滑的那个点），注意他有默认值Point(-1,-1)。如果这个点坐标是负值的话，就表示取核的中心为锚点，所以默认值Point(-1,-1)表示这个锚点在核的中心。
* 第五个参数，int类型的borderType，用于推断图像外部像素的某种边界模式。有默认值BORDER_DEFAULT，我们一般不去管它。

### 示例代码

```cpp
//
// Descriptions: 均值滤波
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
    //均值滤波
    blur(src,out,Size(7,7));

    imshow("效果图",out);

    imwrite("../mean_blur.png",out);

    waitKey(0);

    return 0;
}
```

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
*
* Description: 均值滤波
*
* Created by liuguoquan on 2017/10/11 16:47
*
"""

import cv2

src = cv2.imread("../lena.png")

cv2.imshow("src", src)

dst = cv2.blur(src, (5, 5))

cv2.imshow("averaging dst", dst)

cv2.waitKey(0)
```

![mean_blu](http://onke0yoit.bkt.clouddn.com/mean_blur.png)


## 高斯滤波

高斯滤波是一种线性平滑滤波，适用于消除高斯噪声，广泛应用于图像处理的减噪过程。通俗的讲，高斯滤波就是对整幅图像进行加权平均的过程，每一个像素点的值，都由其本身和邻域内的其他像素值经过加权平均后得到。高斯滤波的具体操作是：用一个模板（或称卷积、掩模）扫描图像中的每一个像素，用模板确定的邻域内像素的加权平均灰度值去替代模板中心像素点的值。

高斯模糊技术生成的图像，其视觉效果就像是经过一个半透明屏幕在观察图像，这与镜头焦外成像效果散景以及普通照明阴影中的效果都明显不同。高斯平滑也用于计算机视觉算法中的预先处理阶段，以增强图像在不同比例大小下的图像效果（参见尺度空间表示以及尺度空间实现）。从数学的角度来看，图像的高斯模糊过程就是图像与正态分布做卷积。由于正态分布又叫作高斯分布，所以这项技术就叫作高斯模糊。

图像与圆形方框模糊做卷积将会生成更加精确的焦外成像效果。由于高斯函数的傅立叶变换是另外一个高斯函数，所以高斯模糊对于图像来说就是一个低通滤波操作。

 高斯滤波器是一类根据高斯函数的形状来选择权值的线性平滑滤波器。高斯平滑滤波器对于抑制服从正态分布的噪声非常有效。一维零均值高斯函数为：
 
![](http://img.blog.csdn.net/20140401202935953?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其中，高斯分布参数Sigma决定了高斯函数的宽度。对于图像处理来说，常用二维零均值离散高斯函数作平滑滤波器。

二维高斯函数为：

![](http://img.blog.csdn.net/20140401203030078?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcG9lbV9xaWFubW8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### GaussianBlur函数原型

GaussianBlur函数的作用是用高斯滤波器来模糊一张图片，对输入的图像src进行高斯滤波后用dst输出。它将源图像和指定的高斯核函数做卷积运算，并且支持就地过滤（In-placefiltering）

>void GaussianBlur(InputArray src,OutputArray dst, Size ksize, double sigmaX, double sigmaY=0, intborderType=BORDER_DEFAULT )  

* 第一个参数，InputArray类型的src，输入图像，即源图像，填Mat类的对象即可。它可以是单独的任意通道数的图片，但需要注意，图片深度应该为CV_8U,CV_16U, CV_16S, CV_32F 以及 CV_64F之一。
* 第二个参数，OutputArray类型的dst，即目标图像，需要和源图片有一样的尺寸和类型。比如可以用Mat::Clone，以源图片为模板，来初始化得到如假包换的目标图。
* 第三个参数，Size类型的ksize高斯内核的大小。其中ksize.width和ksize.height可以不同，但他们都必须为正数和奇数。或者，它们可以是零的，它们都是由sigma计算而来。
* 第四个参数，double类型的sigmaX，表示高斯核函数在X方向的的标准偏差。
* 第五个参数，double类型的sigmaY，表示高斯核函数在Y方向的的标准偏差。若sigmaY为零，就将它设为sigmaX，如果sigmaX和sigmaY都是0，那么就由ksize.width和ksize.height计算出来。
* 为了结果的正确性着想，最好是把第三个参数Size，第四个参数sigmaX和第五个参数sigmaY全部指定到。
* 第六个参数，int类型的borderType，用于推断图像外部像素的某种边界模式。有默认值BORDER_DEFAULT，我们一般不去管它。

### 示例代码

```cpp
//
// Descriptions: 高斯滤波
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
    //高斯滤波
    GaussianBlur(src,out,Size(5,5),0,0);

    imshow("效果图",out);

    imwrite("../gaussian_blur.png",out);

    waitKey(0);

    return 0;
}
```

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
*
* Description: 高斯滤波
*
* Created by liuguoquan on 2017/10/11 16:47
*
"""

import cv2

src = cv2.imread("../lena.png")

cv2.imshow("src", src)

dst = cv2.GaussianBlur(src, (5, 5), 0)

cv2.imshow("gaussian dst", dst)

cv2.waitKey(0)
```

![gaussian_blu](http://onke0yoit.bkt.clouddn.com/gaussian_blur.png)

