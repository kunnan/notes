**自定义View2-3定制 Layout 的内部布局**

[TOC]

[原文转自 HenCoder Android 2-3：定制 Layout 的内部布局](https://zhuanlan.zhihu.com/p/32957168)

这期是 HenCoder 布局部分的最后一期：重写 onMeasure() 和 onLayout() 来定制 Layout 的内部布局。

[视频地址 https://www.bilibili.com/video/av18330166/](https://www.bilibili.com/video/av18330166/)

# 定制 Layout 内部布局的方式

1. 重写 onMeasure() 来计算内部布局
2. 重写 onLayout() 来摆放子 View

# 重写 onMeasure() 的三个步骤

1. 调用每个子 View 的 measure() 来计算子 View 的尺寸
2. 计算子 View 的位置并保存子 View 的位置和尺寸
3. 计算自己的尺寸并用 setMeasuredDimension() 保存

# 计算子 View 尺寸的关键

计算子 View 的尺寸，关键在于 measure() 方法的两个参数——也就是子 View 的两个 MeasureSpec 的计算。

# 子 View 的 MeasureSpec 的计算方式

* 结合开发者的要求（xml 中 layout_ 打头的属性）和自己的可用空间（自己的尺寸上限 - 已用尺寸）
* 尺寸上限根据自己的 MeasureSpec 中的 mode 而定
    * EXACTLY / AT_MOST：尺寸上限为 MeasureSpec 中的 size
    * UNSPECIFIED：尺寸无上限

# 重写 onLayout() 的方式

在 onLayout() 里调用每个子 View 的 layout() ，让它们保存自己的位置和尺寸。

