OpenCV3⑪离散傅里叶变换
===
[TOC]

# 简介

对一张图片使用傅里叶变换就是将它分解成正弦和余弦两部分，也就是将图像从空间域（spatial domain）转换到频域（frequency domain）。这一转换的理论基础为：任意函数都可以表示成无数个正弦和余弦函数的和的形式，傅里叶变换就是一个用来将函数分解的工具。

二位图像的傅里叶变换可以用以下数学公式表达。

![](http://onke0yoit.bkt.clouddn.com/opencv_dft_formula.jpg)

其中 f 是空间域值，F 是频域值。转换之后的频域值是复数，因此，显示傅里叶变换之后的结果需要使用实数图像（real image）加虚数图像（complex image），或者幅度图像（magitude image）叫相位图像（phase image）的形式。在实际的图像处理过程中，仅仅使用了幅度图像，因为幅度图像包含了原图像的几乎所有我们需要的几何信息。然而如果想通过修改幅度图像或者相位图像的方法来间接修改原空间图像，需要使用逆傅里叶变换得到修改后的空间图像，这样就必须同时保留幅度图像和相位图像了。

在频域里面，对于一幅图像，高频部分代表了图像的细节、纹理信息；低频部分代表了图像的轮廓信息。如果对一幅精细的图像使用低通滤波器，那么滤波后的结果就只剩下轮廓了。如果图像受到的噪声恰好位于某个特定的“频率”范围内，则可以通过滤波器来恢复原来的图像。傅里叶变换在图像处理中可以做到图像增强与图像去噪、图像分割之边缘检测、图像特征提取、图像压缩等。

# 相关函数

## dft()

dft 的作用是对一维或二维浮点数组进行正向或反向离散傅里叶变换。C++ 函数原型如下：

```cpp
void dft(InputArray src, OutputArray dst, int flags = 0, int nonzeroRows = 0)
```
*   src：输入矩阵，可以为实数或者虚数
*   dst：函数运算后的结果存在dst，其尺寸和类型取决于第三个参数 flags。
*   flags：转换的标识符，有默认值 0 ，取值可以有：


| 标识符名称 | 意义 
| --- | --- 
| DFT_INVERSE | 用一维或二维逆变替代默认的正向变换
| DFT_SCALE |  缩放比例标识符，输出的结果都会以 1 / N 进行缩放，通常结合 DFT_INVERSE 一起使用
| DFT_ROWS |  对输入矩阵的每行进行正向或反向的变换，此标识符可以在处理多种矢量的时候用于减少资源开销，这些处理常常是三维或高维变换等复杂操作
| DFT_COMPLEX_OUTPUT |  进行一维或二维实数数组正变换。这样的结果虽然是复数矩阵，但拥有复数的共轭对称性，所以可以被写成一个拥有同样尺寸的实数阵列
| DFT_REAL_OUTPUT |  进行一维或二维实数数组反变换。这样的结果通常是一个大小相同的复数矩阵，如果输入的矩阵有复数的共轭对称性，便会输出实矩阵

*   nonzeroRows：

## getOptimalDFTSize()

getOptimalDFTSize 函数返回给定向量尺寸的傅里叶最优尺寸大小。为了提高离散傅里叶变换的运行速度，需要扩展图像，而具体扩充多少，就有这个函数计算得到，C++ 函数如下：

```cpp
int getOptimalDFTSize(int vecsize)
```
*   vecsize：向量尺寸，即图片的rows、cols。

## copyMakeBorder()

copyMakeBorder 函数的作用是扩充图像边界。C++ 函数原型如下：

```cpp
void copyMakeBorder(InputArray src, OutputArray dst,
                                 int top, int bottom, int left, int right,
                                 int borderType, const Scalar& value = Scalar() )
```
* src：源图像
* dst：输出图像，需要和源图片有一样的尺寸和类型，且 Size 应该为`Size(src.cols + left + right,src.row + top + bottom)`
* top、bottom、left、right：分别表示在源图像的四个方向上扩充多少像素
* borderType：边界类型，常见取值为 BORDER_CONSTANT
* value：当 borderType 取值 BORDER_CONSTANT 时，这个参数表示边界值

## magnitude()

magnitude 函数用于计算二位矢量的幅值。C++ 函数原型为：

```cpp
void magnitude(InputArray x, InputArray y, OutputArray magnitude)
```
* x：表示矢量的浮点型 X 坐标值，也就是实部
* y：表示矢量的浮点型 Y 坐标值，也就是虚部
* magnitude：输出的幅值，它和第一个参数 x 有着同样的尺寸和类型。

magnitude 函数的原理为：

![mangnitude原理](http://7xs7a3.com1.z0.glb.clouddn.com/mangnitude原理.png)

## log()

log 函数的功能是计算每个数组元素绝对值的自然对数。C++ 函数原型:

```cpp
void log(InputArray src, OutputArray dst)
```

## normalize()

normalize 函数的作用是进行矩阵归一化。C++ 函数原型如下：

```cpp
void normalize( InputArray src, InputOutputArray dst, double alpha = 1, double beta = 0,
                             int norm_type = NORM_L2, int dtype = -1, InputArray mask = noArray())
```
*   src：输入图像
*   dst：输出图像，和源图片有一样的尺寸和类型
*   alpha：归一化后的最大值，默认值 1
*   beta：归一化后的最大值，默认值 0
*   norm_type：归一化类型有 NORM_INF、NORM_L1、NORM_L2、NORM_MINMAX等，默认值为 NORM_L2
*   dtype：默认值 -1。当为负数时，输出矩阵和 src 有同样的类型，否则，它和 src 有同样的通道数，且此时图像深度为 CV_MATH_DEPTH（dtype）。
*   mask：可选的操作掩膜，有默认值 noArray()

# 示例程序

下面展示如何计算以及显示傅里叶变换后的幅度图像，下面的步骤将以输入图像为单通道的灰度图像为例：

*   第一步：载入原始图像
*   第二步：将图像扩大到合适的尺寸
*   第三步：为傅里叶变换的结果（实部和虚部）分配存储空间
*   第四步：进行离散傅里叶变换
*   第五步：将复数转换为幅值，复数包含实数部分（Re）和虚数部分（Im）。离散傅里叶变换的结果是复数，对应的幅度可以表示为

    ![离散傅里叶幅度](http://7xs7a3.com1.z0.glb.clouddn.com/离散傅里叶幅度.png)


*   第六步：进行对数尺度缩放
*   第七步：剪切和重分布幅度图象限
*   第八步：归一化，现在有了重分布的幅度值，但是幅度值仍然超过可显示范围 [0,1]，我们使用 normalize 函数将幅度归一化到可显示范围。

```cpp
//
// opencv_dft.cpp 离散傅里叶变换
// 
// Created by 刘涤生 on 2017/3/30 15:27.
//

#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

using namespace cv;

int main() {

    //1.以灰度读取原始图像并显示
    Mat image = imread("../lena.png",0);
    imshow("Source Image",image);

    //2.将输入图像扩展到最近尺寸，边界用0补充
    int m = getOptimalDFTSize(image.rows);
    int n = getOptimalDFTSize(image.cols);

    //扩充图像边界
    Mat padded;
    copyMakeBorder(image,padded,0,m - image.rows,0,n - image.cols,BORDER_CONSTANT,Scalar::all(0));

    //3.为傅里叶变换的结果（实部和虚部）分配存储空间
    Mat planes[] = {Mat_<float>(padded),Mat::zeros(padded.size(),CV_32F)};
    Mat complexI;
    //将planes[]和合并成一个多通道的complexI
    merge(planes,2,complexI);

    //4.进行离散傅里叶变换
    dft(complexI,complexI);

    //5.将复数转换为幅值
    //将多通道的complexI分离成单通道数组
    split(complexI,planes);
    magnitude(planes[0],planes[1],planes[0]);
    Mat magnitudeImage = planes[0];

    //6.进行对数尺度缩放
    magnitudeImage += Scalar::all(1);
    log(magnitudeImage,magnitudeImage);

    //7.剪切和重分布幅度图象限
    //若有奇数行或奇数列，进行频谱剪裁
    magnitudeImage = magnitudeImage(Rect(0,0,magnitudeImage.cols & -2,magnitudeImage.rows & -2));

    //重新排列傅里叶图像的象限，使得原点位于图像中心
    int cx = magnitudeImage.cols / 2;
    int cy = magnitudeImage.rows / 2;
    Mat q0(magnitudeImage,Rect(0,0,cx,cy));
    Mat q1(magnitudeImage,Rect(cx,0,cx,cy));
    Mat q2(magnitudeImage,Rect(0,cy,cx,cy));
    Mat q3(magnitudeImage,Rect(cx,cy,cx,cy));

    //交换象限（左上和右下进行交换）
    Mat tmp;
    q0.copyTo(tmp);
    q3.copyTo(q0);
    tmp.copyTo(q3);

    //交换象限（右上和左下进行交换）
    q1.copyTo(tmp);
    q2.copyTo(q1);
    tmp.copyTo(q2);

    //8.归一化，用0到1之间的浮点值将矩阵变为可视的图像格式
    normalize(magnitudeImage,magnitudeImage,0,1,NORM_MINMAX);

    //显示效果图
    imshow("DFT",magnitudeImage);

    waitKey(0);

}
```
*   灰度图

![](http://onke0yoit.bkt.clouddn.com/lena_gray.jpg)

*   DFT

![](http://onke0yoit.bkt.clouddn.com/opencv_dft_dft.jpg)

