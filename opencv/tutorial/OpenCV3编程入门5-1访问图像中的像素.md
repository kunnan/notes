OpenCV3⑤访问图像中的像素
===

[TOC]

>任何图像处理算法，都是从操作每个像素开始的。OpenCV中的彩色图像不是以 RGB 的顺序存放的，而是 BGR 顺序存放的。

# 指针方法

```cpp
Mat ptr(Mat src) {

    Mat mat = Mat::zeros(src.size(),CV_8UC3);
    //行
    for (int row = 0; row < src.rows; row++) {
        //列
        for (int col = 0; col < src.cols; col++) {
            //行索引
            const uchar* cuurentRow = src.ptr(row);
            uchar* myRow = mat.ptr(row);

            if (src.channels() == 1) {

                myRow[col] = 255 - cuurentRow[col];

            } else if (src.channels() == 3) {

                myRow[col*3] = 255 - cuurentRow[col*3];
                myRow[col*3 + 1] = 255 - cuurentRow[col*3 + 1];
                myRow[col*3 + 2] = 255 - cuurentRow[col*3 + 2];

            } else {

                printf("未知的图像格式\n");
            }
        }
    }
    return mat;

}
```

# 动态地址

```cpp
Mat at(Mat src) {

    Mat mat = Mat::zeros(src.size(),CV_8UC3);
    //行
    for (int row = 0; row < src.rows; row++) {
        //列
        for (int col = 0; col < src.cols; col++) {

            if (src.channels() == 1) {

                mat.at<uchar>(row,col) = 255 - src.at<uchar>(row,col);

            } else if (src.channels() == 3) {

                Vec3b pixels = src.at<Vec3b>(row,col);
                mat.at<Vec3b>(row,col)[0] = 255 - pixels[0]; //B
                mat.at<Vec3b>(row,col)[1] = 255 - pixels[1]; //G
                mat.at<Vec3b>(row,col)[2] = 255 - pixels[2]; //R

            } else {

                printf("未知的图像格式\n");
            }
        }
    }
    return mat;
}
```
>Mat类中的 cols 和 rows 给出了图像的宽和高，而成员函数`at (int x, int y)`可以用来存储图像元素，但是必须在编译期间知道图像的数据类型。需要注意的是，一定要确保指定的数据类型要和矩阵中的数据类型相符合，因为 at 方法本身不会对任何数据类型进行转换。
>对于彩色图像，每个像素由三个部分构成：蓝色通道、绿色通道和红色通道（BGR）。因此。对于一个包含彩色图像的 Mat，会返回一个由三个 8 位数组成的向量。OpenCV将此类型定义为 Vec3b，即由三个 unsigned char 组成的向量。
 
# 示例程序

```cpp
//
// opencv_get_piexls.cpp OpenCV获取图像像素
// 
// Created by 刘涤生 on 2017/3/24 17:50.
//

#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

using namespace cv;

Mat ptr(Mat src) {

    Mat mat = Mat::zeros(src.size(),CV_8UC3);
    //行
    for (int row = 0; row < src.rows; row++) {
        //列
        for (int col = 0; col < src.cols; col++) {
            //行索引
            const uchar* cuurentRow = src.ptr(row);
            uchar* myRow = mat.ptr(row);

            if (src.channels() == 1) {

                myRow[col] = 255 - cuurentRow[col];

            } else if (src.channels() == 3) {

                myRow[col*3] = 255 - cuurentRow[col*3];
                myRow[col*3 + 1] = 255 - cuurentRow[col*3 + 1];
                myRow[col*3 + 2] = 255 - cuurentRow[col*3 + 2];

            } else {

                printf("未知的图像格式\n");
            }
        }
    }
    return mat;

}

/**
 * 将图片进行每个像素点的每个通道的像素值取反
 * @param src
 * @return
 */
Mat at(Mat src) {

    Mat mat = Mat::zeros(src.size(),CV_8UC3);
    //行
    for (int row = 0; row < src.rows; row++) {
        //列
        for (int col = 0; col < src.cols; col++) {

            if (src.channels() == 1) {

                mat.at<uchar>(row,col) = 255 - src.at<uchar>(row,col);

            } else if (src.channels() == 3) {

                Vec3b pixels = src.at<Vec3b>(row,col);
                mat.at<Vec3b>(row,col)[0] = 255 - pixels[0];
                mat.at<Vec3b>(row,col)[1] = 255 - pixels[1];
                mat.at<Vec3b>(row,col)[2] = 255 - pixels[2];

            } else {

                printf("未知的图像格式\n");
            }
        }
    }
    return mat;
}

int main() {

    Mat mat = imread("../lena.png");

    Mat dst = at(mat);

    imshow("Pixels",dst);
    waitKey(0);

}
```
*   原图

![](http://onke0yoit.bkt.clouddn.com/lena.png)

*  原图取反

![](http://onke0yoit.bkt.clouddn.com/lena_not.jpg)

