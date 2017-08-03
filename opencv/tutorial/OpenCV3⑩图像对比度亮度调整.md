OpenCV3⑩图像对比度亮度调整
===

# 理论依据

首先了解一下算子的概念，一般的图像处理算子都是一个函数，它接受一个或多个输入图像，并产生输出图像。下面是算子的一般形式。

>`g(x) = (h(f(x)) 或者 g(x) = h(f0(x)......fn(x))`

图像亮度和对比度的操作，其实属于图像处理变换中比较简单的一种-点操作（point operators）。点操作有一个特点：仅仅根据输入像素值来计算相应的输出像素值。这类算子包括亮度（brightness）和对比度（contrast）调整、颜色校正（color correction）和变换（transformations）。

两种最常见的点操作是乘上一个常数（对比度的调节）以及加上一个常数（亮度值的调节）。公式如下：

>`g(x) = a * f(x) + b`

* f(x) 表示原图像像素。
* g(x) 表示输出图像像素。
* 参数 a（ a > 0）被称为增益，常用来控制图像的对比度。
* 参数 b 通常称为偏置（bias），常用来控制图像的亮度。

进一步，我们这样改写这个公式：

>`g(i,j) = a * f(i,j) + b`

其中i，j 表示像素位于第 i 行和第 j 列，这个公式可以用来作为我们在 OpenCV 中控制图像亮度和对比度的理论公式。

# 示例程序

```cpp
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

using namespace cv;

#define WINDOW_NAME "Tracker"

const int MAX_VALUE = 100;
int contrastValue; //对比度
int brightnessValue; //亮度

Mat src;
Mat dstImage;

void onTrack(int, void *) {

    for (int row = 0; row < src.rows; row++) {
        //列
        for (int col = 0; col < src.cols; col++) {

            Vec3b pixels = src.at<Vec3b>(row, col);
            dstImage.at<Vec3b>(row, col)[0] = saturate_cast<uchar>((contrastValue * 0.01) * pixels[0] + brightnessValue);
            dstImage.at<Vec3b>(row, col)[1] = saturate_cast<uchar>((contrastValue * 0.01) * pixels[1] + brightnessValue);
            dstImage.at<Vec3b>(row, col)[2] = saturate_cast<uchar>((contrastValue * 0.01) * pixels[2] + brightnessValue);
        }
    }
    imshow(WINDOW_NAME,dstImage);
    imwrite("../opencv_contrast_brightness.jpg",dstImage);
}

int main() {

    //读取图片

    src = imread("../lena.png");
    dstImage = Mat::zeros(src.size(), src.type());
    //显示图片
    imshow("Tracker", src);

    contrastValue = 60;
    brightnessValue = 50;

    namedWindow(WINDOW_NAME, WINDOW_NORMAL);

    createTrackbar("Contrast", WINDOW_NAME, &contrastValue, MAX_VALUE, onTrack);
    createTrackbar("Brightness", WINDOW_NAME, &brightnessValue, MAX_VALUE, onTrack);

    onTrack(contrastValue, 0);
    onTrack(brightnessValue, 0);

    waitKey(0);
    return 0;
}
```

![](http://onke0yoit.bkt.clouddn.com/opencv_contrast_brightness.jpg)

