Android屏幕相关概念
=============

##1 屏幕密度(dpi)  

> 屏幕密度:dot per inch,即每英寸屏幕所拥有的像素数，像素密度越大，显示画面细节就越丰富

像素密度 = 像素密度=√{（长度像素数^2+宽度像素数^2）}/ 屏幕尺寸  

> Android系统定义了几个标准dpi值  

|  密度  | ldpi | mdpi | hdpi | xhdpi | xxhdpi  
|-----  |----  |----  | ----- |  ---- | ---
| 密度值 | 120  | 160  | 240  | 320    | 480   
| 分辨率 | 240X320 | 320X480 | 480X800 | 720X1280 | 1080X1920 

##2 分辨率

> 物理屏幕上总的像素点数，如1080X1920、720X180等

##3 像素pixels(px)

##4 独立像素密度(dp)

> dp(density-independent pixel)独立像素单位  
> Android系统使用mdpi即密度值为160dpi的屏幕为标准，在这个屏幕上1px = 1dp ,hdpi中1dp = 1.5px,的到各个分辨率直接的换算比例为ldpi : mdpi : hdpi : xhdpi : xxhdpi = 3 : 4 : 6 : 8 : 12  

##5 单位转换公式

	package com.imooc.viewlayout;
	
	import android.content.Context;
	import android.util.TypedValue;
	
	public class DisplayUtil {
	
		/**
		 * px转换为dp值。保证尺寸大小不变
		 * 
		 * @param context
		 * @param pxValue
		 * @return
		 */
		public static int px2dip(Context context, float pxValue) {
	
			final float scale = context.getResources().getDisplayMetrics().density;
			return (int) (pxValue / scale + 0.5f);
		}
	
		/**
		 * dp转成px值，以保证尺寸大小不变
		 * 
		 * @param context
		 * @param dipValue
		 * @return
		 */
		public static int dip2px(Context context, float dipValue) {
	
			final float scale = context.getResources().getDisplayMetrics().density;
			return (int) (dipValue * scale + 0.5f);
		}
	
		/**
		 * px转成sp值，以保证文字大小不变
		 * 
		 * @param context
		 * @param pxValue
		 * @return
		 */
		public static int px2sp(Context context, float pxValue) {
	
			final float scale = context.getResources().getDisplayMetrics().scaledDensity;
			return (int) (pxValue / scale + 0.5f);
	
		}
	
		/**
		 * sp转成px值，以保证文字大小不变
		 * 
		 * @param context
		 * @param pxValue
		 * @return
		 */
		public static int sp2px(Context context, float pxValue) {
	
			final float scale = context.getResources().getDisplayMetrics().scaledDensity;
			return (int) (pxValue * scale + 0.5f);
	
		}
	
		/**
		 * 系统提供的计算dp转px值方法
		 * @param context
		 * @param dp
		 * @return
		 */
		public static int dp2px(Context context, int dp) {
	
			return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dp,
					context.getResources().getDisplayMetrics());
	
		}
		
		/**
		 * 系统提供的计算sp转px值方法
		 * @param context
		 * @param dp
		 * @return
		 */
		public static int sp2px(Context context, int sp) {
	
			return (int) TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP, sp,
					context.getResources().getDisplayMetrics());
	
		}
	}
