Android绘图机制与处理技巧  
=======

*	[1. Android绘图](#draw)
*	[2. Android图像处理](#process)
*	[3. SurfaceView的使用](#surfaceview)

<h2 id="draw">1. Android绘图</h2>

*	[1.1 2D绘图](#draw2D)
*	[1.2 Android XML绘图](#drawxml)
*	[1.3 Android绘图技巧](#drawskill)

<h3 id="draw2D">1.1 2D绘图</h3>

> 系统通过提供的Canvas对象来提供绘图方法。它提供了各种绘制图像的API，如drawPoint（点）、drawLine（线）、drawRect（矩形）、drawVertices（多边形）、drawArc（弧）、drawCircle（圆），drawPath（路径）、drawText（文本）、drawOval（椭圆）、drawBitmap（贴图）等等。  

> Paint画笔也是绘图中一个非常重要的元素.  

*	`setAntiAlias()`	　　　//设置画笔的锯齿效果
*	`setColor()`	　　　　//设置画笔的颜色
*	`setARGB()`	　　　　　//设置画笔的A、R、G、B值  
*	`setAlpha()`	　　　　//设置画笔的Alpha值
*	`setTextSize()`	　　　//设置字体的尺寸
*	`setStyle()`	　　　　//设置画笔的风格(空心或者实心)
*	`setStrokeWidth()`	　//设置空心边框的宽度  

>Canvas家族成员  

*	DrawPoint，绘制点  

		canvas.drawPoint(x,y,paint); 
		//绘制多个点 
		canvas.drawPoints(pts,paint);

*	DrawLine，绘制直线  

		canvas.drawLine(startX,startY,endX,endY,paint);  

*	DrawLines，绘制多条直线  

		float[] pts = {startX1,startX2,endX1,endX2,...,startXn,startYn,endXn,endYn};
		canvas.drawLines(pts,point);

*	DrawRect，绘制矩形  

		canvas.drawRect(left,top,right,bottom,paint);  

*	DrawRoundRect,绘制圆角矩形  

		canvas.drawRoundRect(left,top,right,bottom,radiusX,radiusY,paint);

*	DrawCircle，绘制圆  

		canvas.drawCircle(circleX,circleY,radius,paint);

*	DrawArc，绘制弧形、扇形  

		paint.setStyle(Paint.STROKE);
		canvas.drawArc(left,top,right,bottom,startAngle,sweepAngle,(boolean类型)useCenter,paint);
		//Panit.Style和useCenter属性的组合
		//1.绘制扇形
		Panit.Style.STROKE + useCenter(true);
		
		//2.绘制弧形
		Panit.Style.STROKE + useCenter(false);
		
		//3.绘制实心扇形
		Panit.Style.FILL + useCenter(true);
		
		//4.绘制实心弧形
		Panit.Style.FILL + useCenter(false); 

*	DrawOval，绘制椭圆  

		//通过椭圆的外接矩形来绘制椭圆  
		canvas.drawOval(left,top,right,bottom,paint);  

*	DrawText，绘制文本  

		canvas.drawText(text,startX,startY,paint);

*	DrawPosText，在指定位置绘制文本  

		canvas.drawPosText(text,new float[]{x1,x2,x2,y2,...xn,yn},paint); 

*	DrawPath，绘制路径

		Path path = new Path();
		path.moveTo(50,50);
		path.lineTo(100,100);
		path.lineTo(100,300);
		path.lineTo(300,50);
		canvas.drawPath(path,paint);


<h3 id="drawxml">1.2 Android XML绘图</h3>

####1.2.1 Bitmap

	<?xml version="1.0" encoding="utf-8"?>
	<bitmap xmlns:android="http://schema.android.com/apk/res/android
		android:src="@drawable/ic_lancher" />

####1.2.2 Shape 

> Shape可以说是XML绘图的精华所在。Shape功能十分强大，无论是扁平化、拟物化还是渐变，它都能绘制.

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schema.android.com/apk/res/android 
	//默认为rectangle
	android:shape=["rectangle"|"oval"|"line"|"ring"] > 
	<corners 　//当shape="rectangle"时使用 
		android:radius="1dp"
		android:topLeftRadius="0.5dp"
		android:topRightRadius="0.5dp"
		android:bottomLeftRadius="0.5dp"
		android:bottomRightRadius="0.5dp" />

	<gradient  //渐变  
		android:angle="0.5dp"
		android:centerX="0.5dp"
		android:centerY="0.5dp"
		android:centerColor="#ffffff"
		android:endColor="@color/"
		android:gradientRadius="0.5dp"
		android:startColor="@color/"
		android:type=["linear"|"radial"|"sweep"]
		android:useLevel=["true" | "false"] />

	<padding  
		android:left="20dp"
		android:top="20dp"
		android:right="20dp"
		android:bottom="20dp" />

	<size  //指定大小，一般用imageview配合scaleType属性使用
		android:width="20dp"
		android:height="20dp" />

	<solid	//填充颜色
		android:color="@color/" />
	
	<stroke	//指定边框	
		android:width="20dp"
		android:color="@color/"
		//虚线宽度
		android:dashWidth="5dp"
		//虚线间隔宽度
		android:dashGap="2dp"		

</shape>
```

####1.2.3 layer

> 通过layer、layer-list实现图层效果，图片会依次叠加  

```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schema.android.com/apk/res/android >

<!--图片1-->
<item
	android:drawable="@drawable/ic_launcher" />

<!--图片2-->
<item
	android:drawable="@drawable/ic_launcher" 
	android:left="10dp"
	android:top="10dp"
	android:right="10dp"
	android:bottom="10dp" />

</layer-list>
```

####1.2.4 Selector

> Selector的作用在于实现静态绘图中的事件反馈，通过给不同的事件设置不同的图像，从而在程序中根据用户输入，返回不同的效果。 

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schema.android.com/apk/res/android >

<!--默认时的背景图片-->
<item android:drawable="@drawable/X1" />

<!--没有焦点时的背景图片-->
<item android:state_window_focused="false"
	  android:drawable="@drawable/X2 />

<!--非触摸模式下获得焦点并单击时的背景图片-->
<item android:state_focused="true"
	  android:state_pressed="true"
	  android:drawable="@drawable/X3" />

<!--触摸模式下获得焦点并单击时的背景图片-->
<item android:state_focused="false"
	  android:state_pressed="true"
	  android:drawable="@drawable/X4" />

<!--选中时的背景图片-->
<item android:state_selected="true"
	  android:drawable="@drawable/X5" />

<!--获得焦点时的图片背景-->
<item android:state_focused="true"
	  android:drawable="@drawable/X6 />

</selector>
```

> Selector也可以使用Shape作为它的Item，实现具有点击反馈效果的Selector  

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schema.android.com/apk/res/android >

<item android:state_pressed="true" >
	<shape android:shape="rectangle" >

<!--填充的颜色-->
<solid android:color="#00ff00" />

<!--设置按钮的四个角为弧形-->
<corners android:radius="5dp" />

<padding
	android:bottom="10dp"
	android:left="10dp"
	android:right="10dp"
	android:top="10dp" />

	</shape>
</item>

<item >
	<shape android:shape="rectangle" >

<!--填充的颜色-->
<solid android:color="#00ffff" />

<!--设置按钮的四个角为弧形-->
<corners android:radius="5dp" />

<padding
	android:bottom="10dp"
	android:left="10dp"
	android:right="10dp"
	android:top="10dp" />

	</shape>
</item>

</selector>
```

<h3 id="drawskill">1.3 Android绘图技巧</h3>

####1.3.1 Canvas

*	**`Canvas.save()`**

> `Canvas.save()`这个方法，从字面上可以理解为保存画布。它的作用就是将之前的所有已绘制图像保存起来，让后续的操作就好像在一个新的图层上操作一样。

*	**`Canvas.restore()`**

> `Canvas.restore()`这个方法，可以理解为合并图层操作。它的作用是将我们在`sava()`之后绘制的所有图像与`sava()`之前的图像进行合并。

*	**`Canvas.translate()`**

> `Canvas.translate()`这个方法，可以理解为坐标系的平移。默认绘制坐标零点位于屏幕左上角，那么早调用`translate(x,y)`方法之后，则将原点(0,0)移动到了(x,y)。之后的所有绘图操作都将以(x,y)为原点执行。

*	**`Canvas.rotate()`**

> `Canvas.rotate()`将坐标系旋转了一定的角度。

####1.3.2 Layer图层

> Android通过调用`saveLayer()`、`saveLayerAlpha()`方法将一个图层入栈，使用`restore()`、`restoreToCount()`方法将一个图层出栈。入栈的时候，后面所有的操作都发生在这个图层上，而出栈的时候，则会把图像绘制到上层Canvas上。


<h2 id="process">2. Android图像处理</h2>



<h2 id="surfaceview">3. SurfaceView的使用</h2>

###3.1 View与SurfaceView的区别  

View通过刷新来重绘视图，Android系统通过发出VSYNC信号来进行屏幕的重绘，刷新的间隔时间为16ms。如果在16ms内View完成了你所需要执行的所有操作，那么用户在视觉上，就不会产生卡顿的感觉；而如果执行的操作逻辑太多，特别是需要频繁刷新的界面是哪个，例如游戏界面，那么就会不断阻塞主线程，从而导致画面卡顿。很多时候，自定义View的Log中经常看见如下警告`Skipped 47 frames!This application may be doing too much work on its main thread`。这些警告的产生，很多情况下就是因为在绘制过程中，处理太多逻辑造成的。  

Android系统提供了SurfaceView组件来解决这个问题，它与View的区别主要体现在: 

*	View主要适应于主动更新的情况下，而SurfaceView主要适用于被动更新，例如频繁的刷新。
*	View在主线程中对画面进行刷新，而SurfaceView通常会通过一个子线程来进行页面的刷新。
*	View在绘图时没有使用双缓冲机制，而SurfaceView在底层实现机制中就已经实现了双缓冲机制。


总之，如果你的自定义View需要频繁刷新，或者刷新时数据处理量比较大，那么你就可以考虑使用SurfaceView来取代View了。

###3.2 SurfaceView的使用

SurfaceView的模板代码

```
package com.imooc.surfaceviewtest;

import android.content.Context;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Path;
import android.util.AttributeSet;
import android.view.SurfaceHolder;
import android.view.SurfaceView;

public class SinView extends SurfaceView
        implements SurfaceHolder.Callback, Runnable {

    private SurfaceHolder mHolder;
    //用于绘图的Canvas
    private Canvas mCanvas;
    //控制子线程
    private boolean mIsDrawing;
    private int x = 0;
    private int y = 0;
    private Path mPath;
    private Paint mPaint;

    public SinView(Context context) {
        super(context);
        initView();
    }

    public SinView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public SinView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        initView();
    }

    private void initView() {
        mHolder = getHolder();
        mHolder.addCallback(this);
        setFocusable(true);
        setFocusableInTouchMode(true);
        this.setKeepScreenOn(true);
        mPath = new Path();
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.RED);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(10);
        //设置画笔的笔触风格
        mPaint.setStrokeCap(Paint.Cap.ROUND);
        //设置接合处的形态
        mPaint.setStrokeJoin(Paint.Join.ROUND);
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        mPath.moveTo(0, 400);
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder,
                               int format, int width, int height) {
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }

    @Override
    public void run() {
        while (mIsDrawing) {
            draw();
            x += 1;
            y = (int) (100*Math.sin(x * 2 * Math.PI / 180) + 400);
            mPath.lineTo(x, y);
        }
    }

    private void draw() {
        try {
            //获取当前的Canvas绘画对象(继续上次的Canvas对象，而不是一个新的对象)
            mCanvas = mHolder.lockCanvas();
            // SurfaceView背景
            mCanvas.drawColor(Color.WHITE);
            mCanvas.drawPath(mPath, mPaint);
        } catch (Exception e) {
        } finally {
            if (mCanvas != null)
                mHolder.unlockCanvasAndPost(mCanvas);
        }
    }
}
```

