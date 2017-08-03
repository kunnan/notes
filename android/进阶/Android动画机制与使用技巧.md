Android动画机制与使用技巧

[TOC]

# Android视图动画

> Android框架定义了透明度、旋转、位移、缩放几种常见的动画，而且控制的是整个View，实现原理是每次绘制视图时View所在的ViewGroup中的drawChild函数获取该View的Animation的Transformation值，然后调用`canvas.concat(transformToApply.getMatrix())`，通过矩阵运算完成动画帧。如果动画没有完成，就继续调`invalidate()`函数，启动下次绘制来驱动动画，从而能完成整个动画的绘制。  
效率高，使用
* **优点:** 方便  
* **缺点:** 不具备交互性，当某个元素发生视图动画后，其响应事件的位置依然停留在动画前的位置，所以视图动画只能做普通的动画效果，使用时应避免交互的发生。  

## 透明度动画  

> alpha可以实现透明度渐变的动画效果，也就是淡入淡出的效果，可通过设置下面三个属性来设置淡入或淡出效果：

```
AlphaAnimation aa = new AlphaAnimation(0, 1);
aa.setDuration(1000);
view.startAnimation(aa);   
```

> 属性

- android:duration 动画从开始到结束持续的时长，单位为毫秒  
- android:fromAlpha 动画开始时的透明度，0.0为全透明，1.0为不透明，默认为1.0  
- android:toAlpha 动画结束时的透明度，0.0为全透明，1.0为不透明，默认为1.0  
- 当设置开始时透明度为0.0，结束时为1.0，就能实现淡入效果；相反，当设置开始时透明度为1.0，结束时为0.0，那就能实现淡出效果

> 示例

```
<!-- res/anim/fade_in.xml -->
<?xml version="1.0" encoding="utf-8"?>
<alpha xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:fromAlpha="0.0"
    android:toAlpha="1.0" />  
```

> 将Xml动画添加到view上  

```
view.startAnimation(AnimationUtils.loadAnimation(this, R.anim.fade_in)); 

或者

AlphaAnimation fadeInAnimation = (AlphaAnimation) AnimationUtils.loadAnimation(this, R.anim.fade_in);
view.startAnimation(fadeInAnimation);
```

## 旋转动画  

> RotateAnimation可以实现旋转的动画效果，主要的属性如下：  

- android:duration 动画从开始到结束持续的时长，单位为毫秒
- android:fromDegrees 旋转开始的角度
- android:toDegrees 旋转结束的角度
- android:pivotX 旋转中心点的X坐标，纯数字表示相对于View本身左边缘的像素偏移量；带"%"后缀时表示相对于View本身左边缘的百分比偏移量；带"%p"后缀时表示相对于父View左边缘的百分比偏移量
- android:pivotY 旋转中心点的Y坐标，纯数字表示相对于View本身顶部边缘的像素偏移量；带"%"后缀时表示相对于View本身顶部边缘的百分比偏移量；带"%p"后缀时表示相对于父View顶部边缘的百分比偏移量  

> 以下示例代码旋转角度从0到360，即旋转了一圈，旋转的中心点都设为了50%，即是View本身中点的位置。

```
<!-- res/anim/rotate_one.xml -->
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
android:duration="2000"
android:fromDegrees="0"
android:toDegrees="360"
android:pivotX="50%"
android:pivotY="50%" />
```

> 任意旋转参考系  

```
RotateAnimation ra = new RotateAnimation(0, 360, 100, 100);
ra.setDuration(1000);
view.startAnimation(ra);  
```

> 旋转参考系为自身中心点  

```
RotateAnimation ra = new RotateAnimation(0, 360,
        RotateAnimation.RELATIVE_TO_SELF, 0.5F,
        RotateAnimation.RELATIVE_TO_SELF, 0.5F);
ra.setDuration(1000);
view.startAnimation(ra);
```

## 位移动画  

```
TranslateAnimation ta = new TranslateAnimation(0, 200, 0, 300);
ta.setDuration(1000);
view.startAnimation(ta);
```

> **translate**可以实现位置移动的动画效果，可以是垂直方向的移动，也可以是水平方向的移动。
> 坐标的值可以有三种格式：从-100到100，"%"结束，表示相对于View本身的百分比位置；如果以"%p"结束，表示相对于View的父View的百分比位置；如果没有任何后缀，表示相对于View本身具体的像素值。主要的属性如下：  

- android:duration 动画从开始到结束持续的时长，单位为毫秒
- android:fromXDelta 起始位置的X坐标的偏移量
- android:toXDelta 结束位置的X坐标的偏移量
- android:fromYDelta 起始位置的Y坐标的偏移量
- android:toYDelta 结束位置的Y坐标的偏移量

> 示例，以下代码实现的是从左到右的移动效果，起始位置为相对于控件本身-100%的位置，即在控件左边，与控件本身宽度一致的位置；结束位置为相对于父控件100%的位置，即会移出父控件右边缘的位置。

```
<!-- res/anim/move_left_to_right.xml -->
<?xml version="1.0" encoding="utf-8"?>
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="2000"
    android:fromXDelta="-100%"
    android:fromYDelta="0"
    android:toXDelta="100%p"
    android:toYDelta="0" />
```


## 缩放动画  

> 属性如下:  

*	android:duration 动画从开始到结束持续的时长，单位为毫秒  
*	android:fromXScale 动画开始时X坐标上的缩放尺寸
*	android:toXScale 动画结束时X坐标上的缩放尺寸
*	android:fromYScale 动画开始时Y坐标上的缩放尺寸
*	android:toYScale 动画结束时Y坐标上的缩放尺寸  
**PS：以上四个属性，0.0表示缩放到没有，1.0表示正常无缩放，小于1.0表示收缩，大于1.0表示放大**
*	android:pivotX 缩放时的固定不变的X坐标，一般用百分比表示，0%表示左边缘，100%表示右边缘
*	android:pivotY 缩放时的固定不变的Y坐标，一般用百分比表示，0%表示顶部边缘，100%表示底部边缘  

> xml布局动画  

```
<!-- res/anim/zoom_out.xml -->
<?xml version="1.0" encoding="utf-8"?>
<scale xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:fromXScale="1.0"
    android:fromYScale="1.0"
    android:pivotX="0%"
    android:pivotY="100%"
    android:toXScale="1.5"
    android:toYScale="1.5" />
```

> 任意缩放位置  

```
ScaleAnimation sa = new ScaleAnimation(0, 2, 0, 2);
sa.setDuration(1000);
view.startAnimation(sa);
```

> 缩放中心点为自身中心点  

```
ScaleAnimation sa = new ScaleAnimation(0, 1, 0, 1,
        Animation.RELATIVE_TO_SELF, 0.5F,
        Animation.RELATIVE_TO_SELF, 0.5F);
sa.setDuration(1000);
view.startAnimation(sa);
```

## 动画集合

> 代码

```
AnimationSet as = new AnimationSet(true);
as.setDuration(1000);

AlphaAnimation aa = new AlphaAnimation(0, 1);
aa.setDuration(1000);
as.addAnimation(aa);

TranslateAnimation ta = new TranslateAnimation(0, 100, 0, 200);
ta.setDuration(1000);
as.addAnimation(ta);

view.startAnimation(as);  
```

> 布局

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="2000">
    <translate
        android:fromXDelta="0"
        android:fromYDelta="0"
        android:toXDelta="200%"
        android:toYDelta="0" />
    <scale
        android:fromXScale="1.0"
        android:fromYScale="1.0"
        android:pivotX="0%"
        android:pivotY="100%"
        android:toXScale="1.5"
        android:toYScale="1.5" />
</set>  
```

布局加载:

```
AnimationSet as = (AnimationSet) AnimationUtils.loadAnimation(this,anim.fade_in);
view.startAnimation(as);
```

## 动画监听  

```
animation.setAnimationListener(new AnimationListener() {
	
	@Override
	public void onAnimationStart(Animation animation) {
		// 开始动画
		
	}
	
	@Override
	public void onAnimationRepeat(Animation animation) {
		// 动画执行
		
	}
	
	@Override
	public void onAnimationEnd(Animation animation) {
		// 动画结束
		
	}
});
```

# Android属性动画

> Android 3.0之后推出。属性动画通过调用属性的get、set方法来真实的改变了一个View的属性，所以事件响应的区域也同样发生了改变，点击动画移动后的按钮，就会响应点击事件。 

## ObjectAnimator

> ObjectAnimator是属性动画框架中最重要的实行类，创建一个ObjectAnimator只需通过它的静态工厂类直接返回一个ObjectAnimator对象。参数包括一个对象和对象的属性名字，**这个属性必须有get和set函数，内部会通过java反射机制来调用set函数修改对象属性值。**同样，你也可以调用`setInterpolator()`设置相应的差值器。

```
@param 1 需要操纵的view
@param 2 需要操纵的属性  
@param 3 可变数组参数，需要传入去改变属性的变化   

ObjectAnimator animator = ObjectAnimator.ofFloat(view,
        "alpha", 0.5F, 1F);
//设置差值器
animator.setInterpolator(new BounceInterpolator());
animator.setDuration(500);  
animator.start();  
```

> 常用的可以直接使用属性动画的属性值。  

*	**translationX和translationY：** 这两个属性作为一种增量来控制着View对象从它布局容器的左上角坐标偏移的位置。
*	**rotation、rotationX和rotationY：** 这三个属性控制View对象围绕支点进行2D和3D旋转。  
*	**scaleX和scaleY：** 这两个属性控制View对象围绕它的支点进行2D缩放。
*	**pivotX和pivotY：** 这两个属性控制着View对象的支点位置，围绕这个支点进行旋转和缩放变换处理。默认情况下，该支点的位置就是View对象的中心点。
*	**x和y：** 它描述了View对象在它的容器中的最终位置，它是最初的左上角坐标和translationX、translationY值的累计和。
*	**alpha：** 它表示view对象的alpha透明度。

> **如果一个属性没用get、set方法，应用层提供两种解决方案来解决这个问题:** 

1. 自定义一个属性类或者包装类，间接地给这个属性增加get、set方法  

```
private class  WrapperView {
	
	private View mTarget;

	public WrapperView(View target) {
	
		mTarget = target;
	}

	public int getWidth() {
		
		return mTarget.getLayoutParams().width;
	}

	public voide setWidth(int width) {
	
		mTarget.getLayoutParams().width = width;
		mTarget.requestLayout();
	
	}		
}  

ViewWrapper wrapper = new ViewWrapper(mButton);
ObjectAnimator.ofInt(wrapper,"width",500).setDuration(5000).start();
``` 

## PropertyValuesHolder

> 类似视图动画中的AnimationSet，在属性动画中，如果针对同一个对象的多个属性，要同时作用多种动画，可以使用PropertyValuesHolder来实现。

```
//平移
PropertyValuesHolder pvh1 =  PropertyValuesHolder.ofFloat("translationX",300f);
//缩放
PropertyValuesHolder pvh2 =  PropertyValuesHolder.ofFloat("scaleX",1.0f,0,1.0f);
PropertyValuesHolder pvh3 =  PropertyValuesHolder.ofFloat("scaleY",1.0f,0,1.0f);
ObjectAnimator.ofPropertyValuesHolder(view,pvh1,pvh2,pvh3).setDuration(1000).start();
```


## ValueAnimator

> ValueAnimator是属性动画的核心所在，ObjectAnimator也是继承自ValueAnimator。  
> ValueAnimator本身不提供任何动画效果，它更像一个数值发生器，用来产生具有一定规律的数字，从而让调用者来控制动画的实现过程，ValueAnimator的一般使用方法如下所示。通常情况下，在ValueAnimator的AnimatorUpdateListener中监听数值的变换，从而完成动画的变换。

```	
ValueAnimator animator = ValueAnimator.ofFloat(0,100);
animator.setTarget(view);
animator.setDuration(1000).start();
animator.addUpdateListener(new AnimatorUpdateListener() {

	@Override
	public void onAnimatorUpdate(ValueAnimator animation) {
		
		Float value = (Float)animation.getAnimatedValue();
		// use the value
	}

});
```

## 动画事件的监听

> 一个完整的动画具有Start、Repeat、End、Cancel四个过程。通过Android提供了接口可以方便地监听到四个事件。  

```
ObjectAnimator anima = ObjectAnimatorofFloat(view,"alpha",0.5f);
anim.addListener(new AnimatorListner() {

	@Override
	public void onAnimationStart(Animator animator) {

	}

	@Override
	public void onAnimationRepeat(Animator animator) {

	}

	@Override
	public void onAnimationEnd(Animator animator) {

	}

	@Override
	public void onAnimationCancel(Animator animator) {

	}

});
anim.start();  
```

> 大部分时候，我们只关心`onAnimationEnd()`事件，所以Android也提供了一个AnimatorListnerAdapter来让我们选择必要的事件进行监听。  

```
anim.addListner(new AnimatorListenerAdapter() {

	@Override
	public void onAnimationEnd(Animator animator) {

	}

});
```

## AnimatorSet

> 对于一个属性同时作用于多个属性动画效果，用PropertValuesHolder可以实现这样的效果。而AnimatorSet不仅能实现这样的效果，同时也能实现更加精准的顺序控制。

```
ObjectAnimator animator0 = ObjectAnimator.ofFloat(view,
        "alpha", 0.5F, 1F);
ObjectAnimator animator1 = ObjectAnimator.ofFloat(view,
        "translationY", 200F, 0);
ObjectAnimator animator2 = ObjectAnimator.ofFloat(view,
        "translationX", 200F, 0);
ObjectAnimator animator3 = ObjectAnimator.ofFloat(view,
        "translationY", -200F, 0);
ObjectAnimator animator4 = ObjectAnimator.ofFloat(view,
        "translationX", -200F, 0);
AnimatorSet set = new AnimatorSet();
set.setDuration(500);
set.setInterpolator(new BounceInterpolator());
set.playTogether(animator0, animator1, animator2, animator3, animator4);
set.start();  
```

> 在属性动画中，AnimatorSet正是通过`playTogether()、playSequentially()、animSet.play().with()、before()、after()`这些方法来控制多个动画的协作方式，从而做到对动画播放顺序的精确控制。

## Xml中使用属性动画

> 属性动画的xml文件则放于res/animator/目录,不同的是，视图动画的xml文件放于res/anim/目录下

```
<?xml verson="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemes.android.com/apk/res/android"
	android:duration="1000"
	android:propertyName="sacleX"
	android:valueFrom="1.0"
	android:valueTo="2.0"
	android:valueType="floatType"
</objectAnimator>  
```

> 在程序中使用xml定义的属性动画如下:  

```
Animator anim = AnimatorInflator.loadAnimation(this,R.animator.scalex);
anim.setTarget(view);
anim.set();  
```

## View的animate方法

> 在Android3.0以后，Google给View增加了animate方法来直接驱动属性动画，其实animate方法可以认为是属性动画的一种简写方式。

```
view.animate()
			.alpha(0)
			.y(300)
			.setDuration(1000)
			.withStartAction(new Runnable() {
				
				@Override
				public void run() {

				}
			})
			.withEndAction(new Runnable() {
		
				@Override
				public void run() {
					runOnUiThread(new Runnable() {
						@Override
						public void run() {
						
						}
					});
				}

			}).start();  
```

# Android布局动画

> Android布局动画是指作用在ViewGroup上，给ViewGroup增加View时添加一个动画过渡效果。最简单的布局动画是在ViewGroup的XML中，使用一下代码打开布局动画，这个效果是Android默认的显示的过渡效果,`android:animateLayoutChanges="true"`.

> 通过使用LayoutAnimationController类来自定义一个子View的过渡效果。

```
LinearLayout ll = (LinearLayout)findViewById(R.id.ll);
//设置过渡动画
ScaleAnimation sa = new ScaleAnimation(0,1,0,1);
sa.setDuration(1000);
//设置布局动画的显示属性
LayoutAnimationController lac = new LayoutAnimationController(sa,0.5F);
lac.setOrder(LayoutAnimationController.ORDER_NORMAL);
//为ViewGroup设置布局动画
ll.setLayoutAnimation(lac);  
```

> LayoutAnimationController的第一个参数是需要作用的动画，第二个参数，则是每个子View显示的delay时间，当delay时间不为0时，可以显示子View显示的顺序。

*	LayoutAnimationController.ORDER_NORMAL ---顺序
*	LayoutAnimationController.ORDER_RANDOM ---随机
*	LayoutAnimationController.ORDER_REVERSE --反序


# Interpolators(插值器)

## 概念

> 插值器是动画中一个重要的概念，通过插值器可以定义动画变换速率，类似物理中的加速度，其作用主要是控制目标变量的变化值进行对应的变化。同样的一个动画变换起始值，在不同插值器的作用下，每个单位内所达到的变化值也是不一样的。如线性插值器、加速度插值器

*	AccelerateDecelerateInterpolator 在动画开始与结束的地方速率改变比较慢，在中间的时候加速

*	AccelerateInterpolator  在动画开始的地方速率改变比较慢，然后开始加速

*	AnticipateInterpolator 开始的时候向后然后向前甩

*	AnticipateOvershootInterpolator 开始的时候向后然后向前甩一定值后返回最后的值

*	BounceInterpolator   动画结束的时候弹起

*	CycleInterpolator 动画循环播放特定的次数，速率改变沿着正弦曲线

*	DecelerateInterpolator 在动画开始的地方快然后慢

*	LinearInterpolator   以常量速率改变

*	OvershootInterpolator    向前甩一定值后再回到原来位置

## 系统分类

> 通过interpolator可以定义动画速率变化的方式，比如加速、减速、匀速等，每种interpolator都是Interpolator 类的子类，Android系统已经实现了多种interpolator，对应也提供了公共的资源ID，如下表：


|	Interpolator class	|	Resource ID	|	Description	  
|-----------------------|---------------|--------------  
|AccelerateDecelerateInterpolator|@android:anim/accelerate_decelerate_interpolator|在动画开始与结束时速率改变比较慢，在中间的时候加速
|AccelerateInterpolator|@android:anim/accelerate_interpolator|在动画开始时速率改变比较慢，然后开始加速
|AnticipateInterpolator|@android:anim/anticipate_interpolator|动画开始的时候向后然后往前抛
|AnticipateOvershootInterpolator|@android:anim/anticipate_overshoot_interpolator|动画开始的时候向后然后向前抛，会抛超过目标值后再返回到最后的值
|BounceInterpolator|@android:anim/bounce_interpolator|动画结束的时候会弹跳
|CycleInterpolator|@android:anim/cycle_interpolator|动画循环做周期运动，速率改变沿着正弦曲线
|DecelerateInterpolator|@android:anim/decelerate_interpolator|在动画开始时速率改变比较快，然后开始减速
|LinearInterpolator|@android:anim/linear_interpolator|动画匀速播放
|OvershootInterpolator|@android:anim/overshoot_interpolator|动画向前抛，会抛超过最后值，然后再返回

## 自定义  

> 自定义的方式有两种，一种是通过继承 Interpolator 父类或其子类；另一种是通过自定义的xml文件，可以更改上表中Interpolator的属性。 

### 实现Interpolator

```Java
public class MyInterpolator implements Interpolator {
  @Override public float getInterpolation(float input) {
    return 0;
  }
}
```

### 自定义xml

> 自定义的xml文件需存放于res/anim/目录下，根标签与上表相应的有九种如下：

- accelerateDecelerateInterpolator 在动画开始与结束时速率改变比较慢，在中间的时候加速。没有可更改设置的属性，所以设置的效果和系统提供的一样
- accelerateInterpolator 在动画开始时速率改变比较慢，然后开始加速。有一个属性可以设置加速的速率
**android:factor**浮点值，加速的速率，默认为1
- anticipateInterpolator动画开始的时候向后然后往前抛。有一个属性设置向后拉的值**android:tension** 浮点值，向后的拉力，默认为2，当设为0时，则不会有向后的动画了
- anticipateOvershootInterpolator动画开始的时候向后然后向前抛，会抛超过目标值后再返回到最后的值。可设置两个属性:**android:tension** 浮点值，向后的拉力，默认为2，当设为0时，则不会有向后的动画了;**android:extraTension** 浮点值，拉力的倍数，默认为1.5(2*1.5)，当设为0时，则不会有拉力了
- bounceInterpolator动画结束的时候会弹跳。没有可更改设置的属性
- cycleInterpolator动画循环做周期运动，速率改变沿着正弦曲线。有一个属性设置循环次数**android:cycles 整数值**，循环的次数，默认为1
- decelerateInterpolator 在动画开始时速率改变比较快，然后开始减速。有一个属性设置减速的速率**android:factor 浮点值**，减速的速率，默认为1
- linearInterpolator 动画匀速播放。没有可更改设置的属性
- overshootInterpolator 动画向前抛，会抛超过最后值，然后再返回。有一个属性**android:tension** 浮点值，超出终点后的拉力，默认为2

> 示例

先定义个interpolator的xml文件:

```
<?xml version="1.0" encoding="utf-8"?>
<anticipateOvershootInterpolator
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:tension="3"
    android:extraTension="2" />
```

接着，将其设置到要应用的动画的android:interpolator属性即可，代码如下：

```
<!-- res/anim/rotate_one.xml -->
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="2000"
    android:fromDegrees="0"
    android:toDegrees="360"
    android:pivotX="50%"
    android:pivotY="50%"
    android:interpolator="@anim/my_interpolator" />
```

# 自定义动画

> 创建自定义的动画，创建一个动画类继承Animation类，然后实现它的applyTransformation方法就可以了，不过通常情况下，还需要覆盖父类的initialize方法来实现一些初始化工作。

```
@param interpolatedTime 插值器的取值因子，这个因子是由动画当前完成的百分比和当前时间所对应的插值所计算得来的，取值范围为0到1.0。
@param t 矩阵的封装类，一般使用这个类来获得当前的矩阵对象
applyTransformation(float interpolatedTime,Transformation t) {
	
	final Matrix matrix = t.getMatrix();
	//通过matrix的各种操作来实现动画
}
```

> 通过改变获得的matrix对象，可以将动画效果实现出来，而对于matrix的变换操作，基本可以实现任何动画效果。  

> 此外,在初始化方法中对一些其他参数进行初始化  

```
@Override
public void initialize(int width,int height,int parentWidth,int parentHeight) {

	super.initialize(width,height,parentWidth,parentHeight);
	
	setDuration(2000);		//设置默认时长
	setFillAfter(true);		//动画结束后保留状态
	setInterpolatore();		//设置插值器
	...
}
```


# 视图动画与属性动画的区别

*	视图动画只能对View做动画；属性动画可以对任意对象做动画
*	当某个元素发生视图动画后，其响应事件的位置依然停留在动画前的位置，所以视图动画只能做普通的动画效果;属性动画通过调用属性的get、set方法来真实的改变了一个View的属性，所以事件响应的区域也同样发生了改变，点击动画移动后的按钮，就会响应点击事件

