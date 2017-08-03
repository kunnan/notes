OpenCV3⑥图像颜色空间转换
===

[TOC]

# cvtColor()

`cvtColor()`函数是 OpenCV 里的颜色空间转换函数，可以实现 RGB 颜色向 HSV、HSI 等颜色空间的转换，也可以转换为灰度图像。原型是

```
void cvtColor( InputArray src, OutputArray dst, int code, int dstCn = 0 )
```
*   src：输入图像。
*   dst：输出图像。
*   code：颜色空间转换的标识符。常用的有COLOR_BGR2GRAY、COLOR_GRAY2BGR、COLOR_BGR2HSV 、COLOR_HSV2BGR等。
*   dstCn：目标图像的通道数，0 表示目标图像读取源图像的通道数。

下面是一个调用示例：

```cpp
//openCV2
cvtColor(src, dst, CV_GRAY2BGR); //转换为灰度图
//openCV3
cvtColor(src, dst, COLOR_GRAY2BGR); //转换为灰度图
```

# 示例程序

```cpp
//
// opencv_cvt_color.cpp 颜色空间转换
// 
// Created by 刘涤生 on 2017/3/29 14:13.
//

#include <opencv2/imgproc/imgproc.hpp>
#include <opencv2/highgui/highgui.hpp>

using namespace cv;

int main() {

    Mat image = imread("../lena.png");
    Mat dst;
    //转为HSV
    cvtColor(image,dst,COLOR_BGR2HSV);
    imwrite("../lena_hsv.jpg",dst);

    //转为灰度图片
    cvtColor(image,dst,COLOR_BGR2GRAY);
    imwrite("../lena_gray.jpg",dst);

    waitKey(0);
}
```
*   BGR图片

![](http://onke0yoit.bkt.clouddn.com/lena.png)

*   HSV图片

![](http://onke0yoit.bkt.clouddn.com/lena_hsv.jpg)

*   GRAY灰度图片

![](http://onke0yoit.bkt.clouddn.com/lena_gray.jpg)


