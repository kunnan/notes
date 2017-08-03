**Android显示Bitmap**


# 读取Bitmap尺寸和类型  

>设置options.inJustDecodeBounds = true;

```Java
Bitmap bitmap =  null;
BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = true;  
//获取图片的尺寸,注意此处返回的值bitmap = null
bitmap = BitmapFactory.decodeResource(getResources(), R.id.myimage, options);
int imageHeight = options.outHeight;
int imageWidth = options.outWidth;
String imageType = options.outMimeType;
```

# 加载缩放的Bitmap到内存

## 计算缩放比  

```Java
public static int calculateInSampleSize(
            BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;

    if (height > reqHeight || width > reqWidth) {

        final int halfHeight = height / 2;
        final int halfWidth = width / 2;

        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // 直到宽高小于所要求的宽高
        while ((halfHeight / inSampleSize) > reqHeight
                && (halfWidth / inSampleSize) > reqWidth) {
            inSampleSize *= 2;
        }
    }

    return inSampleSize;
}  
```

## 重新获取Bitmap  

```Java
	 public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
	        int reqWidth, int reqHeight) {
	
	    // 第一次 inJustDecodeBounds=true 
	    final BitmapFactory.Options options = new BitmapFactory.Options();
	    options.inJustDecodeBounds = true;
	    BitmapFactory.decodeResource(res, resId, options);
	
	    // Calculate inSampleSize
	    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
	
	    // 第二次 options.inJustDecodeBounds = false
	    options.inJustDecodeBounds = false;
	    return BitmapFactory.decodeResource(res, resId, options);
	}
```

## 显示Bitmap  
	
```Java
mImageView.setImageBitmap(
    decodeSampledBitmapFromResource(getResources(), R.id.myimage, 100, 100));
```

