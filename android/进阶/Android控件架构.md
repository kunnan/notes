Android控件架构与自定义控件
======

*	[1. Android控件架构](#structure)
*	[2. View](#view)
	*	[2.1 View的测量](#viewmeasure)
	*	[2.2 View的绘制](#viewdraw)
	*	[2.3 自定义View](#viewcustom)
*	[3. ViewGroup](#viewgroup)
	*	[3.1 ViewGroup的测量](#groupmeasure)
	*	[3.2 ViewGroup的绘制](#groupdraw)
	*	[3.3 自定义ViewGroup](#groupcustom)


<h2 id="structure">1. Android控件架构</h2>

<h2 id="view">2. View</h2>

<h3 id="viewmeasure">2.1 View的测量</h3>

>测量的三种模式  

1. EXACTLY：精确值模式  
 当我们将控件的layout\_width或layout\_height属性指定为具体的数值时，或指定为match\_parent属性时，系统使用的是EXACTLY模式。

2. AT_MOST：最大值模式  
 当我们将控件的layout\_width或layout\_height属性指定为wrap\_content时,系统使用的是AT\_MOST模式  

3. UNSPECIFIED：  
 此模式不指定其大小测量模式，View想多大就多大，通常情况下载绘制自定义View时才会使用  

**Note:: View类默认的onMeasure()方法只支持EXACTLY模式，而如果要让自定义View支持wrap\_content属性时，那么必须重写onMeasure()方法来指定wrap\_content的大小**

>测量工具类:MeasureSpec  

1. 获取View的测量模式和View想要绘制的大小  

```
	//测量模式
    int specMode = MeasureSpec.getMode(measureSpec);  
	//想要绘制的大小
    int specSize = MeasureSpec.getSize(measureSpec);  
```

2. 判断测量的模式，给出不同的测量值  

```
private int measureWidth(int measureSpec) {
    int result = 0;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    if (specMode == MeasureSpec.EXACTLY) {
        result = specSize;
    } else {
        result = 200;
        if (specMode == MeasureSpec.AT_MOST) {
            result = Math.min(result, specSize);
        }
    }
    return result;
}

private int measureHeight(int measureSpec) {
    int result = 0;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    if (specMode == MeasureSpec.EXACTLY) {
        result = specSize;
    } else {
        result = 200;
        if (specMode == MeasureSpec.AT_MOST) {
            result = Math.min(result, specSize);
        }
    }
    return result;
}
```

3. 重写onMeasure()方法

```
@Override
protected void onMeasure(int widthMeasureSpec,int heightMeasureSpec) {
    setMeasuredDimension(
            measureWidth(widthMeasureSpec),
            measureHeight(heightMeasureSpec));
}
```

<h3 id="viewdraw">2.2 View的绘制</h3>



<h3 id="viewcustom">2.3 自定义View</h3>

###2.3.1 现有控件的扩展 



###2.3.2 创建复合控件  

> 创建XMl布局  

> 资源文件中定义自定义属性项  

```
<declare-styleable name="aboutItemView">
    <attr name="left_drawable" format="reference" />
    <attr name="left_text" format="reference" />
    <attr name="left_textColor" format="reference" />
    <attr name="right_drawable" format="reference" />
    <attr name="right_text" format="reference" />
    <attr name="right_textColor" format="reference" />
</declare-styleable>
```

> 继承自定义布局类型  

```
public class AboutItemView extends LinearLayout {
	
	private TextView mLeftView;
	private TextView mRightView;
	private ImageView mLeftImage;
	
	private int mLeftDrawable;
	private int mLeftText;
	private int mLeftTextColor;
	private int mRightDrawable;
	private int mRightText;
	private int mRightTextColor;
	

	public AboutItemView(Context context, AttributeSet attrs, int defStyle) {
		super(context, attrs, defStyle);
		// TODO Auto-generated constructor stub
		initView(context);
	}

	//布局文件时调用,引用自定义属性
	public AboutItemView(Context context, AttributeSet attrs) {
		super(context, attrs);
		// TODO Auto-generated constructor stub
		initView(context);
		
		TypedArray arributes = context.obtainStyledAttributes(attrs, R.styleable.aboutItemView);
		mLeftDrawable = arributes.getResourceId(R.styleable.aboutItemView_left_drawable, R.drawable.icon_feedback);
		mLeftText = arributes.getResourceId(R.styleable.aboutItemView_left_text, R.string.introduction_suggestion_feedback);
		mLeftTextColor = arributes.getResourceId(R.styleable.aboutItemView_left_textColor, R.color.dark);
		mRightDrawable = arributes.getResourceId(R.styleable.aboutItemView_right_drawable, R.drawable.icon_new_list);
		mRightText = arributes.getResourceId(R.styleable.aboutItemView_right_text, R.string.string_null);
		mRightTextColor = arributes.getResourceId(R.styleable.aboutItemView_right_textColor, R.color.dark);
		
//		Drawable drawableLeft = getResources().getDrawable(mLeftDrawable);
		mLeftImage.setImageResource(mLeftDrawable);
		
		mLeftView.setText(mLeftText);
		mLeftView.setTextColor(mLeftTextColor);
		
		Drawable drawableRight = getResources().getDrawable(mRightDrawable);
		drawableRight.setBounds(0, 0, drawableRight.getMinimumWidth(), drawableRight.getMinimumHeight());
		mRightView.setCompoundDrawables(null, null, drawableRight, null);
		mRightView.setText(mRightText);
		mRightView.setTextColor(mRightTextColor);
		
		arributes.recycle();
	}
	//new类型时调用
	public AboutItemView(Context context) {
		super(context);
		// TODO Auto-generated constructor stub
		initView(context);
	}
	
	
	private void initView(Context context){
		
		View.inflate(context, R.layout.item_about, this);
		mLeftImage = (ImageView) this.findViewById(R.id.iv_left_image);
		mLeftView = (TextView) this.findViewById(R.id.tv_left_desc);
		mRightView = (TextView) this.findViewById(R.id.tv_right_desc);
		
		mLeftView.setCompoundDrawables(null, null, null, null);
		
	}
}  
```


>使用控件  

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:about="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/view_setting_bg" >

    <com.gokemicro.dvr.ui.AboutItemView
        android:id="@+id/tv_introduction_setting"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/show1"
		//自定义属性
        about:left_drawable="@drawable/icon_new_setting"
        about:left_text="@string/introduction_setting" />

</RelativeLayout>
```

###2.3.3 实现全新的控件



<h2 id="viewgroup">3. ViewGroup</h2>


<h3 id="groupmeasure">3.1 ViewGroup的测量</h3>


<h3 id="groupdraw">3.2 ViewGroup的绘制</h3>


<h3 id="groupcustom">3.3 自定义ViewGroup</h3>

