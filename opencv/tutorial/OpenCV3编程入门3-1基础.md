OpenCV3①基础
===

[TOC]

# 概述

OpenCV 的全称是「Open Source Computer Vision Library」，意思就是「开源计算机视觉库」。OpenCV 的设计目标是执行速度尽量快，主要关注实时应用。OpenCV 又一系列的 C 函数和 C++ 函数构成，拥有包括500多个 C 函数的中高层 API。它不依赖于其他外部库-如果有需要，也可以使用某些外部库。OpenCV 覆盖类计算机视觉的许多应用领域，如产品检测、医学成像、信息安全、用户界面、摄像机标定、人脸识别、立体视觉和机器人等，OpenCV 还提供了机器学习库，主要用于统计方面的模式识别和聚类。


# 基本架构

OpenCV 的库文件主要包括 opencv 和 opencv2 两个文件夹。opencv 文件夹里面包括旧版的头文件，而opencv2 这个文件夹包含着具有时代意义的新版 OpenCV2 系列的头文件。下面主要介绍 opencv2 文件夹中的内容。

opencv2 文件夹中有个名为 opencv_modules.hpp的 hpp 文件，里面存放的是 OpenCV2 中的与新模块构造相关的说明代码，打开可以看到其定义的是 OpenCV2所有组件的宏，具体如下：

```c
#define HAVE_OPENCV_CALIB3D
#define HAVE_OPENCV_CORE
#define HAVE_OPENCV_FEATURES2D
#define HAVE_OPENCV_FLANN
#define HAVE_OPENCV_HIGHGUI
#define HAVE_OPENCV_IMGCODECS
#define HAVE_OPENCV_IMGPROC
#define HAVE_OPENCV_ML
#define HAVE_OPENCV_OBJDETECT
#define HAVE_OPENCV_PHOTO
#define HAVE_OPENCV_SHAPE
#define HAVE_OPENCV_STITCHING
#define HAVE_OPENCV_SUPERRES
#define HAVE_OPENCV_VIDEO
#define HAVE_OPENCV_VIDEOIO
#define HAVE_OPENCV_VIDEOSTAB
```
下面介绍 OpenCV 的所有模块：

*   **calib3d**：主要是相机校准和三维重建相关的内容，包括基本的多视角几何算法、单个立体摄像机标定、物体姿态估计、立体相似性算法、3、信息的重建等。

*   **core**：核心功能模块，包含以下的内容：OpenCV 基本数据结构、动态数据结构、绘图函数、数组操作相关函数、辅助功能与系统函数和宏、与OpenGL 的互操作。

*   **features2d**：2D功能框架，包括特征检测和描述、特征检测器「Feature Detectors」通用接口、描述符提取器「Descriptor Extractors」通用接口、描述符匹配器「Descriptor Matchers」通用接口、通用描述符匹配器「Generic Descriptor」通用接口、关键点绘制函数和匹配功能绘制函数。

*   **flann**：高维近似近邻快速搜索算法库，包括快速近似最近邻搜索、聚类。

*   **highgui**：高层GUI图形用户界面，包含媒体的输入输出、视频捕捉、图形交互界面的接口等内容。

*   **imgcodecs**：图片文件的输入和输出，包括图像的编码解码

*   **imgproc**：图像处理模块，包括线性和非线性的图像滤波、图像的几何变换、其他图像转换、直方图相关、结构分析和形状描述、运动分析和对象跟踪、特征检测、目标检测等内容。

*   **ml**：机器学习模块，基本上是统计模型和分类算法，包括统计模型、一般贝叶斯分类器、K-近邻、支持向量机、决策树、提升、梯度提高树、随机树、超随机树、期望最大化、神经网络、ML Data。

*   **objectect**，目标检测模块，包含级联分类「Cascade Classification」和 Latent SVM。

*   **photo**，计算摄影学，包含图像修复、图像去噪。

*   **shape**，Shape Distance and Matching

*   **stitching**，图像拼接模型，包括拼接流水线、特点寻找和匹配图像、估计旋转、自动校准、图片歪斜、接缝估测、曝光补偿、图片混合。

*   **superres**，超分辨率技术的相关功能模块。

*   **video**，视频分析组件，包括运动估计、背景分离、对象跟踪等视频处理相关内容。

*   **videoio**，视频输入输出模块。

*   **videostab**，视频稳定相关的组件。

# OpenCV的命名空间

OpenCV 中的 C++ 类和函数都是定义在命名空间 cv 之内的，有两种方法可以访问：第一种是`using namespace cv`，另一种是，在使用 OpenCV 的每一个类和函数时都加入 cv:: 命名空间。一般写法如下:

```cpp
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

using namespace cv;

或者

#include <opencv2/opencv.hpp>

using namespace cv;
```

# 图像的输入显示输出

## Mat 类

OpenCV 2.0 以来，OpenCV 采用了 Mat 类作为数据结构进行图像的存储，新版的 OpenCV 中有些函数名都和 Matlab 一样，比如 `imread`、`imwrite`、`imshow`等函数。

## imread()函数

>用于读取文件中的图片到 OpenCV 中。原型是`Mat 	cv::imread (const String &filename, int flags=IMREAD_COLOR)`

* filename：图片的路径名。支持如下类型的图像载入：

    *   Windows位图：`*.bmp,*dib`
    *   JPEG文件：`*.jpeg *.jpg *.jpe`
    *   JPEG2000文件：`*.jp2`
    *   PNG图片：`*.png`
    *   便携文件格式：`*.pbm, *.pgm, *.ppm *.pxm, *.pnm`
    *   Sun rasters 光栅文件：`*.sr, *.ras`
    *   TIFF文件：`*.tiff, *.tif`
    *   OpenEXR 图片文件 - `*.exr`
    *   Radiance HDR - `*.hdr, *.pic`

* flag：载入标识，默认为`IMREAD_COLOR`

```
//imagecodes模块
enum  	cv::ImreadModes { 
  cv::IMREAD_UNCHANGED = -1, //返回包含Alpha通道的图片
  cv::IMREAD_GRAYSCALE = 0,  //返回灰度图像
  cv::IMREAD_COLOR = 1,     // 返回一个3通道的图像
  cv::IMREAD_ANYDEPTH = 2, 
  cv::IMREAD_ANYCOLOR = 4, 
  cv::IMREAD_LOAD_GDAL = 8, 
  cv::IMREAD_REDUCED_GRAYSCALE_2 = 16, 
  cv::IMREAD_REDUCED_COLOR_2 = 17, 
  cv::IMREAD_REDUCED_GRAYSCALE_4 = 32, 
  cv::IMREAD_REDUCED_COLOR_4 = 33, 
  cv::IMREAD_REDUCED_GRAYSCALE_8 = 64, 
  cv::IMREAD_REDUCED_COLOR_8 = 65, 
  cv::IMREAD_IGNORE_ORIENTATION = 128 
}
```

## inshow()函数

>imshow()用于在指定的窗口中显示一幅图像，函数原型为`cv::imshow (const String &winname, InputArray mat)`

*   winname：显示的窗口标识名称。
*   mat： 显示的图像。

## namedWindow()函数

>namedWindow函数用于创建一个窗口，函数原型为`cv::namedWindow (const String &winname, int flags=WINDOW_AUTOSIZE)`

* winname：指定窗口标识名称。
* flags：窗口的标识，有如下几种值：

    *   WINDOW_NORMAL：用户可以改变窗口的大小。
    *   WINDOW_AUTOSIZE：窗口大小会自动地调整以适应所显示的图像，并且用户不能手动改变窗口大小。

## imwrite()函数

>imwrite()函数用于将图像输出到指定文件，函数原型是`cv::imwrite (const String &filename, InputArray img, const std::vector< int > &params=std::vector< int >())`

*   filename：写入的文件名。
*   img：Mat 类型的图像数据
*   params： 表示为特定格式保存的参数编码

图像格式是基于文件扩展名的，可保存的扩展名和 `imread` 中可以读取的图像扩展名一致。只有 8-bit (或者 16-bit unsigned (CV_16U) in case of PNG, JPEG 2000, and TIFF) 单通道 或者 3-通道的图片能够通过此函数保存。否则先使用 `Mat::convertTo` 和 `cv::cvtColor to` 转换后再保存. 或者, 使用通用的 `FileStorage I/O`函数将图片保存为 XML 或者 YAML 格式。

下面示例讲解如何创建一幅 BGRA 图片并保存为 png 图片文件：

```cpp
#include <opencv2/opencv.hpp>
using namespace cv;
using namespace std;

void createAlphaMat(Mat &mat)
{
    CV_Assert(mat.channels() == 4);
    for (int i = 0; i < mat.rows; ++i) {
        for (int j = 0; j < mat.cols; ++j) {
            Vec4b& bgra = mat.at<Vec4b>(i, j);
            bgra[0] = UCHAR_MAX; // Blue
            bgra[1] = saturate_cast<uchar>((float (mat.cols - j)) / ((float)mat.cols) * UCHAR_MAX); // Green
            bgra[2] = saturate_cast<uchar>((float (mat.rows - i)) / ((float)mat.rows) * UCHAR_MAX); // Red
            bgra[3] = saturate_cast<uchar>(0.5 * (bgra[1] + bgra[2])); // Alpha
        }
    }
}
int main(int argv, char **argc)
{
    // Create mat with alpha channel
    Mat mat(480, 640, CV_8UC4);
    createAlphaMat(mat);
    vector<int> compression_params;
    compression_params.push_back(IMWRITE_PNG_COMPRESSION);
    compression_params.push_back(9);
    try {
        imwrite("alpha.png", mat, compression_params);
    }
    catch (cv::Exception& ex) {
        fprintf(stderr, "Exception converting image to PNG format: %s\n", ex.what());
        return 1;
    }
    fprintf(stdout, "Saved PNG file with alpha data.\n");
    return 0;
}
```

## 综合示例

```c
//
// opencv_show.cpp OpenCV读取、显示、保存图像
//
// Created by 刘涤生 on 2017/3/24 14:40.
//

#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>

using namespace cv;

int main() {

    //读取图片

    Mat image = imread("../lena.png");
    //显示图片
    imshow("原始",image);

    Mat dstImage;
    //均值滤波处理
    blur(image, dstImage, Size(7, 7));

    imshow("Blur", dstImage);

    //保存图片
    imwrite("../lena_blur.jpg",dstImage);
    waitKey(0);
    return 0;
}
```
*   原图

![](http://onke0yoit.bkt.clouddn.com/lena.png)

*   均值滤波处理

![](http://onke0yoit.bkt.clouddn.com/lena_blur.jpg)

# HighGUI

## 滑动条

滑动条（TrackBar）是 OpenCV 动态调节参数特别好用的一种工具，它依附于窗口而存在。

### 创建滑动条

>createTrackerbar函数用于创建一个可以调整数值的滑动条，并将滑动条附加到指定的窗口上，它往往和一个回调函数配合起来使用。函数原型`int 	cv::createTrackbar (const String &trackbarname, const String &winname, int *value, int count, TrackbarCallback onChange=0, void *userdata=0)`

* trackbarname：滑动条的名称。
* winname：窗口的名称。
* value：表示滑动条当前位置的值。
* count：滑动条的最大位置的值。
* onChange：指向回调函数的指针，每次滑块位置改变时，这个函数都会进行回调，并且这个回调函数的原型必须是`void XXX(int,void*)`，第一个参数表示滑动条的位置，第二个参数是用户数据。如果回调时 NULL 指针，则表示满意回调函数的调用，尽第三个参数 value 有变化。
* userdata：用户传给回调函数的数据，用来处理滑动条事件，如果第三个参数 value 实参是全局变量的话，可以不用去管 userdata 参数。

### 获取当前轨迹条的位置

>getTrackbarPos()用于获取当前滑动条的位置，函数原型是`int 	cv::getTrackbarPos (const String &trackbarname, const String &winname)`。

*   trackbarname：滑动条名称。
*   winname：滑动条的父窗口的名称。

### 示例

```cpp
//
// opencv_tracker.cpp 创建滑动条
// 
// Created by 刘涤生 on 2017/3/24 16:06.
//

#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

using namespace cv;

#define WINDOW_NAME "Tracker"

const int MAX_ALPAH_VALUE = 100;
int slider;
double alpha;
double beta;

Mat image;
Mat dstImage;

void onTrack(int ,void*) {

    printf("%d\n",slider);
    alpha = (double)slider / MAX_ALPAH_VALUE;
    beta = 1.0 - alpha;

    addWeighted(image,alpha,image,alpha,0.0,dstImage);
    imshow(WINDOW_NAME,dstImage);
}

int main() {

    //读取图片

    image = imread("../lena.png");
    //显示图片
    imshow("Tracker",image);

    slider = 70;
    namedWindow(WINDOW_NAME,WINDOW_NORMAL);

    createTrackbar("Alpha 100",WINDOW_NAME,&slider,MAX_ALPAH_VALUE,onTrack);
    onTrack(slider,0);

    waitKey(0);
    return 0;
}
```

![]()

## 鼠标操作

## 设置鼠标监听

>setMouseCallback()函数用于为指定的窗口设置鼠标回调函数，原型如下`void 	cv::setMouseCallback (const String &winname, MouseCallback onMouse, void *userdata=0)`

*   winname：窗口的名称。
*   onMouse：鼠标事件发生时的回调函数指针，这个回调函数的原型为`void XXX(int event, int x, int y, int flags, void *userdata)`，其中 event 是 MouseEventTypes 鼠标事件常量，x 和 y 是鼠标指针在图像坐标系中的坐标值，flag 是 MouseEventTypes 的组合，userdata 是用户传到 setMouseCallback 函数中的 userdata 参数。
*   userdata：用户定义的传递到回调函数的参数。

### 示例

```cpp
//
// opencv_mouse_event.cpp 鼠标操作
// 
// Created by 刘涤生 on 2017/3/27 15:10.
//

#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

using namespace cv;

#define WINDOW_NAME "Mouse"

void onMouse(int event, int x, int y, int flags, void *userdata) {

    switch(event) {

        case EVENT_MOUSEMOVE:
            printf("EVENT_MOUSEMOVE\n");
            break;

        case EVENT_LBUTTONDOWN:
            printf("EVENT_LBUTTONDOWN\n");

            break;

        case EVENT_LBUTTONUP:
            printf("EVENT_LBUTTONUP\n");

            break;

        default:

            break;
    }

}

int main() {

    //读取图片
    Mat image = imread("../lena.png");
    //显示图片
    imshow(WINDOW_NAME,image);

    namedWindow(WINDOW_NAME,WINDOW_NORMAL);

    setMouseCallback(WINDOW_NAME, onMouse);

    waitKey(0);
}

/**
EVENT_MOUSEMOVE
EVENT_MOUSEMOVE
EVENT_LBUTTONUP
EVENT_MOUSEMOVE
 */
```
![](http://onke0yoit.bkt.clouddn.com/opencv_trackerbar.gif)

# 参考资料

* 《OpenCV3编程入门》

