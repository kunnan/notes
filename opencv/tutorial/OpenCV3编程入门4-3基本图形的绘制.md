OpenCV3④基本图形的绘制
===

[TOC]

# 直线

>line 函数用于绘制直线，函数原型：

```cpp
void cv::line (InputOutputArray img, Point pt1, Point pt2, const Scalar &color, int thickness=1, int lineType=LINE_8, int shift=0)
```

# 矩形

>rectangle 函数用于绘制矩形

```cpp
void 	cv::rectangle (InputOutputArray img, Point pt1, Point pt2, const Scalar &color, int thickness=1, int lineType=LINE_8, int shift=0)

void 	cv::rectangle (Mat &img, Rect rec, const Scalar &color, int thickness=1, int lineType=LINE_8, int shift=0)
```

# 椭圆

>ellipse 函数用于绘制椭圆

```cpp
cv::ellipse (InputOutputArray img, Point center, Size axes, double angle, double startAngle, double endAngle, const Scalar &color, int thickness=1, int lineType=LINE_8, int shift=0)

cv::ellipse (InputOutputArray img, const RotatedRect &box, const Scalar &color, int thickness=1, int lineType=LINE_8)
```
# 圆

>circle 函数用于绘制圆

```cpp
cv::circle (InputOutputArray img, Point center, int radius, const Scalar &color, int thickness=1, int lineType=LINE_8, int shift=0)
```

# 多边形

>fillPoly 函数用于绘制多边形

```cpp
void 	cv::fillPoly (Mat &img, const Point **pts, const int *npts, int ncontours, const Scalar &color, int lineType=LINE_8, int shift=0, Point offset=Point())
 
void 	cv::fillPoly (InputOutputArray img, InputArrayOfArrays pts, const Scalar &color, int lineType=LINE_8, int shift=0, Point offset=Point())
```

# 绘制文字

```cpp
cv::putText (InputOutputArray img, const String &text, Point org, int fontFace, double fontScale, Scalar color, int thickness=1, int lineType=LINE_8, bool bottomLeftOrigin=false)
```

# 示例

```cpp
//
// draw.cpp
// 
// Created by 刘涤生 on 2017/3/28 14:23.
//

#include "opencv2/core.hpp"
#include "opencv2/imgproc.hpp"
#include "opencv2/highgui.hpp"

using namespace cv;


static Scalar randomColor(RNG& rng)
{
    int icolor = (unsigned)rng;
    return Scalar(icolor&255, (icolor>>8)&255, (icolor>>16)&255);
}

int main()
{
    char wndname[] = "Drawing";
    const int NUMBER = 100;
    const int DELAY = 5;
    int lineType = LINE_AA; // change it to LINE_8 to see non-antialiased graphics
    int i, width = 500, height = 500;
    int x1 = -width/2, x2 = width*3/2, y1 = -height/2, y2 = height*3/2;
    RNG rng(0xFFFFFFFF);

    Mat image = Mat::zeros(height, width, CV_8UC3);
    imshow(wndname, image);
    waitKey(DELAY);

    for (i = 0; i < NUMBER; i++)
    {
        Point pt1, pt2;
        pt1.x = rng.uniform(x1, x2);
        pt1.y = rng.uniform(y1, y2);
        pt2.x = rng.uniform(x1, x2);
        pt2.y = rng.uniform(y1, y2);

        line( image, pt1, pt2, randomColor(rng), rng.uniform(1,10), lineType );

        imshow(wndname, image);
    }
    imwrite("../draw_line.jpg",image);

    for (i = 0; i < NUMBER; i++)
    {
        Point pt1, pt2;
        pt1.x = rng.uniform(x1, x2);
        pt1.y = rng.uniform(y1, y2);
        pt2.x = rng.uniform(x1, x2);
        pt2.y = rng.uniform(y1, y2);
        int thickness = rng.uniform(-3, 10);

        rectangle( image, pt1, pt2, randomColor(rng), MAX(thickness, -1), lineType );

        imshow(wndname, image);
    }
    imwrite("../draw_rectangle.jpg",image);

    for (i = 0; i < NUMBER; i++)
    {
        Point center;
        center.x = rng.uniform(x1, x2);
        center.y = rng.uniform(y1, y2);
        Size axes;
        axes.width = rng.uniform(0, 200);
        axes.height = rng.uniform(0, 200);
        double angle = rng.uniform(0, 180);

        ellipse( image, center, axes, angle, angle - 100, angle + 200,
                 randomColor(rng), rng.uniform(-1,9), lineType );

        imshow(wndname, image);
    }
    imwrite("../draw_ellipse.jpg",image);

    for (i = 0; i< NUMBER; i++)
    {
        Point pt[2][3];
        pt[0][0].x = rng.uniform(x1, x2);
        pt[0][0].y = rng.uniform(y1, y2);
        pt[0][1].x = rng.uniform(x1, x2);
        pt[0][1].y = rng.uniform(y1, y2);
        pt[0][2].x = rng.uniform(x1, x2);
        pt[0][2].y = rng.uniform(y1, y2);
        pt[1][0].x = rng.uniform(x1, x2);
        pt[1][0].y = rng.uniform(y1, y2);
        pt[1][1].x = rng.uniform(x1, x2);
        pt[1][1].y = rng.uniform(y1, y2);
        pt[1][2].x = rng.uniform(x1, x2);
        pt[1][2].y = rng.uniform(y1, y2);
        const Point* ppt[2] = {pt[0], pt[1]};
        int npt[] = {3, 3};

        polylines(image, ppt, npt, 2, true, randomColor(rng), rng.uniform(1,10), lineType);

        imshow(wndname, image);
    }
    imwrite("../draw_polylines.jpg",image);


    for (i = 0; i< NUMBER; i++)
    {
        Point pt[2][3];
        pt[0][0].x = rng.uniform(x1, x2);
        pt[0][0].y = rng.uniform(y1, y2);
        pt[0][1].x = rng.uniform(x1, x2);
        pt[0][1].y = rng.uniform(y1, y2);
        pt[0][2].x = rng.uniform(x1, x2);
        pt[0][2].y = rng.uniform(y1, y2);
        pt[1][0].x = rng.uniform(x1, x2);
        pt[1][0].y = rng.uniform(y1, y2);
        pt[1][1].x = rng.uniform(x1, x2);
        pt[1][1].y = rng.uniform(y1, y2);
        pt[1][2].x = rng.uniform(x1, x2);
        pt[1][2].y = rng.uniform(y1, y2);
        const Point* ppt[2] = {pt[0], pt[1]};
        int npt[] = {3, 3};

        fillPoly(image, ppt, npt, 2, randomColor(rng), lineType);

        imshow(wndname, image);
    }
    imwrite("../draw_fillpoly.jpg",image);

    for (i = 0; i < NUMBER; i++)
    {
        Point center;
        center.x = rng.uniform(x1, x2);
        center.y = rng.uniform(y1, y2);

        circle(image, center, rng.uniform(0, 300), randomColor(rng),
               rng.uniform(-1, 9), lineType);

        imshow(wndname, image);
    }
    imwrite("../draw_circle.jpg",image);


    for (i = 1; i < NUMBER; i++)
    {
        Point org;
        org.x = rng.uniform(x1, x2);
        org.y = rng.uniform(y1, y2);

        putText(image, "Testing text rendering", org, rng.uniform(0,8),
                rng.uniform(0,100)*0.05+0.1, randomColor(rng), rng.uniform(1, 10), lineType);

        imshow(wndname, image);
    }

    Size textsize = getTextSize("OpenCV forever!", FONT_HERSHEY_COMPLEX, 1, 5, 0);
    Point org((width - textsize.width)/2, (height - textsize.height)/2);

    Mat image2;
    for( i = 0; i < 255; i += 2 )
    {
        image2 = image - Scalar::all(i);
        putText(image2, "OpenCV forever!", org, FONT_HERSHEY_COMPLEX, 1,
                Scalar(i, i, 255), 5, lineType);

        imshow(wndname, image2);
    }
    imwrite("../draw_text.jpg",image2);
    waitKey();
    return 0;
}
```

* 线条  

![](http://onke0yoit.bkt.clouddn.com/draw_line.jpg)

* 矩形  

![](http://onke0yoit.bkt.clouddn.com/draw_rectangle.jpg)

* 椭圆   

![](http://onke0yoit.bkt.clouddn.com/draw_ellipse.jpg)
 
*  拟合曲线  

![](http://onke0yoit.bkt.clouddn.com/draw_polylines.jpg)

* 多边形  

![](http://onke0yoit.bkt.clouddn.com/draw_fillpoly.jpg)

* 文字  

![](http://onke0yoit.bkt.clouddn.com/draw_text.jpg)

* 圆  

![](http://onke0yoit.bkt.clouddn.com/draw_circle.jpg)

