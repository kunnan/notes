OpenCV3⑨分离颜色通道
===

[TOC]

为了更好地观察一些图像材料的特征，有时需要对 RGB 三个颜色通道的分量进行分别显示和调整。通过 OpenCV 的 spit 和merge 方法可以很方便地达到目的。

# 通道分离

split 函数用于将一个多通道数组分离成几个单通道数组，它的 C++ 函数原型是

```cpp
void split(const Mat& src, Mat* mvbegin)
void split(InputArray m, OutputArrayOfArrays mv)
```

* src 或者 m：需要进行分离的多通道数组
* mvbegin 或者 mv：输出数组或者输出的 vector 容器。

# 通道合并

merge 函数是 spit 函数的逆向操作，它将多个数组合并成一个多通道的数组。它通过组合一些给定的单通道数组，将这些孤立的单通道数组合并成一个多通道数组，从而创建出一个由多个单通道阵列组成的多通道阵列。它 C++ 的函数原型如下：

```cpp
void merge(const Mat* mv, size_t count, OutputArray dst)
void merge(InputArrayOfArrays mv, OutputArray dst);
```
*   mv：需要被合并的输入矩阵或 vector 容器的阵列，这个 mv 参数中的所有矩阵必须有着一样的尺寸和深度。
*   count：输入矩阵 mv（mv 为数组时） 的个数
*   dst：输出矩阵

# 示例程序

```c
#include <opencv2/core/core.hpp>  
#include <opencv2/highgui/highgui.hpp>  
#include <vector>  

using namespace cv;
using namespace std;

int main() {

    Mat mat = imread("../lena.png");
    vector<Mat> channels;
    split(mat,channels);
    Mat imageBlue,imageGreen,imageRed;
    imageBlue = channels.at(0);
    imageGreen = channels.at(1);
    imageRed = channels.at(2);
    imshow("b",imageBlue);
    imshow("g",imageGreen);
    imshow("r",imageRed);
    Mat dst;
    merge(channels,dst);
    imshow("merge",dst);
    imwrite("../split_blue.jpg",imageBlue);
    imwrite("../split_green.jpg",imageGreen);
    imwrite("../split_red.jpg",imageRed);
    waitKey(0);

}
```

*   蓝色通道

![](http://onke0yoit.bkt.clouddn.com/split_blue.jpg)

*   绿色通道

![](http://onke0yoit.bkt.clouddn.com/split_green.jpg)

*   红色通道

![](http://onke0yoit.bkt.clouddn.com/split_red.jpg)

*   合并

![](http://onke0yoit.bkt.clouddn.com/lena.png)


