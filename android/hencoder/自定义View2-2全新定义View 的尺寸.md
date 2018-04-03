**自定义View2-2全新定义View 的尺寸**

[TOC]

[原文转自 HenCoder 自定义View2-2全新定义 View 的尺寸](https://zhuanlan.zhihu.com/p/32414322)

[视频](https://v.vzuu.com/video/929630654482743296)

# 全新定制尺寸和修改尺寸的最重要区别

需要在计算的同时，保证计算结果满足父 View 给出的的尺寸限制

# 父 View 的尺寸限制

1. 由来：开发者的要求（布局文件中 layout_ 打头的属性）经过父 View 处理计算后的更精确的要求；
2. 限制的分类：UNSPECIFIED：不限制、AT_MOST：限制上限、EXACTLY：限制固定值

# 全新定义自定义 View 尺寸的方式

1. 重新 onMeasure()，并计算出 View 的尺寸；
2. 使用 resolveSize() 来让子 View 的计算结果符合父 View 的限制（当然，如果你想用自己的方式来满足父 View 的限制也行。


