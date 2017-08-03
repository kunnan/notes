**自定义View1-1 绘制基础**

[TOC]

[原文转载自扔物线：Android 开发进阶: 自定义 View 1-1 绘制基础](https://zhuanlan.zhihu.com/p/27787919)

# 自定义绘制知识的四个级别

## Canvas

Canvas 的 drawXXX() 方法及 Paint 最常见的使用。

Canvas.drawXXX() 是自定义绘制最基本的操作。掌握了这些方法，才知道怎么绘制内容，例如怎么画圆、画方、画图形和文字。组合绘制这些内容，再配合Paint上的一些常见方法来对绘制内容的颜色和风格进行简单的配置，就能实现大部分的绘制需求了。

![](https://pic4.zhimg.com/v2-764d6f89dadf10635f60e9f1444b4a3f_b.png)

![](https://pic3.zhimg.com/v2-4c904db3ec7c80b1946bfe8d81332bea_b.png)

![](https://pic4.zhimg.com/v2-325d2d5e4ca61b9acc4d9f2fa9daa7fb_b.png)

## Paint的完全攻略

Paint可以做的事不只是设置颜色，也不只是实心空心、线条粗细、有没有阴影，它可以做的风格设置非常多，例如：

>拐弯要什么形状？

![](https://pic2.zhimg.com/v2-c1ceb235034c191ee3038cbdd759abf5_b.png)

>开不开双线性过滤？

![](https://pic3.zhimg.com/v2-6b75511ed15d18875158311b74ac60ce_b.png)

>加不加特效？

![](https://pic3.zhimg.com/v2-d5ac530b894810ed6551cf68a1fc0656_b.png)

## 范围裁剪和几何变换

范围裁剪和几何变换一般用于辅助绘制

范围裁剪：

![](https://pic3.zhimg.com/v2-5b054225e346034d70cf6d4d6240ee56_b.png)

几何变换：

![](https://pic3.zhimg.com/v2-785dbc3841fcab4677a47444794ad39e_b.png)

## 控制绘制顺序

**控制绘制顺序解决的并不是「做不到」的问题，而是性能问题**。同样的一种效果，你不用绘制顺序的控制往往也能做到，但需要用多个 View 甚至是多层 View 才能拼凑出来，因此代价是 UI 的性能；而使用绘制顺序的话，一个View就全部搞定了。

自定义绘制的知识，大概就分为上面这四个级别。在你把这四个级别依次掌握了之后，你就是一个自定义绘制的高手了。下面开始第一篇： Canvas.drawXXX() 系列方法及 Paint 最基本的使用。


# 一切的开始：OnDraw()

自定义绘制的过程非常简单：提前创建好Paint对象，重写onDraw()，把绘制代码写在onDraw()里面，这就是自定义绘制最基本的实现。

```Java
Paint paint = new Paint();

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    // 绘制一个圆
    canvas.drawCircle(300, 300, 200, paint);
}
```

# Canvas.drawXXX() 和 Paint 基础

drawXXX() 系列方法和 Paint 的基础掌握了，就能够应付简单的绘制需求。它们主要包括：

* Canvas 类下的所有 draw- 打头的方法，例如 `drawCircle() drawBitmap()`。
* Paint 类的几个最常用的方法。具体是：
    * Paint.setStyle(Style style) 设置绘制模式
    * Paint.setColor(int color) 设置颜色
    * Paint.setStrokeWidth(float width) 设置线条宽度
    * Paint.setTextSize(float textSize) 设置文字大小
    * Paint.setAntiAlias(boolean aa) 设置抗锯齿开关

[Canvas官方文档](https://developer.android.google.cn/reference/android/graphics/Canvas.html)
[Paint官方文档](https://developer.android.google.cn/reference/android/graphics/Paint.html)

## drawColor

>Canvas.drawColor(@ColorInt int color) 颜色填充，这是最基本的 drawXXX() 方法：在整个绘制区域统一涂上指定的颜色。

例如 drawColor(Color.BLACK) 会把整个区域染成纯黑色，覆盖掉原有内容； `drawColor(Color.parse("#88880000")` 会在原有的绘制效果上加一层半透明的红色遮罩

`drawColor(Color.BLACK);  // 纯黑`

![](https://pic2.zhimg.com/v2-bd7b32f98a2ca396362e32eef607da91_b.png)

`drawColor(Color.parse("#88880000"); // 半透明红色`

![](https://pic1.zhimg.com/v2-7fb1a6ddde036c532a25b45a830e2468_b.png)

类似的方法还有 `drawRGB(int r, int g, int b)` 和 `drawARGB(int a, int r, int g, int b)` ，它们和 `drawColor(color)` 只是使用方式不同，作用都是一样的。

```Java
canvas.drawRGB(100, 200, 100);
canvas.drawARGB(100, 100, 200, 100);
```
**这类颜色填充方法一般用于在绘制之前设置底色，或者在绘制之后为界面设置半透明蒙版。**

## drawCircle

>drawCircle(float centerX, float centerY, float radius, Paint paint) 画圆

* centerX centerY：是圆心的坐标，单位是像素
* radius：是圆的半径，单位是像素
* paint：画笔，它提供基本信息之外的所有风格信息，例如颜色、线条粗细、阴影等。

>canvas.drawCircle(300, 300, 200, paint);

![](https://pic3.zhimg.com/v2-57bfe51b2fdf4e5c9b4d6d57987d86ca_b.png)

在Android坐标系中的位置：

![](https://pic4.zhimg.com/v2-7379659fce2874d477f79fc81490d42b_b.png)

圆心坐标和半径，这些都是圆的基本信息，也是它的独有信息。什么叫独有信息？就是只有它有，别人没有的信息。你画圆有圆心坐标和半径，画方有吗？画椭圆有吗？这就叫独有信息。独有信息都是直接作为参数写进 drawXXX() 方法里的（比如 drawCircle(centerX, centerY, radius, paint) 的前三个参数）。

而除此之外，其他的都是公有信息。比如图形的颜色、空心实心这些，你不管是画圆还是画方都有可能用到的，这些信息则是统一放在 paint 参数里的。

### Paint.setColor(int color)

例如，你要画一个红色的圆，并不是写成 canvas.drawCircle(300, 300, 200, RED, paint) 这样，而是像下面这样：

```Java
paint.setColor(Color.RED); // 设置为红色
canvas.drawCircle(300, 300, 200, paint);
```
![](https://pic4.zhimg.com/v2-299df7cf4a404f2cee837a76778aec67_b.png)

Paint.setColor(int color) 是 Paint 最常用的方法之一，用来设置绘制内容的颜色。你不止可以用它画红色的圆，也可以用它来画红色的矩形、红色的五角星、红色的文字。

### Paint.setStyle(Paint.Style style)

而如果你想画的不是实心圆，而是空心圆（或者叫环形），也可以使用 paint.setStyle(Paint.Style.STROKE) 来把绘制模式改为画线模式。

```Java
paint.setStyle(Paint.Style.STROKE); // Style 修改为画线模式
canvas.drawCircle(300, 300, 200, paint);
```
`setStyle(Style style)` 这个方法设置的是绘制的 Style 。Style 具体来说有三种： FILL, STROKE 和 FILL_AND_STROKE 。FILL 是填充模式，STROKE 是画线模式（即勾边模式），FILL_AND_STROKE 是两种模式一并使用：既画线又填充。它的默认值是 FILL，填充模式。

### Paint.setStrokeWidth(float width)

在 STROKE 和 FILL_AND_STROKE 下，还可以使用 paint.setStrokeWidth(float width) 来设置线条的宽度：

```Java
paint.setStyle(Paint.Style.STROKE);
paint.setStrokeWidth(20); // 线条宽度为 20 像素
canvas.drawCircle(300, 300, 200, paint);
```
![](https://pic1.zhimg.com/v2-33317c32913d3b18a2bd936aa0f5b5c0_b.png)

### Paint.setAntiAlias(boolean aa)

在绘制的时候，往往需要开启抗锯齿来让图形和文字的边缘更加平滑。开启抗锯齿很简单，只要在 new Paint() 的时候加上一个 ANTI_ALIAS_FLAG 参数就行：

```Java
Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
```
另外，你也可以使用 Paint.setAntiAlias(boolean aa) 来动态开关抗锯齿。抗锯齿的效果如下：

![](https://pic1.zhimg.com/v2-7dcde98a1ed45327e10244279faaca08_b.png)

## drawRect

>drawRect(float left, float top, float right, float bottom, Paint paint) 画矩形
>drawRect(RectF rect, Paint paint) 通过RectF对象画矩形
>drawRect(Rect rect, Paint paint) 通过Rect对象画矩形

* left, top, right, bottom 是矩形四条边的坐标

```Java
paint.setStyle(Style.FILL);
canvas.drawRect(100, 100, 500, 500, paint);
  
paint.setStyle(Style.STROKE);
canvas.drawRect(700, 100, 1100, 500, paint);
```
![](https://pic3.zhimg.com/v2-716c6e178ae6cae876495e13e76394ba_b.png)

## drawPoint

>drawPoint(float x, float y, Paint paint) 画点

x 和 y 是点的坐标。点的大小可以通过 paint.setStrokeWidth(width) 来设置；点的形状可以通过 paint.setStrokeCap(cap) 来设置：ROUND 画出来是圆形的点，SQUARE 或 BUTT 画出来是方形的点。[Cap官方文档](https://developer.android.google.cn/reference/android/graphics/Paint.Cap.html)

>注：Paint.setStrokeCap(cap) 可以设置点的形状，但这个方法并不是专门用来设置点的形状的，而是一个设置线条端点形状的方法。端点有圆头 (ROUND)、平头 (BUTT) 和方头 (SQUARE) 三种

```Java
paint.setStrokeWidth(20);
paint.setStrokeCap(Paint.Cap.ROUND);
canvas.drawPoint(50, 50, paint);
```
![](https://pic3.zhimg.com/v2-b71189c7d0e3f47dccc5c40ae0ec21da_b.png)

```Java
paint.setStrokeWidth(20);
paint.setStrokeCap(Paint.Cap.SQUARE;
canvas.drawPoint(50, 50, paint);
```
![](https://pic1.zhimg.com/v2-2dba848a12123f8769ac682ad018038c_b.png)

## drawPoints

>drawPoints(float[] pts, int offset, int count, Paint paint)
>drawPoints(float[] pts, Paint paint) 画点（批量）

* pts 这个数组是点的坐标，每两个成一对
* offset 表示跳过数组的前几个数再开始记坐标
* count 表示坐标的个数

```Java
float[] points = {0, 0, 50, 50, 50, 100, 100, 50, 100, 100, 150, 50, 150, 100};
// 绘制四个点：(50, 50) (50, 100) (100, 50) (100, 100)
canvas.drawPoints(points, 2 /* 跳过两个数，即前两个 0 */,
          8 /* 一共绘制 8 个数（4 个点）*/, paint);
```
![](https://pic1.zhimg.com/v2-d9037a7ffc3ff2ace2c8e60861b1be7c_b.png)

## drawOval

>drawOval(RectF rect, Paint paint)
>drawOval(float left, float top, float right, float bottom, Paint paint) 画椭圆
>只能绘制横着的或者竖着的椭圆。left, top, right, bottom 是这个椭圆的左、上、右、下四个边界点的坐标。

```Java
paint.setStyle(Style.FILL);
canvas.drawOval(50, 50, 350, 200, paint);

paint.setStyle(Style.STROKE);
canvas.drawOval(400, 50, 700, 200, paint);
```
![](https://pic2.zhimg.com/v2-4d7afb1aa2ba028d36243f2dc6a8430d_b.png)

## drawLine

>drawLine(float startX, float startY, float stopX, float stopY, Paint paint) 画线

* startX, startY, stopX, stopY 分别是线的起点和终点坐标。

>drawLines(float[] pts, Paint paint)  批量画线
>drawLines(float[] pts, int offset, int count, Paint paint)

```Java
canvas.drawLine(200, 200, 800, 500, paint);
```
![](https://pic1.zhimg.com/v2-d47cb08fdf64f6cf9b6631bb43c27534_b.png)

>由于直线不是封闭图形，所以 setStyle(style) 对直线没有影响。

```Java
float[] points = {20, 20, 120, 20, 70, 20, 70, 120, 20, 120, 120, 120, 150, 20, 250, 20, 150, 20, 150, 120, 250, 20, 250, 120, 150, 120, 250, 120};
canvas.drawLines(points, paint);
```
![](https://pic4.zhimg.com/v2-2fc1c7f19de673e332db5180304eceef_b.png)

## drawRoundRect

>drawRoundRect(RectF rect, float rx, float ry, Paint paint)
>drawRoundRect(float left, float top, float right, float bottom, float rx, float ry, Paint paint) 画圆角矩形

* left, top, right, bottom 是四条边的坐标
* rx 和 ry 是圆角的横向半径和纵向半径

```Java
canvas.drawRoundRect(100, 100, 500, 300, 50, 50, paint);
```
![](https://pic2.zhimg.com/v2-435b255556293566c99a8a54c12383a5_b.png)

## drawArc

>drawArc(float left, float top, float right, float bottom, float startAngle, float sweepAngle, boolean useCenter, Paint paint) 绘制弧形或扇形

* left, top, right, bottom 描述的是这个弧形所在的椭圆;
* startAngle 是弧形的起始角度（x 轴的正向，即正右的方向，是 0 度的位置；顺时针为正角度，逆时针为负角度）;
* sweepAngle 是弧形划过的角度;
* useCenter 表示是否连接到圆心，如果不连接到圆心，就是弧形，如果连接到圆心，就是扇形。

```Java
paint.setStyle(Paint.Style.FILL); // 填充模式
canvas.drawArc(200, 100, 800, 500, -110, 100, true, paint); // 绘制扇形
canvas.drawArc(200, 100, 800, 500, 20, 140, false, paint); // 绘制弧形
paint.setStyle(Paint.Style.STROKE); // 画线模式
canvas.drawArc(200, 100, 800, 500, 180, 60, false, paint); // 绘制不封口的弧形
```
![](https://pic3.zhimg.com/v2-96faf09a2471c5dbcb76e9a118ffd292_b.png)

到此为止，以上就是 Canvas 所有的简单图形的绘制。除了简单图形的绘制， Canvas 还可以使用 drawPath(Path path) 来绘制自定义图形。

## drawBitmap

>drawBitmap(Bitmap bitmap, float left, float top, Paint paint) 画 Bitmap
>drawBitmap(Bitmap bitmap, Rect src, RectF dst, Paint paint) 
>drawBitmap(Bitmap bitmap, Rect src, Rect dst, Paint paint)
>drawBitmap(Bitmap bitmap, Matrix matrix, Paint paint)

绘制 Bitmap 对象，也就是把这个 Bitmap 中的像素内容贴过来。其中 left 和 top 是要把 bitmap绘制到的位置坐标。

```Java
drawBitmap(bitmap, 200, 100, paint);
```
![](https://pic1.zhimg.com/v2-2216a99e4aa817855def6265ee572bc0_b.png)

## drawText

>drawText(String text, float x, float y, Paint paint) 绘制文字

* 参数 text 是用来绘制的字符串;
* x 和 y 是绘制的起点坐标;

```Java
canvas.drawText(text, 200, 100, paint);
```
![](https://pic3.zhimg.com/v2-a18a85ff3555029c5b56706d4ff71e02_b.png)

### Paint.setTextSize(float textSize)

通过 Paint.setTextSize(textSize)，可以设置文字的大小。

```Java
paint.setTextSize(18);
canvas.drawText(text, 100, 25, paint);
paint.setTextSize(36);
canvas.drawText(text, 100, 70, paint);
paint.setTextSize(60);
canvas.drawText(text, 100, 145, paint);
paint.setTextSize(84);
canvas.drawText(text, 100, 240, paint);
```
![](https://pic2.zhimg.com/v2-10c7a042deb59e69c9be3fe044b83c0d_b.png)

## drawPath

>drawPath(Path path, Paint paint) 画自定义图形 [Path官方文档](https://developer.android.google.cn/reference/android/graphics/Path.html)

前面的这些方法，都是绘制某个给定的图形，而 drawPath() 可以绘制自定义图形。当你要绘制的图形比较特殊，使用前面的那些方法做不到的时候，就可以使用 drawPath() 来绘制。

![](https://pic3.zhimg.com/v2-555a511cce80232edfb792906d02b4e2_b.png)

drawPath(path) 这个方法是通过描述路径的方式来绘制图形的，它的 path 参数就是用来描述图形路径的对象。path 的类型是 Path ，使用方法大概像下面这样：

```Java
public class PathView extends View {

    Paint paint = new Paint();
    Path path = new Path(); // 初始化 Path 对象
    
    ......
    
    {
      // 使用 path 对图形进行描述（这段描述代码不必看懂）
      path.addArc(200, 200, 400, 400, -225, 225);
      path.arcTo(400, 200, 600, 400, -180, 225, false);
      path.lineTo(400, 542);
    }

    @Override
    protected void onDraw(Canvas canvas) {
      super.onDraw(canvas);
      
      canvas.drawPath(path, paint); // 绘制出 path 描述的图形（心形），大功告成
    }
}
```
![](https://pic2.zhimg.com/v2-ea827ac49efdebece7167dcdc67b2445_b.png)

Path 可以描述直线、二次曲线、三次曲线、圆、椭圆、弧形、矩形、圆角矩形。把这些图形结合起来，就可以描述出很多复杂的图形。下面我就说一下具体的怎么把这些图形描述出来。

Path 有两类方法，一类是直接描述路径的，另一类是辅助的设置或计算。

### Path 方法第一类：直接描述路径

#### addXxx() ——添加子图形

* addCircle(float x, float y, float radius, Direction dir) 添加圆

>x, y, radius 这三个参数是圆的基本信息，最后一个参数 dir 是画圆的路径的方向。
>路径方向有两种：顺时针 (CW clockwise) 和逆时针 (CCW counter-clockwise) 。对于普通情况，这个参数填 CW 还是填 CCW 没有影响。它只是在需要填充图形 (Paint.Style 为 FILL 或 FILL_AND_STROKE) ，并且图形出现自相交时，用于判断填充范围的。

* addOval(float left, float top, float right, float bottom, Direction dir) / addOval(RectF oval, Direction dir) 添加椭圆

* addRect(float left, float top, float right, float bottom, Direction dir) / addRect(RectF rect, Direction dir) 添加矩形

* addRoundRect(RectF rect, float rx, float ry, Direction dir) / addRoundRect(float left, float top, float right, float bottom, float rx, float ry, Direction dir) / addRoundRect(RectF rect, float[] radii, Direction dir) / addRoundRect(float left, float top, float right, float bottom, float[] radii, Direction dir) 添加圆角矩形

* addArc(float left, float top, float right, float bottom, float startAngle, float sweepAngle) / addArc(RectF oval, float startAngle, float sweepAngle) 添加弧形

* addPath(Path path) 添加另一个 Path

#### xxxTo() ——画线（直线或曲线）

* lineTo(float x, float y) / rLineTo(float x, float y) 画直线

从**当前位置**向目标位置画一条直线， x 和 y 是目标位置的坐标。这两个方法的区别是，lineTo(x, y)的参数是**绝对坐标**，而 rLineTo(x, y) 的参数是相对当前位置的**相对坐标** （前缀 r 指的就是 relatively 「相对地」)。

>**当前位置**：所谓当前位置，即最后一次调用画 Path 的方法的终点位置。初始值为原点 (0, 0)。

```Java
paint.setStyle(Style.STROKE);
path.lineTo(100, 100); // 由当前位置 (0, 0) 向 (100, 100) 画一条直线
path.rLineTo(100, 0); // 由当前位置 (100, 100) 向正右方 100 像素的位置画一条直线
```
![](https://pic4.zhimg.com/v2-7657eee8380b034d95d4333591d4698f_b.png)

* quadTo(float x1, float y1, float x2, float y2) / rQuadTo(float dx1, float dy1, float dx2, float dy2) 画二次贝塞尔曲线

>这条二次贝塞尔曲线的起点就是当前位置，而参数中的 x1, y1 和 x2, y2 则分别是控制点和终点的坐标。和 rLineTo(x, y) 同理，rQuadTo(dx1, dy1, dx2, dy2) 的参数也是相对坐标

* cubicTo(float x1, float y1, float x2, float y2, float x3, float y3) / rCubicTo(float x1, float y1, float x2, float y2, float x3, float y3) 画三阶贝塞尔曲线

* moveTo(float x, float y) / rMoveTo(float x, float y) 移动到目标位置

>不论是直线还是贝塞尔曲线，都是以当前位置作为起点，而不能指定起点。但你可以通过 moveTo(x, y)或 rMoveTo() 来改变当前位置，从而间接地设置这些方法的起点。

* arcTo(RectF oval, float startAngle, float sweepAngle, boolean forceMoveTo) / arcTo(float left, float top, float right, float bottom, float startAngle, float sweepAngle, boolean forceMoveTo) / arcTo(RectF oval, float startAngle, float sweepAngle) 画弧形

>forceMoveTo 参数的意思是，绘制是要「抬一下笔移动过去」，还是「直接拖着笔过去」，区别在于是否留下移动的痕迹。

```Java
paint.setStyle(Style.STROKE);
path.lineTo(100, 100);
path.arcTo(100, 100, 300, 300, -90, 90, true); // 强制移动到弧形起点（无痕迹）
```
![](https://pic4.zhimg.com/v2-8faa6acdbd58d50789f43b03badd0b2b_b.png)

```Java
paint.setStyle(Style.STROKE);
path.lineTo(100, 100);
path.arcTo(100, 100, 300, 300, -90, 90, false); // 直接连线连到弧形起点（有痕迹
```
![](https://pic3.zhimg.com/v2-0e058857d727d56f9a23ec5f5daa4fa6_b.png)

#### close() 封闭当前子图形

它的作用是把当前的子图形封闭，即由当前位置向当前子图形的起点绘制一条直线。close() 和 lineTo(起点坐标) 是完全等价的。

```Java
paint.setStyle(Style.STROKE);
path.moveTo(100, 100);
path.lineTo(200, 100);
path.lineTo(150, 150);
// 子图形未封闭
```
![](https://pic2.zhimg.com/v2-46f06258fd6fba811aadb5f5a80add8d_b.png)

```Java
paint.setStyle(Style.STROKE);
path.moveTo(100, 100);
path.lineTo(200, 100);
path.lineTo(150, 150);
path.close(); // 使用 close() 封闭子图形。等价于 path.lineTo(100, 100)
```
![](https://pic4.zhimg.com/v2-0d4b07fa8cc9c32b7a58596a6ce2853f_b.png)

### Path 方法第二类：辅助的设置或计算

#### Path.op(Path path, Path.Op op)

op(Path path, Path.Op op) 方法，用于将两个Path路径进行组合之后的效果设置，靠op方法可以快速组合生成一些复杂的图形效果，例如月牙形 

Path.Op有如下几种参数  
 
* Path.Op.DIFFERENCE：减去Path2后Path1剩下的部分   
* Path.Op.INTERSECT：保留Path1与Path2共同的部分   
* Path.Op.REVERSE_DIFFERENCE：减去Path1后Path2剩下的部分   
* Path.Op.UNION：保留全部Path1和Path2    
* Path.Op.XOR：包含Path1与Path2但不包括两者相交的部分 

```Java
@Override
protected void onDraw(Canvas canvas) {
   //设置绘制风格
   setViewPaint();
   //设置填充风格，方便观察效果
   paint.setStyle(Paint.Style.FILL);
   //构建path
   Path path=new Path();
   path.addCircle(150, 150, 100, Path.Direction.CW);
   Path path2 = new Path();
   path2.addCircle(300, 150, 100, Path.Direction.CW);
   path.op(path2,Path.Op.UNION);
   //Path.Op.UNION
   canvas.drawPath(path,paint);

   //清除路径
   path.reset();
   path2.reset();
   path.addCircle(150, 400, 100, Path.Direction.CW);
   path2.addCircle(300, 400, 100, Path.Direction.CW);
   path.op(path2,Path.Op.REVERSE_DIFFERENCE);
   //Path.Op.REVERSE_DIFFERENCE
   canvas.drawPath(path,paint);

   //清除路径
   path.reset();
   path2.reset();
   path.addCircle(150, 650, 100, Path.Direction.CW);
   path2.addCircle(300, 650, 100, Path.Direction.CW);
   path.op(path2,Path.Op.INTERSECT);
   //Path.Op.INTERSECT
   canvas.drawPath(path,paint);

   //清除路径
   path.reset();
   path2.reset();
   path.addCircle(150, 900, 100, Path.Direction.CW);
   path2.addCircle(300, 900, 100, Path.Direction.CW);
   path.op(path2,Path.Op.DIFFERENCE);
   //Path.Op.DIFFERENCE
   canvas.drawPath(path,paint);

   //清除路径
   path.reset();
   path2.reset();
   path.addCircle(150, 1150, 100, Path.Direction.CW);
   path2.addCircle(300, 1150, 100, Path.Direction.CW);
   path.op(path2,Path.Op.XOR);
   //Path.Op.XOR
   canvas.drawPath(path,paint);

}
```

![](http://7xs7a3.com1.z0.glb.clouddn.com/path-path%E4%B8%ADop%E6%96%B9%E6%B3%95%E7%9A%84%E7%94%A8%E6%B3%95.png)

#### Path.setFillType(Path.FillType ft)

Path.setFillType(Path.FillType ft) 设置填充方式。

前面在说 dir 参数的时候提到， Path.setFillType(fillType) 是用来设置图形自相交时的填充算法的：

![](https://pic1.zhimg.com/v2-83ebbc6bd6a13fcc7217abeafcc6a5b4_b.png)

方法中填入不同的 FillType 值，就会有不同的填充效果。FillType 的取值有四个：

* EVEN_ODD EVEN_ODD 
* WINDING （默认值）全填充
* INVERSE_EVEN_ODD
* INVERSE_WINDING

![](https://pic1.zhimg.com/v2-bc3da8bcc5833989e9e70ec121346f04_b.png)

##### EVEN_ODD的原理

即 even-odd rule （奇偶原则）：对于平面中的任意一点，向任意方向射出一条射线，这条射线和图形相交的次数（相交才算，相切不算哦）如果是奇数，则这个点被认为在图形内部，是要被涂色的区域；如果是偶数，则这个点被认为在图形外部，是不被涂色的区域。还以左右相交的双圆为例：

![](https://pic4.zhimg.com/v2-932fe07e7d5a514a699cf0687d4f9147_r.png)

>射线的方向无所谓，同一个点射向任何方向的射线，结果都是一样的，不信你可以试试。

从上图可以看出，射线每穿过图形中的一条线，内外状态就发生一次切换，这就是为什么 EVEN_ODD 是一个「交叉填充」的模式。

##### WINDING的原理

即 non-zero winding rule （非零环绕数原则）：首先，它需要你图形中的所有线条都是有绘制方向的：

![](https://pic2.zhimg.com/v2-9641f6ef40ed0f60510ceacf3566a6d1_b.png)

然后，同样是从平面中的点向任意方向射出一条射线，但计算规则不一样：以 0 为初始值，对于射线和图形的所有交点，遇到每个顺时针的交点（图形从射线的左边向右穿过）把结果加 1，遇到每个逆时针的交点（图形从射线的右边向左穿过）把结果减 1，最终把所有的交点都算上，得到的结果如果不是 0，则认为这个点在图形内部，是要被涂色的区域；如果是 0，则认为这个点在图形外部，是不被涂色的区域。

![](https://pic3.zhimg.com/v2-692d88842aa31152567833caa062e072_b.png)

>和 EVEN_ODD 相同，射线的方向并不影响结果。

所以，前面的那个「简单粗暴」的总结，对于 WINDING 来说并不完全正确：如果你所有的图形都用相同的方向来绘制，那么 WINDING 确实是一个「全填充」的规则；但如果使用不同的方向来绘制图形，结果就不一样了。

>图形的方向：对于添加子图形类方法（如 Path.addCircle() Path.addRect()）的方向，由方法的 dir 参数来控制，这个在前面已经讲过了；而对于画线类的方法（如 Path.lineTo()Path.arcTo()）就更简单了，线的方向就是图形的方向。

所以，完整版的 EVEN_ODD 和 WINDING 的效果应该是这样的：

![](https://pic1.zhimg.com/v2-32a19f7fcac54fe9d185c7ac0b3bf360_b.png)

而 INVERSE_EVEN_ODD 和 INVERSE_WINDING ，只是把这两种效果进行反转而已。


