OpenCV3③常用类
===

[TOC]

# Point 类

Point 类数据结构表示了二维坐标系下的点，即由其图像坐标 x 和 y 指定的 2D 点。

```cpp
Point point;
point.x = 10;
point.y = 8;

或者

Point point = Point(10,8);
```

# Scalar 类

`Scalar()` 表示具有四个元素的数组，在 OpenCV 中被大量用于传递像素值，如 RGB 颜色值，对于 Scalar 函数来说，如果用不到第四个参数，则不需要写出来；若只写三个参数，OpenCV 会认为我们就想表示三个参数。

```cpp
Scalar(a, b, c) //RGB颜色值：红色分量为c，绿色为b，蓝色分量为a
```

# Size 类

Size() 定义宽度和高度

```cpp
Size size = Size(5， 5)
size.width; //宽
size.height; //高
```

# Rect 类

Rect 类的成员变量有 `x、y、width、height`，分别为左上角点的坐标和矩形的宽和高。常用的成员函数有：`size()` 返回值为 `Size`；`area()` 返回矩形的面积；`contains(Point)` 判断点是否在矩形内；`inside(Rect)` 判断矩形是否在该矩形内；`tl()` 返回左上角点坐标；`br()` 返回右下角点坐标。如果想求两个矩形的交集和并集，可以用如下格式`Rect rect = rect1 & rect2; Rect rect = rect1 | rect2`；如果想让矩形进行平移操作和缩放操作，可以用如下形式`Rect rectShift = rect + point; Rect rectScale = rect + size`



