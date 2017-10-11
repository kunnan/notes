OpenCV3⑦图像算术与逻辑操作

[TOC]

# 算术操作

首先准备两张大小一致的图片，如下：

![](http://onke0yoit.bkt.clouddn.com/lena.png)

![](http://onke0yoit.bkt.clouddn.com/pure.png)

## 加

```cpp
Mat src1,src2,dst;
src1 = imread("../lena.png");
src2 = imread("../pure.png");

//算术加法
add(src1,src2,dst,Mat());
imshow("Add",dst);
```
![](http://onke0yoit.bkt.clouddn.com/math_add.png)

## 减

```cpp
subtract(src1,src2,dst,Mat());
imshow("subtract",dst);
```
![](http://onke0yoit.bkt.clouddn.com/math_subtract.png)

## 乘

```cpp
Mat mulImage = Mat(src1.size(),src1.type(),Scalar(2,2,2));
multiply(src1,mulImage,dst,1);
imshow("multiply",dst);
```
![](http://onke0yoit.bkt.clouddn.com/math_multiply.png)

## 除

```cpp
divide(src1,mulImage,dst,1);
imshow("divide",dst);
```
![](http://onke0yoit.bkt.clouddn.com/math_divide.png)

## 权重加法

```cpp
addWeighted(src1,1.0,src2,0.5,0,dst);
imshow("addWeight",dst);
```
![](http://onke0yoit.bkt.clouddn.com/math_add_weight.png)

# 逻辑操作

## 逻辑非

```cpp
bitwise_not(src1,dst,Mat());
imshow("not",dst);
```
![](http://onke0yoit.bkt.clouddn.com/math_not.png)

## 逻辑与

```cpp
bitwise_and(src1,src2,dst,Mat());
imshow("and",dst);
```
![](http://onke0yoit.bkt.clouddn.com/math_and.png)

## 逻辑或

```cpp
bitwise_or(src1,src2,dst,Mat());
imshow("or",dst);
imwrite("../math_or.png",dst);
```
![](http://onke0yoit.bkt.clouddn.com/math_or.png)

## 逻辑异或

```cpp
//纯白图片
Mat white = Mat(src1.size(),src1.type(),Scalar(255,255,255));
imshow("white",white);
//纯黑图片
Mat dark = Mat(src1.size(),src1.type(),Scalar(0,0,0));
imshow("dark",dark);

//逻辑异或
bitwise_xor(src1,white,dst,Mat());
imshow("xor",dst);
```
![](http://onke0yoit.bkt.clouddn.com/math_xor.png)


