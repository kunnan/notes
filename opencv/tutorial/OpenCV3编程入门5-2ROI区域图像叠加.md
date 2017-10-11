OpenCV3⑧ROI区域图像叠加
===

[TOC]

在图像处理领域，我们常常需要设置感兴趣区域（ROI，region of interest）来专注或者简化工作过程，也就是从图像中选择的一个图像区域，这个区域是图像分析所关注的重点，圈定这个区域以便进行进一步处理。使用 ROI 指定想读入的目标，可以减少处理时间，增加精度，提高图像处理的效率。

# 定义 ROI 区域

定义 ROI 区域有两种方法：

*   使用表示矩形区域的 Rect。

Rect 指定矩形的左上角坐标和矩形的长宽以定义一个矩形区域。

```cpp
Mat imageRoi;
imageRoi = image(Rect(0,0,100,100)); //image为已经载入好的图片
```

*   指定感兴趣的行或列的范围（Range）。

Range 是指从起始索引到终止索引（不包括终止索引）的一段连续序列。

```cpp
Mat imageRoi;
imageRoi = image(Range(100,100 + logo.rows),Range(100,100 + logo.cols));
```

# 示例程序

首先准备两张图片：

![lena](http://onke0yoit.bkt.clouddn.com/lena.png)

![pure](http://onke0yoit.bkt.clouddn.com/pure.png)

```cpp
//
// opencv_roi.cpp 感兴趣区域
// 
// Created by 刘涤生 on 2017/3/30 09:39.
//

#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

using namespace cv;

int main() {

    Mat mat = imread("../lena.png");
    Mat matRoi = mat(Rect(300,0,200,200));
    Mat src = imread("../pure.png");
    Mat imageRoi = src(Rect(300,0,200,200));
    //替换感兴趣区域
    imageRoi.copyTo(matRoi);
    imshow("roi",mat);
    imwrite("../opencv_roi.jpg",mat);

    waitKey(0);

}
```
![roi](http://onke0yoit.bkt.clouddn.com/opencv_roi.jpg)

