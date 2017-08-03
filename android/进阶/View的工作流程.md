**View的工作流程**

[TOC]

View的工作流程主要指measure、layout、draw这三大流程，即测量、布局和绘制，其中measure确定View的测量宽/高，layout确定View的最终宽/高和四个顶点的位置（即View在父容器中的放置位置），draw则将View绘制到屏幕上。

# measure过程

measure过程要分情况来看，如果只是一个原始的View，那么通过measure方法就完成了其测量的过程，如果是一个ViewGroup，除了完成自己的测量过程外，还会遍历去调用所有子元素的measure方法，各个子元素再递归去执行这个流程。

## View的测量过程

View的measure过程是由其measure方法来完成，measure方法是一个final类型的方法，这意味着子类不能重写此方法，在View的measure方法中会去调用View的onMeasure方法，因此只需要看onMeasure的实现即可，代码如下所示：

```Java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
			getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

如上所示，setMeasuredDimension方法会设置View宽/高的测量值，因此我们只需要看getDefaultSize这个方法即可：

## 测量的三种模式

1. EXACTLY：精确值模式  
 当我们将控件的layout\_width或layout\_height属性指定为具体的数值时，或指定为match\_parent属性时，系统使用的是EXACTLY模式。

2. AT_MOST：最大值模式  
 当我们将控件的layout\_width或layout\_height属性指定为wrap\_content时,系统使用的是AT\_MOST模式  

3. UNSPECIFIED：  
 此模式不指定其大小测量模式，View想多大就多大，通常情况系统绘制自定义View时才会使用  

**Note:: View类默认的onMeasure()方法只支持EXACTLY模式，而如果要让自定义View支持wrap\_content属性时，那么必须重写onMeasure()方法来指定wrap\_content的大小**

```Java
public static int getDefaultSize(int size, int measureSpec) {
	int result = size;
	int specMode = MeasureSpec.getMode(measureSpec); //测量模式
	int specSize = MeasureSpec.getSize(measureSpec); //绘制的大小

	switch (specMode) {
	case MeasureSpec.UNSPECIFIED:
		result = size;
		break;
	case MeasureSpec.AT_MOST: //重写指定AT_MOST模式的值
	case MeasureSpec.EXACTLY:
		result = specSize;
		break;
	}
	return result;
}
```

可以看出，其实getDefaultSize返回的大小就是measureSpec中的specSize，而这个specSize就是View测量后的大小，这里多次提到测量后的大小是因为View的最终大小是在layout阶段确定的，所以必须加以区分，但是几乎所有情况下View的测量大小和最终大小是相等的。

在上述情况下，View的大小为getDefaultSize的第一个参数size，及宽高分别为getSuggestedMinimumWidth和getSuggestedMinimumHeight的返回值，源码如下：

```Java
protected int getSuggestedMinimumWidth() {
	return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
}

protected int getSuggestedMinimumHeight() {
	return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());

}
```


# layout过程

Layout的作用是ViewGroup用来确定子元素的位置，当ViewGroup的位置被确定后，它在onLayout中会遍历所有子元素并调用其layout方法，View的layout方法中onLayout方法又会被调用。layout方法确定View本身的位置，而onLayout方法则会确定所有子元素的位置，View的layout方法如下：

```Java
public void layout(int l, int t, int r, int b) {
	if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
		onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
		mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
	}

	int oldL = mLeft;
	int oldT = mTop;
	int oldB = mBottom;
	int oldR = mRight;

	boolean changed = isLayoutModeOptical(mParent) ?
			setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

	if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
		onLayout(changed, l, t, r, b);
		mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

		ListenerInfo li = mListenerInfo;
		if (li != null && li.mOnLayoutChangeListeners != null) {
			ArrayList<OnLayoutChangeListener> listenersCopy =
					(ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
			int numListeners = listenersCopy.size();
			for (int i = 0; i < numListeners; ++i) {
				listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
			}
		}
	}

	mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
	mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
}
```

layout方法大致流程如下：首先通过setFrame方法设定View的四个顶点的位置，即初始化mLeft、mTop、mBottom、mRight这四个值，View的四个顶点一旦确定，那么View在父容器的位置也就确定了；接着会调用onLayout方法，用途是父容器确定子元素的位置，和onMeasure方法类似，onLayout的具体实现同样和具体的布局有关，所以View和ViewGroup均没有真正的实现。我们可以看一下TextView的onLayout方法，如下所示：

```Java
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
	super.onLayout(changed, left, top, right, bottom);
	if (mDeferScroll >= 0) {
		int curs = mDeferScroll;
		mDeferScroll = -1;
		bringPointIntoView(Math.min(curs, mText.length()));
	}
}
```

# draw过程

Draw过程较简单，作用是将View绘制到屏幕上面。

当测量好一个View之后，我们就可以重写onDraw方法，并在Canvas对象上绘制所需要的图形，最终显示在屏幕上。

# 参考文献

Android开发艺术探索

