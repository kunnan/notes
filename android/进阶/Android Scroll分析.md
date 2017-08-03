Android 滑动分析

[TOC]

# 滑动效果是如何产生的

> 滑动一个View，本质上来说就是移动一个View。改变其当前所处的位置，它的原理都是通过不断改变View的坐标来实现这一效果。要实现View的滑动就必须监听用户触摸的事件，并根据事件传入的坐标，动态且不断改变View的坐标，从而实现View跟随用户触摸的滑动而滑动。

# Android坐标系

> Android中，将屏幕最左上角的顶点作为Android坐标系的原点，从这个点向右是X轴正方向，向下是Y轴正方向。  
> 系统提供了`getLocationOnScreen(intlocation[])`这样的方法来获取Android坐标系中点的位置，即该视图左上角Android坐标系中的坐标。另外，在触控事件中使用`getRawX()`、`getRawY()`方法所获取的坐标同样是Android坐标系中的坐标。

![ALT TEXT](http://img.blog.csdn.net/20140808090119158?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveXVhbl9jaG9uZ2ppZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## 视图坐标系

> 视图坐标系，它描述了子视图在父视图中的位置关系。视图坐标系同样以原点向右为X轴正方向，以原点向下为Y轴正方向，只不过在视图坐标系中，原点不再是Android坐标系中的屏幕左上角，而是以父视图左上角为坐标原点。  
> 在触控事件中，通过`getX()`、`getY()`所获得的坐标就是视图坐标系中的坐标。  

![ALT TEXT](http://img.blog.csdn.net/20140808090814078?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveXVhbl9jaG9uZ2ppZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## 触控事件-MotionEvent  

触控事件的常用事件常量，定义触控事件的不同类型。

```
//单点触摸按下动作
public static final int ACTION_DOWN = 0;

//单点触摸离开动作
public static final int ACTION_UP = 1;

//触摸点移动动作
public static final int ACTION_MOVE = 2;

//触摸动作取消
public static final int ACTION_CANCEL = 3;

//触摸动作超出边界
public static final int ACTION_OUTSIDE = 4;

//多点触摸
public static final int ACTION_POINTER_DOWN = 5;

//多点离开动作
public static final int ACTION_POINTER_UP = 6;
```

通常情况下，我们会在`onTouchEvent(MotionEvent event`)方法中通过`event.getAction()`方法来获取触控事件的类型，并使用switch-case方法来进行筛选，这个代码的模式基本固定，如下所示

```
public boolean onTouchEvent(MotionEvent event) {

	//获取当前输入点的X、Y坐标（视图坐标）
	int x = (int) event.getX();
	int y = (int) event.getY();
	
	switch (event.getAction()) {
	
	case MotionEvent.ACTION_DOWN:
		//处理输入的按下事件
		break;

	case MotionEvent.ACTION_MOVE:
		//处理输入的移动事件
		break;
		
	case MotionEvent.ACTION_UP:
		//处理输入的离开事件
		break;
	}
	return true;
}
```

在Android中，系统提供了非常多的方法来获取坐标值
相对距离等。下面总结一些API结合Android坐标系来看看该如何使用它们

![ALT TEXT](http://img.blog.csdn.net/20150115155321445?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamFzb24wNTM5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## View获取自身宽高

* getHeight()：获取View自身高度
* getWidth()：获取View自身宽度

## View获取坐标方法

*	**getTop():** 获取到的是View自身的顶边到其父视图顶边的距离

*	**getLeft():** 获取到的是View自身的左边到其父视图左边的距离

*	**getRight():** 获取到的是View自身的右边到其父视图的距离

*	**getBottom():** 获取到的是View自身的底边到其父视图顶边的距离  

## 	MotionEvent提供的方法

*	**getX():** 获取点击事件距离控件左边的距离，即视图坐标

*	**getY():** 获取点击事件距离控件顶边的距离，即视图坐标

*	**getRawX():** 获取点击事件距离屏幕左边的距离，即绝对坐标

*	**getRawY():** 获取点击事件距离屏幕顶边的距离，即绝对坐标


# 实现滑动的方法

滑动效果实现的思想基本上是一致的，当触摸View时，系统记下当前触摸点坐标；当手指移动时，系统记下移动后的触摸点坐标，从而获取到相对于前一次坐标点的偏移量，并通过偏移量来修改View的坐标，这样不断重复，从而实现滑动过程。

## layout方法

我们知道，在View进行绘制时，会调用onLayout()方法来设置显示的位置。同样，可以通过修改View的left、top、right、bottom四个属性来控制View的坐标。在每次回调onTouchEvent的时候，我们都来获取一下触摸点的坐标:

```
int x = (int)event.getX();
int y = (int)event.getY();
```

接着在Action_DOWN事件只能怪记录触摸点的坐标

```
case MotionEvent.ACTION_DOWN:
	//记录触摸点坐标
	lastX = x;
	lastY = y;
	break;
```

最后，可以在ACTION_MOVE事件中计算偏移量，并将偏移量作用到layout方法中，目前layout的left、top、right、bottom基础上，增加计算出来的偏移量

```
// 视图坐标方式
@Override
public boolean onTouchEvent(MotionEvent event) {
	// TODO Auto-generated method stub

	int x = (int) event.getX();
	int y = (int) event.getY();

	switch (event.getAction()) {

	case MotionEvent.ACTION_DOWN:

		lastX = x;
		lastY = y;

		break;

	case MotionEvent.ACTION_MOVE:

		int offsetX = x - lastX;
		int offsetY = y - lastY;

		// 在当前left、top、right、bottom的基础上加上偏移量
		layout(getLeft() + offsetX, getTop() + offsetY, getRight()
				+ offsetX, getBottom() + offsetY);

		break;
	}

	return true;
}
```

这样每次移动后，View都会调用Layout方法来对自己重新布局，从而达到移动View的效果

在上面的代码中，使用的是getX()、getY()方法获取坐标值，即通过视图坐标来获取偏移量。当然，同样可以使用getRawX()、getRawY()来获取坐标，并使用绝对坐标来计算偏移量。

```
//绝对坐标方式
@Override
public boolean onTouchEvent(MotionEvent event) {
	// TODO Auto-generated method stub

	int rawX = (int) event.getRawX();
	int rawY = (int) event.getRawY();

	switch (event.getAction()) {

	case MotionEvent.ACTION_DOWN:

		//记录触摸点坐标
		lastX = rawX;
		lastY = rawY;
		
		break;

	case MotionEvent.ACTION_MOVE:

		int offsetX = rawX - lastX;
		int offsetY = rawY - lastY;

		// 在当前left、top、right、bottom的基础上加上偏移量
		layout(getLeft() + offsetX, getTop() + offsetY, getRight()
				+ offsetX, getBottom() + offsetY);
		
		//重新设置初始坐标
		lastX = rawX;
		lastY = rawY;
		
		break;
	}

	return true;
}
```

> **Note:** 使用绝对坐标时，有一点要非常注意，就是每次执行完ACTION_DOWN的逻辑后，一定要重新设置初始坐标，这样才能准确的获取偏移量。

## offsetLeftAndRight()与offsetTopAndBottom()

这个方法相当于系统提供了一个对左右、上下移动的API的封装。当计算出偏移量后，只需要使用如下方法就可以完成View的重新布局，效果与使用Layout方法一样

```
@Override
public boolean onTouchEvent(MotionEvent event) {
	// TODO Auto-generated method stub

	int x = (int) event.getX();
	int y = (int) event.getY();

	switch (event.getAction()) {

	case MotionEvent.ACTION_DOWN:

		lastX = x;
		lastY = y;

		break;

	case MotionEvent.ACTION_MOVE:

		int offsetX = x - lastX;
		int offsetY = y - lastY;

		// 在当前left、top、right、bottom的基础上加上偏移量
//			layout(getLeft() + offsetX, getTop() + offsetY, getRight()
//					+ offsetX, getBottom() + offsetY);
		
		offsetLeftAndRight(offsetX);
		offsetTopAndBottom(offsetY);

		break;
	}

	return true;
}
```

## LayoutParams

LayoutParams保存了一个View的布局参数。因此可以在程序中，提供改变LayoutParams来动态地修改一个布局的位置参数，从而达到改变View位置的效果。

```
@Override
public boolean onTouchEvent(MotionEvent event) {
	// TODO Auto-generated method stub

	int x = (int) event.getX();
	int y = (int) event.getY();

	switch (event.getAction()) {

	case MotionEvent.ACTION_DOWN:

		lastX = x;
		lastY = y;

		break;

	case MotionEvent.ACTION_MOVE:

		int offsetX = x - lastX;
		int offsetY = y - lastY;

		LinearLayout.LayoutParams params = (LayoutParams) getLayoutParams();
		params.leftMargin = getLeft() + offsetX;
		params.topMargin = getTop() + offsetY;
		setLayoutParams(params);
		
		break;
	}

	return true;
}
```

**这里需要注意的是**，通过getLayoutParams()获取LayoutParams时，需要根据View所在父布局的类型来设置不同的类型，比如这里将View放在LinearLayout中，那么就可以使用LinearLayout.LayoutParams。类似的，如果在RelativeLayout中，就使用RelativeLayout.LayoutParams。当然，这一切的前提是你必须要有一个父布局，不然系统无法获取LayoutParams。

在通过改变LayoutParams来改变一个View的位置时，通常改变的是这个View的Margin属性，所以除了使用布局的LayoutParams之外，还可以使用ViewGroup.MarginLayoutParams来实现在这样的功能，此时不需要考虑父布局的类型，当然上述两种方式的本质都是一样的。

```
ViewGroup.MarginLayoutParams params = (MarginLayoutParams) getLayoutParams();
params.leftMargin = getLeft() + offsetX;
params.topMargin = getTop() + offsetY;
setLayoutParams(params);
```

## scrollTo与scrollBy

在一个View中，系统提供了scrollTo、sccrollBy两种方式改变一个View的位置。`scrollTo(x,y)`表示移动到一个具体的坐标点(x,y)，`scrollBy(dx,dy)`表示移动的增量为dx、dy。

```
int offsetX = x - lastX;
int offsetY = y - lastY;
scrollBy(offsetX, offsetY); 
```

运行上述程序，当我们拖动View的时候，发现View并没有移动。原因是：scrollTo、scrollBy方法移动的是View的content，即让View的内容移动，如果在ViewGroup中使用scrollTo、scrollBy方法，那么移动的将是所有子View，但如果在子View中使用，那么移动的将是View的内容。例如TextView，content就是它的文本；ImageView，content就是它的drawable对象。

```
int offsetX = x - lastX;
int offsetY = y - lastY;
((View) getParent()).scrollBy(offsetX, offsetY);
```

运行上述程序后，当拖动View的时候，View虽然移动了，但View在乱动，并不是我们想要的跟随触摸点的移动而移动。如果将scrollBY中的参数dx和dy设置为正数，那么content将向坐标轴负方向移动；如果将scrollBY中的参数dx和dy设置为负数，那么content将向坐标轴正方向移动。所有要实现跟随手指移动和滑动的效果，就必须将偏移量改为负值。

```
int offsetX = x - lastX;
int offsetY = y - lastY;
((View) getParent()).scrollBy(-offsetX, -offsetY);
```

如下，就实现了我们想要的效果。**需要注意的是，此种方式会导致ViewGroup上所有的子View同时移动.**

scrollTo()和scrollBy()的意义，那么举个例子，感性地认识一下。

假设有一个View，它叫做SView。
如果想把SView从(0, 0)移动到(100, 100)。注意，这里说的(0, 0)和(100, 100)，指的是SView左上角的坐标。那么偏移量就是原点(0, 0)到目标点(100, 100)的距离，即(0 , 0) - (100, 100) = (-100, -100)。
只需要调用SView.scrollTo(-100, -100)就可以了。请再次注意，scrollTo(int x, int y)的两个参数x和y，代表的是偏移量，这时的参照物是(0, 0)点。
然而，scrollBy()是有一定的区别的。scrollBy()的参照物是(0, 0)点加上偏移量之后的坐标。
这么描述比较抽象，举个例子。假设SView调用了scrollTo(-100, -100)，此时SView左上角的坐标是(100, 100)，这时再调用scrollBy(-20, -20)，此时SView的左上角就被绘制到了(120, 120)这个位置。

总结一下，scrollTo()是一步到位，而scrollBy()是逐步累加。


## Scroller 

Scroll类可以实现平滑移动的效果，而不再是瞬间完成的移动。

下面演示一下Scroller类如何实现平移滑动。在这个实例中，同样让子View跟随手指的滑动而滑动，但是在手指离开屏幕时，让子View平滑的移动到初始位置，即屏幕左上角。一般情况下，使用Scroller类需要如下三个步骤。

*	初始化Scroller

首先，通过它的构造方法来创建一个Scroller对象

```
private void ininView(Context context) {
    setBackgroundColor(Color.BLUE);
    // 初始化Scroller
    mScroller = new Scroller(context);
}
```

*	重写computeScroll()方法，实现模拟滑动

系统在绘制View的时候会在draw()方法中调用computeScroll()方法

```
@Override
public void computeScroll() {
    super.computeScroll();
    // 判断Scroller是否执行完毕
    if (mScroller.computeScrollOffset()) {
        ((View) getParent()).scrollTo(
                mScroller.getCurrX(),
                mScroller.getCurrY());
        // 通过重绘来不断调用computeScroll
        invalidate();
    }
}
```

*	startScroll开启模拟过程

使用Scroller类的startScroll()方法来开启平滑移动过程。startScroll()方法具有两个重载方法。

*	public void startScroll(int startX, int startY, int dx, int dy, int duration)
*	public void startScroll(int startX, int startY, int dx, int dy)

区别在于一个具有指定的持续时长，而另一个则没有。  

开启实现平滑移动到原位置，代码如下:

```
@Override
public boolean onTouchEvent(MotionEvent event) {
    int x = (int) event.getX();
    int y = (int) event.getY();
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            lastX = (int) event.getX();
            lastY = (int) event.getY();
            break;
        case MotionEvent.ACTION_MOVE:
            int offsetX = x - lastX;
            int offsetY = y - lastY;
            ((View) getParent()).scrollBy(-offsetX, -offsetY);
            break;
        case MotionEvent.ACTION_UP:
            // 手指离开时，执行滑动过程
            View viewGroup = ((View) getParent());
            mScroller.startScroll(
                    viewGroup.getScrollX(),
                    viewGroup.getScrollY(),
                    -viewGroup.getScrollX(),
                    -viewGroup.getScrollY(),1000);
            invalidate();
            break;
    }
    return true;
}
```


## 属性动画

可以采用View动画来移动，在res目录新建anim文件夹并创建translate.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate android:fromXDelta="0" android:toXDelta="300" android:duration="1000"/>
</set>
```
使用属性动画移动:

```java
ObjectAnimator.ofFloat(mCustomView,"translationX",0,300).setDuration(1000).start();
```


