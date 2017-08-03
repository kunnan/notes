**自定义View1-2 Paint详解**

[原文转载自扔物线HenCoder Android 开发进阶: 自定义 View 1-2 Paint 详解](https://zhuanlan.zhihu.com/p/27919855)

[TOC]

Paint 的 API 大致可以分为 4 类：

* 颜色
* 效果
* drawText() 相关
* 初始化

# 目录

# 颜色

Canvas 绘制的内容，有三层对颜色的处理：

![](https://pic2.zhimg.com/v2-f19702f4861af65de963627e97f9bde9_b.png)

## 基本颜色

像素的基本颜色，根据绘制内容的不同而有不同的控制方式： Canvas 的颜色填充类方法 drawColor/RGB/ARGB() 的颜色，是直接写在方法的参数里，通过参数来设置的（上期讲过了）； drawBitmap() 的颜色，是直接由 Bitmap 对象来提供的（上期也讲过了）；除此之外，是图形和文字的绘制，它们的颜色就需要使用 paint 参数来额外设置了（下面要讲的）。

![](https://pic4.zhimg.com/v2-3a32c933b56cad4527a4c1f4f8934f37_b.png)

Paint 设置颜色的方法有两种：一种是直接用 Paint.setColor/ARGB() 来设置颜色，另一种是使用 Shader 来指定着色方案。

### 直接设置颜色

#### setColor(int color)

```Java
paint.setColor(Color.parseColor("#009688"));  
canvas.drawRect(30, 30, 230, 180, paint);

paint.setColor(Color.parseColor("#FF9800"));  
canvas.drawLine(300, 30, 450, 180, paint);

paint.setColor(Color.parseColor("#E91E63"));  
canvas.drawText("HenCoder", 500, 130, paint); 
```
![](https://pic4.zhimg.com/v2-1db4b9b8606f54f3bed0a79ed2b0b43b_b.png)

>setColor() 对应的 get 方法是 getColor()

#### setARGB(int a, int r, int g, int b)

其实和 setColor(color) 都是一样一样儿的，只是它的参数用的是更直接的三原色与透明度的值。实际运用中，setColor() 和 setARGB() 哪个方便和顺手用哪个吧。

```Java
paint.setARGB(100, 255, 0, 0);  
canvas.drawRect(0, 0, 200, 200, paint);  
paint.setARGB(100, 0, 0, 0);  
canvas.drawLine(0, 0, 200, 200, paint);  
```

### setShader(Shader shader)设置 Shader

除了直接设置颜色， Paint 还可以使用 Shader 。

Shader 它的中文叫做「着色器」，也是用于设置绘制颜色的。「着色器」不是 Android 独有的，它是图形领域里一个通用的概念，它和直接设置颜色的区别是，着色器设置的是一个颜色方案，或者说是一套着色规则。当设置了 Shader 之后，Paint 在绘制图形和文字时就不使用 setColor/ARGB() 设置的颜色了，而是使用 Shader 的方案中的颜色。

在 Android 的绘制里使用 Shader ，并不直接用 Shader 这个类，而是用它的几个子类。具体来讲有 LinearGradient RadialGradient SweepGradient BitmapShaderComposeShader 这么几个：

#### LinearGradient 线性渐变

设置两个点和两种颜色，以这两个点作为端点，使用两种颜色的渐变来绘制颜色。就像这样：

```Java
Shader shader = new LinearGradient(100, 100, 500, 500, Color.parseColor("#E91E63"),  
        Color.parseColor("#2196F3"), Shader.TileMode.CLAMP);
paint.setShader(shader);
...
canvas.drawCircle(300, 300, 200, paint);
```
![](https://pic2.zhimg.com/v2-8298a403a56ae8658a6f06e1e116e08d_b.png)

>设置了 Shader 之后，绘制出了渐变颜色的圆。（其他形状以及文字都可以这样设置颜色） 注意：在设置了 Shader 的情况下， Paint.setColor/ARGB() 所设置的颜色就不再起作用。

**构造方法：** LinearGradient(float x0, float y0, float x1, float y1, int color0, int color1, Shader.TileMode tile)。

*  x0 y0 x1 y1：渐变的两个端点的位置 
* color0 color1 是端点的颜色 
* tile：端点范围之外的着色规则，类型是 TileMode。TileMode 一共有 3 个值可选： CLAMP, MIRROR 和 REPEAT。CLAMP 会在端点之外延续端点处的颜色；MIRROR 是镜像模式；REPEAT 是重复模式。

CLAMP:

![](https://pic1.zhimg.com/v2-54dc84de2008f029170316cf440147a0_b.png)

MIRROR:

![](https://pic4.zhimg.com/v2-8438ec596852b125762392dc5d902a5f_b.png)

REPEAT:

![](https://pic4.zhimg.com/v2-6c04cf4509ecf7cd1d79374a84ea70c7_b.png)

#### RadialGradient 辐射渐变

辐射渐变很好理解，就是从中心向周围辐射状的渐变。大概像这样：

```Java
Shader shader = new RadialGradient(300, 300, 200, Color.parseColor("#E91E63"),  
        Color.parseColor("#2196F3"), Shader.TileMode.CLAMP);
paint.setShader(shader);

...

canvas.drawCircle(300, 300, 200, paint);
```

![](https://pic2.zhimg.com/v2-a42296033b1d17ff527b28c45270df79_b.png)

**构造方法：** RadialGradient(float centerX, float centerY, float radius, int centerColor, int edgeColor, TileMode tileMode)。

参数： centerX centerY：辐射中心的坐标 radius：辐射半径 centerColor：辐射中心的颜色 edgeColor：辐射边缘的颜色 tileMode：辐射范围之外的着色模式。

CLAMP:

![](https://pic4.zhimg.com/v2-4e71f6e6c30bbb1d9d401af549efbc0b_b.png)

MIRROR:

![](https://pic3.zhimg.com/v2-ed3166ca556db5b6af2344fd1a083802_b.png)

REPEAT:

![](https://pic1.zhimg.com/v2-9cfac936f77550326ca5498954def9d8_b.png)

#### SweepGradient 扫描渐变

```Java
Shader shader = new SweepGradient(300, 300, Color.parseColor("#E91E63"),  
        Color.parseColor("#2196F3"));
paint.setShader(shader);

...

canvas.drawCircle(300, 300, 200, paint); 
```
![](https://pic3.zhimg.com/v2-f774fdcb428fd3646521dbd3cdb97686_b.png)

**构造方法：** SweepGradient(float cx, float cy, int color0, int color1)

* cx cy ：扫描的中心 
* color0：扫描的起始颜色 
* color1：扫描的终止颜色

#### BitmapShader

用 Bitmap 来着色（终于不是渐变了）。其实也就是用 Bitmap 的像素来作为图形或文字的填充。

```Java
Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.batman);  
Shader shader = new BitmapShader(bitmap, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);  
paint.setShader(shader);

...

canvas.drawCircle(300, 300, 200, paint);
```
![](https://pic4.zhimg.com/v2-be6cb4b4d60327dd380fad09db174aaf_b.png)

>嗯，看着跟 Canvas.drawBitmap() 好像啊？事实上也是一样的效果。如果你想绘制圆形的 Bitmap，就别用 drawBitmap() 了，改用 drawCircle() + BitmapShader 就可以了（其他形状同理）。

**构造方法：** BitmapShader(Bitmap bitmap, Shader.TileMode tileX, Shader.TileMode tileY)

* bitmap：用来做模板的 Bitmap 对象 
* tileX：横向的 TileMode 
* tileY：纵向的 TileMode。

CLAMP:

![](https://pic2.zhimg.com/v2-10938ddc0555774d8979f63dba70daf9_b.png)

MIRROR:

![](https://pic4.zhimg.com/v2-21b0abfa57d7fee3bf10d24387f3f7fb_b.png)

REPEAT:

![](https://pic2.zhimg.com/v2-f001a242a16c7578e03de823644662ad_b.png)

#### ComposeShader 混合着色器

所谓混合，就是把两个 Shader 一起使用。

```Java
// 第一个 Shader：头像的 Bitmap
Bitmap bitmap1 = BitmapFactory.decodeResource(getResources(), R.drawable.batman);  
Shader shader1 = new BitmapShader(bitmap1, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);

// 第二个 Shader：从上到下的线性渐变（由透明到黑色）
Bitmap bitmap2 = BitmapFactory.decodeResource(getResources(), R.drawable.batman_logo);  
Shader shader2 = new BitmapShader(bitmap2, Shader.TileMode.CLAMP, Shader.TileMode.CLAMP);

// ComposeShader：结合两个 Shader
Shader shader = new ComposeShader(shader1, shader2, PorterDuff.Mode.SRC_OVER);  
paint.setShader(shader);

...

canvas.drawCircle(300, 300, 300, paint); 
```
>注意：上面这段代码中我使用了两个 BitmapShader 来作为 ComposeShader() 的参数，而 ComposeShader() 在硬件加速下是不支持两个相同类型的 Shader 的，所以这里也需要关闭硬件加速才能看到效果。

![](https://pic3.zhimg.com/v2-1653d1c9778ba1d59ce27043b4d2e0a2_b.png)

**构造方法：**ComposeShader(Shader shaderA, Shader shaderB, PorterDuff.Mode mode)

* shaderA, shaderB：两个相继使用的 Shader 
* mode: 两个 Shader 的叠加模式，即 shaderA 和 shaderB 应该怎样共同绘制。它的类型是 PorterDuff.Mode 。

#### PorterDuff.Mode

>PorterDuff.Mode 是用来指定两个图像共同绘制时的颜色策略的。它是一个 enum，不同的 Mode 可以指定不同的策略。「颜色策略」的意思，就是说把源图像绘制到目标图像处时应该怎样确定二者结合后的颜色，而对于 `ComposeShader(shaderA, shaderB, mode)` 这个具体的方法，就是指应该怎样把 shaderB 绘制在 shaderA 上来得到一个结合后的 Shader。 没有听说过 PorterDuff.Mode 的人，看到这里很可能依然会一头雾水：「什么怎么结合？就……两个图像一叠加，结合呗？还能怎么结合？」你还别说，还真的是有很多种策略来结合。 最符合直觉的结合策略，就是我在上面这个例子中使用的 Mode: SRC_OVER。它的算法非常直观：就像上面图中的那样，把源图像直接铺在目标图像上。不过，除了这种，其实还有一些其他的结合方式。例如如果我把上面例子中的参数 mode 改为 PorterDuff.Mode.DST_OUT，就会变成挖空效果：

![](https://pic2.zhimg.com/v2-086e22dcb7cd960005f1e39bbb45ac15_b.png)

>而如果再把 mode 改为 PorterDuff.Mode.DST_IN，就会变成蒙版抠图效果：

![](https://pic1.zhimg.com/v2-c994ef54acbeb68f579926a4badf4258_b.png)

具体来说， PorterDuff.Mode 一共有 17 个，可以分为两类：

* Alpha 合成 (Alpha Compositing)
* 混合 (Blending)

第一类，Alpha 合成，其实就是 「PorterDuff」 这个词所指代的算法。 「PorterDuff」 并不是一个具有实际意义的词组，而是两个人的名字（准确讲是姓）。这两个人当年共同发表了一篇论文，描述了 12 种将两个图像共同绘制的操作（即算法）。而这篇论文所论述的操作，都是关于 Alpha 通道（也就是我们通俗理解的「透明度」）的计算的，后来人们就把这类计算称为Alpha 合成 ( Alpha Compositing ) 。

![](https://pic4.zhimg.com/v2-97beb4e4511d7680d4adb49a2658ea8f_b.png)

Alpha合成:

![](https://pic4.zhimg.com/v2-2268018797457465681f58c5b21e4793_b.png)

第二类，混合，也就是 Photoshop 等制图软件里都有的那些混合模式（multiplydarken lighten 之类的）。这一类操作的是颜色本身而不是 Alpha 通道，并不属于 Alpha 合成，所以和 Porter 与 Duff 这两个人也没什么关系，不过为了使用的方便，它们同样也被 Google 加进了 PorterDuff.Mode 里。

![](https://pic4.zhimg.com/v2-de213af5d5c256a60af7f2e4f524ab43_b.png)

>**结论** 从效果图可以看出，Alpha 合成类的效果都比较直观，基本上可以使用简单的口头表达来描述它们的算法（起码对于不透明的源图像和目标图像来说是可以的），例如 SRC_OVER表示「二者都绘制，但要源图像放在目标图像的上面」，DST_IN 表示「只绘制目标图像，并且只绘制它和源图像重合的区域」。 而混合类的效果就相对抽象一些，只从效果图不太能看得出它们的着色算法，更看不出来它们有什么用。不过没关系，你如果拿着这些名词去问你司的设计师，他们八成都能给你说出来个 123。 所以对于这些 Mode，**正确的做法是：对于 Alpha 合成类的操作，掌握他们，并在实际开发中灵活运用；而对于混合类的，你只要把它们的名字记住就好了，这样当某一天设计师告诉你「我要做这种混合效果」的时候，你可以马上知道自己能不能做，怎么做。**

## 设置ColorFilter

ColorFilter 这个类，它的名字已经足够解释它的作用：为绘制设置颜色过滤。颜色过滤的意思，就是为绘制的内容设置一个统一的过滤策略，然后 Canvas.drawXXX() 方法会对每个像素都进行过滤后再绘制出来。举几个现实中比较常见的颜色过滤的例子

>有色光照射

![](https://pic1.zhimg.com/v2-58067d6a18818f93ee0b61e73a59c25c_b.png)

>有色玻璃透视

![](https://pic1.zhimg.com/v2-e5f5c7fbdaa23a85f9e3a1cc6b4a5ad0_b.png)

>胶卷

![](https://pic1.zhimg.com/v2-0f38b494984269622d9fc9ad3480f6e8_b.png)

在 Paint 里设置 ColorFilter ，使用的是 Paint.setColorFilter(ColorFilter filter)方法。 ColorFilter 并不直接使用，而是使用它的子类。它共有三个子类：LightingColorFilter PorterDuffColorFilter 和 ColorMatrixColorFilter。

### LightingColorFilter

LightingColorFilter 是用来模拟简单的光照效果的。
 
LightingColorFilter 的构造方法是 LightingColorFilter(int mul, int add) ，参数里的 mul 和 add 都是和颜色值格式相同的 int 值，其中 mul 用来和目标像素相乘，add 用来和目标像素相加：
 
```Java
R' = R * mul.R / 0xff + add.R  
G' = G * mul.G / 0xff + add.G  
B' = B * mul.B / 0xff + add.B  
```
一个「保持原样」的「基本 LightingColorFilter 」，mul 为 0xffffff，add 为 0x000000（也就是0），那么对于一个像素，它的计算过程就是：

```Java
R' = R * 0xff / 0xff + 0x0 = R // R' = R  
G' = G * 0xff / 0xff + 0x0 = G // G' = G  
B' = B * 0xff / 0xff + 0x0 = B // B' = B  
```
基于这个「基本 LightingColorFilter 」，你就可以修改一下做出其他的 filter。比如，如果你想去掉原像素中的红色，可以把它的 mul 改为 0x00ffff （红色部分为 0 ） ，那么它的计算过程就是：

```Java
R' = R * 0x0 / 0xff + 0x0 = 0 // 红色被移除  
G' = G * 0xff / 0xff + 0x0 = G  
B' = B * 0xff / 0xff + 0x0 = B  
```
具体效果是这样的：

```Java
ColorFilter lightingColorFilter = new LightingColorFilter(0x00ffff, 0x000000);  
paint.setColorFilter(lightingColorFilter); 
```
![](https://pic1.zhimg.com/v2-82fa14be57aa38d3b8b3e475fa35d95c_b.png)

或者，如果你想让它的绿色更亮一些，就可以把它的 add 改为 0x003000 （绿色部分为 0x30 ），那么它的计算过程就是：

```Java
R' = R * 0xff / 0xff + 0x0 = R  
G' = G * 0xff / 0xff + 0x30 = G + 0x30 // 绿色被加强  
B' = B * 0xff / 0xff + 0x0 = B  
```

```Java
ColorFilter lightingColorFilter = new LightingColorFilter(0xffffff, 0x003000);  
paint.setColorFilter(lightingColorFilter);  
```
效果是这样：

![](https://pic2.zhimg.com/v2-9f63772c4202acbf5f297f81ba202105_b.png)

### PorterDuffColorFilter

这个 PorterDuffColorFilter 的作用是使用一个指定的颜色和一种指定的 PorterDuff.Mode 来与绘制对象进行合成。它的构造方法是 PorterDuffColorFilter(int color, PorterDuff.Mode mode) 其中的 color 参数是指定的颜色， mode 参数是指定的 Mode。同样也是 PorterDuff.Mode ，不过和 ComposeShader不同的是，PorterDuffColorFilter 作为一个 ColorFilter，只能指定一种颜色作为源，而不是一个 Bitmap。

### ColorMatrixColorFilter

ColorMatrixColorFilter 使用一个 ColorMatrix 来对颜色进行处理。 ColorMatrix 这个类，内部是一个 4x5 的矩阵：

```Java
[ a, b, c, d, e,
  f, g, h, i, j,
  k, l, m, n, o,
  p, q, r, s, t ]
```

通过计算， ColorMatrix 可以把要绘制的像素进行转换。对于颜色 [R, G, B, A] ，转换算法是这样的：

```Java
R’ = a*R + b*G + c*B + d*A + e;  
G’ = f*R + g*G + h*B + i*A + j;  
B’ = k*R + l*G + m*B + n*A + o;  
A’ = p*R + q*G + r*B + s*A + t;  
```

ColorMatrix 有一些自带的方法可以做简单的转换，例如可以使用 setSaturation(float sat) 来设置饱和度；另外你也可以自己去设置它的每一个元素来对转换效果做精细调整。具体怎样设置会有怎样的效果，可以试一下程大治同学做的这个库[StyleImageView](https://github.com/chengdazhi/StyleImageView/)

![](https://pic4.zhimg.com/v2-23a997d2d789ca776b896806ba9eb5af_b.png)

以上，就是 Paint 对颜色的第二层处理：通过 setColorFilter(colorFilter) 来加工颜色。

除了基本颜色的设置（ setColor/ARGB(), setShader() ）以及基于原始颜色的过滤（ setColorFilter() ）之外，Paint 最后一层处理颜色的方法是 setXfermode(Xfermode xfermode) ，它处理的是「当颜色遇上 View」的问题。

## setXfermode(Xfermode xfermode)

严谨地讲， Xfermode 指的是你要绘制的内容和 Canvas 的目标位置的内容应该怎样结合计算出最终的颜色。但通俗地说，其实就是要你以绘制的内容作为源图像，以 View 中已有的内容作为目标图像，选取一个 PorterDuff.Mode 作为绘制内容的颜色处理方案。就像这样：

```Java
Xfermode xfermode = new PorterDuffXfermode(PorterDuff.Mode.DST_IN);
...
canvas.drawBitmap(rectBitmap, 0, 0, paint); // 画方  
paint.setXfermode(xfermode); // 设置 Xfermode  
canvas.drawBitmap(circleBitmap, 0, 0, paint); // 画圆  
paint.setXfermode(null); // 用完及时清除 Xfermode 
```
![](https://pic4.zhimg.com/v2-b39042b83bb2eb116fb2363f5951f333_b.png)

![](https://pic2.zhimg.com/v2-2824414c3a59f54ed5933b050ea623ad_b.gif)

又是 PorterDuff.Mode 。 PorterDuff.Mode 在 Paint 一共有三处 API ，它们的工作原理都一样，只是用途不同：

![](https://pic4.zhimg.com/v2-1cd2687b11e1e31dbb211b8dfe9264a7_b.png)

另外，从上面的示例代码可以看出，创建 Xfermode 的时候其实是创建的它的子类 PorterDuffXfermode。而事实上，Xfermode 也只有这一个子类。所以在设置 Xfermode 的时候不用多想，直接用 PorterDuffXfermode 吧。其实在更早的 Android 版本中，Xfermode 还有别的子类，但别的子类现在已经 deprecated 了，如今只剩下了 PorterDuffXfermode。

### Xfermode 注意事项

Xfermode 使用很简单，不过有两点需要注意：

#### 使用离屏缓冲（Off-screen Buffer）

要想使用 setXfermode() 正常绘制，必须使用离屏缓存 (Off-screen Buffer) 把内容绘制在额外的层上，再把绘制好的内容贴回 View 中。

通过使用离屏缓冲，把要绘制的内容单独绘制在缓冲层， Xfermode 的使用就不会出现奇怪的结果了。使用离屏缓冲有两种方式：

* **Canvas.saveLayer()**

saveLayer() 可以做短时的离屏缓冲。使用方法很简单，在绘制代码的前后各加一行代码，在绘制之前保存，绘制之后恢复：

```Java
int saved = canvas.saveLayer(null, null, Canvas.ALLSAVEFLAG);

canvas.drawBitmap(rectBitmap, 0, 0, paint); // 画方 
paint.setXfermode(xfermode); // 设置 Xfermode 
canvas.drawBitmap(circleBitmap, 0, 0, paint); // 画圆 
paint.setXfermode(null); // 用完及时清除 Xfermode

canvas.restoreToCount(saved);
```

* **View.setLayerType()**

View.setLayerType() 是直接把整个 View 都绘制在离屏缓冲中。 setLayerType(LAYER_TYPE_HARDWARE) 是使用 GPU 来缓冲， setLayerType(LAYER_TYPE_SOFTWARE) 是直接直接用一个 Bitmap 来缓冲。

如果没有特殊需求，可以选用第一种方法 Canvas.saveLayer() 来设置离屏缓冲，以此来获得更高的性能。更多关于离屏缓冲的信息，可以看[官方文档](https://developer.android.com/guide/topics/graphics/hardware-accel.html)中对于硬件加速的介绍

####  控制好透明区域

使用 Xfermode 来绘制的内容，除了注意使用离屏缓冲，还应该注意控制它的透明区域不要太小，要让它足够覆盖到要和它结合绘制的内容，否则得到的结果很可能不是你想要的。我用图片来具体说明一下：

![](https://pic1.zhimg.com/v2-41be7ccb3b97e6949d1a77cea4c224f4_b.png)

>如图所示，由于透明区域过小而覆盖不到的地方，将不会受到 Xfermode 的影响。

前面讲的就是 Paint 的第一类 API——关于颜色的三层设置：直接设置颜色的 API 用来给图形和文字设置颜色； setColorFilter() 用来基于颜色进行过滤处理； setXfermode() 用来处理源图像和 View 已有内容的关系。

![](https://pic2.zhimg.com/v2-f19702f4861af65de963627e97f9bde9_b.png)

# 效果

## setAntiAlias (boolean aa)

设置抗锯齿。

![](https://pic3.zhimg.com/v2-1058694b252189fc17b6e2a1f150e57a_b.png)

抗锯齿默认是关闭的，如果需要抗锯齿，需要显式地打开。另外，除了 setAntiAlias(aa)方法，打开抗锯齿还有一个更方便的方式：构造方法。创建 Paint 对象的时候，构造方法的参数里加一个 ANTI_ALIAS_FLAG 的 flag，就可以在初始化的时候就开启抗锯齿。

```Java
Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);  
```

## setStyle(Paint.Style style)

用来设置图形是线条风格还是填充风格的（也可以二者并用）：

```Java
paint.setStyle(Paint.Style.FILL); // FILL 模式，填充  
canvas.drawCircle(300, 300, 200, paint); 
```

![](https://pic4.zhimg.com/v2-1245b4b986a24a01aef186345c71e6f3_b.png)

```Java
paint.setStyle(Paint.Style.STROKE); // STROKE 模式，画线  
canvas.drawCircle(300, 300, 200, paint);
```
![](https://pic2.zhimg.com/v2-724f287e87f1360ef16a2fe8418d0099_b.png)

```Java
paint.setStyle(Paint.Style.FILL_AND_STROKE); // FILL_AND_STROKE 模式，填充 + 画线  
canvas.drawCircle(300, 300, 200, paint); 
```
![](https://pic4.zhimg.com/v2-1245b4b986a24a01aef186345c71e6f3_b.png)

## 线条形状

设置线条形状的一共有 4 个方法：setStrokeWidth(float width), setStrokeCap(Paint.Cap cap), setStrokeJoin(Paint.Join join), setStrokeMiter(float miter) 。

### setStrokeWidth(float width)

设置线条宽度。单位为像素，默认值是 0。

```Java
paint.setStyle(Paint.Style.STROKE);  
paint.setStrokeWidth(1);  
canvas.drawCircle(150, 125, 100, paint);  
paint.setStrokeWidth(5);  
canvas.drawCircle(400, 125, 100, paint);  
paint.setStrokeWidth(40);  
canvas.drawCircle(650, 125, 100, paint);  
```
![](https://pic4.zhimg.com/v2-5d22811060e8fed90446723e77cc86c3_b.png)

>线条宽度 0 和 1 的区别 默认情况下，线条宽度为 0，但你会发现，这个时候它依然能够画出线，线条的宽度为 1 像素。那么它和线条宽度为 1 有什么区别呢？ 其实这个和后面要讲的一个「几何变换」有关：你可以为 Canvas 设置 Matrix 来实现几何变换（如放大、缩小、平移、旋转），在几何变换之后 Canvas 绘制的内容就会发生相应变化，包括线条也会加粗，例如 2 像素宽度的线条在 Canvas 放大 2 倍后会被以 4 像素宽度来绘制。而当线条宽度被设置为 0 时，它的宽度就被固定为 1 像素，就算 Canvas 通过几何变换被放大，它也依然会被以 1 像素宽度来绘制。Google 在文档中把线条宽度为 0 时称作「hairline mode（发际线模式）」。

### setStrokeCap(Paint.Cap cap)

设置线头的形状。线头形状有三种：BUTT 平头、ROUND 圆头、SQUARE 方头。默认为 BUTT。

当线条的宽度是 1 像素时，这三种线头的表现是完全一致的，全是 1 个像素的点；而当线条变粗的时候，它们就会表现出不同的样子：

![](https://pic3.zhimg.com/v2-86c6e3cae25208018cbcbdb026e989ea_b.png)

虚线是额外加的，虚线左边是线的实际长度，虚线右边是线头。有了虚线作为辅助，可以清楚地看出 BUTT 和 SQUARE 的区别。

### setStrokeJoin(Paint.Join join)

设置拐角的形状。有三个值可以选择：MITER 尖角、 BEVEL 平角和 ROUND 圆角。默认为 MITER。

![](https://pic4.zhimg.com/v2-92d4b879ea4164af9ef9e5d4b8e502e7_r.png)

### setStrokeMiter(float miter)

这个方法是对于 setStrokeJoin() 的一个补充，它用于设置 MITER 型拐角的延长线的最大值。所谓「延长线的最大值」，是这么一回事：

当线条拐角为 MITER 时，拐角处的外缘需要使用延长线来补偿：

![](https://pic1.zhimg.com/v2-2cefb9c92b953bd1d24e96e8849a7ba0_b.png)

而这种补偿方案会有一个问题：如果拐角的角度太小，就有可能由于出现连接点过长的情况。比如这样：

![](https://pic3.zhimg.com/v2-9c0930526303c0b9eebe55095730c34e_b.png)

所以为了避免意料之外的过长的尖角出现， MITER 型连接点有一个额外的规则：当尖角过长时，自动改用 BEVEL 的方式来渲染连接点。例如上图的这个尖角，在默认情况下是不会出现的，而是会由于延长线过长而被转为 BEVEL 型连接点：

![](https://pic1.zhimg.com/v2-7617a538c7cedf57c117f4a596e66ad0_b.png)

至于多尖的角属于过于尖，尖到需要转为使用 BEVEL 来绘制，则是由一个属性控制的，而这个属性就是 setStrokeMiter(miter) 方法中的 miter 参数。miter 参数是对于转角长度的限制，具体来讲，是指尖角的外缘端点和内部拐角的距离与线条宽度的比。也就是下面这两个长度的比：

![](https://pic3.zhimg.com/v2-a4ae7d55023dae4ad8d896cbf08491ca_b.png)

用几何知识很容易得出这个比值的计算公式：如果拐角的大小为 θ ，那么这个比值就等于 1 / sin ( θ / 2 ) 。

这个 miter limit 的默认值是 4，对应的是一个大约 29° 的锐角：

![](https://pic3.zhimg.com/v2-aeefdc06c47a4c7d8f6dbd41e0730b7e_b.png)

>默认情况下，大于这个角的尖角会被保留，而小于这个夹角的就会被「削成平头」
>所以，这个方法虽然名叫 setStrokeMiter(miter) ，但它其实设置的是「 线条在 Join类型为 MITER 时对于 MITER 的长度限制」。它的这个名字虽然短，但却存在一定的迷惑性，如果叫 setStrokeJoinMiterLimit(limit) 就更准确了。


## 色彩优化

Paint 的色彩优化有两个方法： setDither(boolean dither) 和 setFilterBitmap(boolean filter) 。它们的作用都是让画面颜色变得更加「顺眼」，但原理和使用场景是不同的。

### setDither(boolean dither)

>设置图像的抖动。

抖动的原理和这个类似。所谓抖动（注意，它就叫抖动，不是防抖动，也不是去抖动，有些人在翻译的时候自作主张地加了一个「防」字或者「去」字，这是不对的），是指把图像从较高色彩深度（即可用的颜色数）向较低色彩深度的区域绘制时，在图像中有意地插入噪点，通过有规律地扰乱图像来让图像对于肉眼更加真实的做法。

比如向 1 位色彩深度的区域中绘制灰色，由于 1 位深度只包含黑和白两种颜色，在默认情况下，即不加抖动的时候，只能选择向上或向下选择最接近灰色的白色或黑色来绘制，那么显示出来也只能是一片白或者一片黑。而加了抖动后，就可以绘制出让肉眼识别为灰色的效果了：

![](https://pic4.zhimg.com/v2-c5167ba2ca2b2661a3af79032df0e287_b.png)

瞧，像上面这样，用黑白相间的方式来绘制，就可以骗过肉眼，让肉眼辨别为灰色了。

嗯？你说你看不出灰色，只看出黑白相间？没关系，那是因为像素颗粒太大，我把像素颗粒缩小，看到完整效果你就会发现变灰了

![](https://pic3.zhimg.com/v2-f6b64242ddca66502dcb004bc1b53fd6_b.png)

不过，抖动可不只可以用在纯色的绘制。在实际的应用场景中，抖动更多的作用是在图像降低色彩深度绘制时，避免出现大片的色带与色块。

![](https://pic2.zhimg.com/v2-b5f3d41339e313971cc24efe2aa70b65_b.png)

在 Android 里使用起来也很简单，一行代码就搞定：

```Java
paint.setDither(true);  
```
不过对于现在（2017年）而言， setDither(dither) 已经没有当年那么实用了，因为现在的 Android 版本的绘制，默认的色彩深度已经是 32 位的 ARGB_8888 ，效果已经足够清晰了。只有当你向自建的 Bitmap 中绘制，并且选择 16 位色的 ARGB_4444 或者 RGB_565 的时候，开启它才会有比较明显的效果。

### setFilterBitmap(boolean filter)

>设置是否使用双线性过滤来绘制 Bitmap 。

图像在放大绘制的时候，默认使用的是最近邻插值过滤，这种算法简单，但会出现马赛克现象；而如果开启了双线性过滤，就可以让结果图像显得更加平滑。

![](https://pic3.zhimg.com/v2-e244e354187c62c5f0622c25d99b3986_b.png)

```Java
paint.setFilterBitmap(true); 
```
加上这一行，在放大绘制 Bitmap 的时候就会使用双线性过滤了。

以上就是 Paint 的两个色彩优化的方法： setDither(dither) ，设置抖动来优化色彩深度降低时的绘制效果； setFilterBitmap(filterBitmap) ，设置双线性过滤来优化 Bitmap放大绘制的效果。

## setPathEffec路径效果

使用 PathEffect 来给图形的轮廓设置效果。对 Canvas 所有的图形绘制有效，也就是 drawLine() drawCircle() drawPath() 这些方法。大概像这样：

```Java
PathEffect pathEffect = new DashPathEffect(new float[]{10, 5}, 10);  
paint.setPathEffect(pathEffect);

...

canvas.drawCircle(300, 300, 200, paint); 
```

![](https://pic2.zhimg.com/v2-57471a798541ee2bead82b185dc34999_b.png)

下面就具体说一下 Android 中的 6 种 PathEffect。PathEffect 分为两类，单一效果的 CornerPathEffect DiscretePathEffect DashPathEffect PathDashPathEffect ，和组合效果的 SumPathEffect ComposePathEffect。

### CornerPathEffect

>把所有拐角变成圆角。
>它的构造方法 CornerPathEffect(float radius) 的参数 radius 是圆角的半径。

```Java
PathEffect pathEffect = new CornerPathEffect(20);  
paint.setPathEffect(pathEffect);

...

canvas.drawPath(path, paint); 
```
![](https://pic2.zhimg.com/v2-88a9e36c06effc63015c70287848dc55_b.png)

### DiscretePathEffect

把线条进行随机的偏离，让轮廓变得乱七八糟。乱七八糟的方式和程度由参数决定。

>DiscretePathEffect 具体的做法是，把绘制改为使用定长的线段来拼接，并且在拼接的时候对路径进行随机偏离。它的构造方法 DiscretePathEffect(float segmentLength, float deviation) 的两个参数中， segmentLength 是用来拼接的每个线段的长度， deviation 是偏离量。这两个值设置得不一样，显示效果也会不一样。

```Java
PathEffect pathEffect = new DiscretePathEffect(20,3);  
paint.setPathEffect(pathEffect);

...

canvas.drawPath(path, paint); 
```
![](https://pic4.zhimg.com/v2-7e2f4a69a6333826c0acfd568c82290f_b.png)

### DashPathEffect

>使用虚线来绘制线条。
>它的构造方法 DashPathEffect(float[] intervals, float phase) 中， 第一个参数 intervals 是一个数组，它指定了虚线的格式：数组中元素必须为偶数（最少是 2 个），按照「画线长度、空白长度、画线长度、空白长度」……的顺序排列，例如上面代码中的 20, 5, 10, 5 就表示虚线是按照「画 20 像素、空 5 像素、画 10 像素、空 5 像素」的模式来绘制；第二个参数 phase 是虚线的偏移量。

```Java
PathEffect pathEffect = new DashPathEffect(new float[]{20, 10, 5, 10}, 0);  
paint.setPathEffect(pathEffect);

...

canvas.drawPath(path, paint); 
```
![](https://pic1.zhimg.com/v2-a279eb2b2c0c149c2e1d61f949c2fe98_r.png)

### PathDashPathEffect

>它是使用一个 Path 来绘制「虚线」。

```Java
Path dashPath = ...; // 使用一个三角形来做 dash  
PathEffect pathEffect = new PathDashPathEffect(dashPath, 40, 0,  
        PathDashPathEffectStyle.TRANSLATE);
paint.setPathEffect(pathEffect);

...

canvas.drawPath(path, paint); 
```

![](https://pic4.zhimg.com/v2-4e87fdd5906a2bc12788cbd1461e4fd7_b.png)

它的构造方法 PathDashPathEffect(Path shape, float advance, float phase, PathDashPathEffect.Style style)

* shape 参数是用来绘制的 Path ； 
* advance 是两个相邻的 shape 段之间的间隔，不过注意，这个间隔是两个 shape 段的起点的间隔，而不是前一个的终点和后一个的起点的距离； 
* phase 和 DashPathEffect 中一样，是虚线的偏移；最后一个参数 
* style，是用来指定拐弯改变的时候 shape 的转换方式。style 的类型为 PathDashPathEffect.Style ，是一个 enum ，具体有三个值：
    * TRANSLATE：位移
    * ROTATE：旋转
    * MORPH：变体

![](https://pic4.zhimg.com/v2-4f8c19fcaf7158b4cfe79f49ad95f9ef_b.png)

### SumPathEffect

这是一个组合效果类的 PathEffect 。它的行为特别简单，就是分别按照两种 PathEffect分别对目标进行绘制。

```Java
PathEffect dashEffect = new DashPathEffect(new float[]{20, 10}, 0);  
PathEffect discreteEffect = new DiscretePathEffect(20, 5);  
pathEffect = new SumPathEffect(dashEffect, discreteEffect);

...

canvas.drawPath(path, paint); 
```

### ComposePathEffect

这也是一个组合效果类的 PathEffect 。不过它是先对目标 Path 使用一个 PathEffect，然后再对这个改变后的 Path 使用另一个 PathEffect。

```Java
PathEffect dashEffect = new DashPathEffect(new float[]{20, 10}, 0);  
PathEffect discreteEffect = new DiscretePathEffect(20, 5);  
pathEffect = new ComposePathEffect(dashEffect, discreteEffect);

...

canvas.drawPath(path, paint); 
```
![](https://pic2.zhimg.com/v2-9f441769c17a333bc1e9f78ec92a7831_b.png)

它的构造方法 ComposePathEffect(PathEffect outerpe, PathEffect innerpe) 中的两个 PathEffect 参数， innerpe 是先应用的， outerpe 是后应用的。所以上面的代码就是「先偏离，再变虚线」。而如果把两个参数调换，就成了「先变虚线，再偏离」。

注意： PathEffect 在有些情况下不支持硬件加速，需要关闭硬件加速才能正常使用：

* Canvas.drawLine() 和 Canvas.drawLines() 方法画直线时，setPathEffect()是不支持硬件加速的；
* PathDashPathEffect 对硬件加速的支持也有问题，所以当使用 PathDashPathEffect 的时候，最好也把硬件加速关了

## setShadowLayer阴影

setShadowLayer(float radius, float dx, float dy, int shadowColor)，在之后的绘制内容下面加一层阴影。

```Java
paint.setShadowLayer(10, 0, 0, Color.RED);

...

canvas.drawText(text, 80, 300, paint); 
```
![](https://pic2.zhimg.com/v2-4908ff659c9c5a1203128e5fb1b22315_b.png)

效果就是上面这样。方法的参数里， radius 是阴影的模糊范围； dx dy 是阴影的偏移量； shadowColor 是阴影的颜色。

如果要清除阴影层，使用 clearShadowLayer() 。

注意：

* 在硬件加速开启的情况下， setShadowLayer() 只支持文字的绘制，文字之外的绘制必须关闭硬件加速才能正常绘制阴影。
* 如果 shadowColor 是半透明的，阴影的透明度就使用 shadowColor 自己的透明度；
* 如果 shadowColor 是不透明的，阴影的透明度就使用 paint 的透明度。

## setMaskFilter

setMaskFilter(MaskFilter maskfilter)，为之后的绘制设置 MaskFilter。上一个方法 setShadowLayer() 是设置的在绘制层下方的附加效果；而这个 MaskFilter 和它相反，设置的是在绘制层上方的附加效果。

>到现在已经有两个 setXxxFilter(filter) 了。前面有一个 setColorFilter(filter)，是对每个像素的颜色进行过滤；而这里的 setMaskFilter(filter) 则是基于整个画面来进行过滤。

MaskFilter 有两种： BlurMaskFilter 和 EmbossMaskFilter。

### BlurMaskFilter

模糊效果的 MaskFilter。

```Java
paint.setMaskFilter(new BlurMaskFilter(50, BlurMaskFilter.Blur.NORMAL));

...

canvas.drawBitmap(bitmap, 100, 100, paint); 
```

![](https://pic3.zhimg.com/v2-ceca6b368f80a5083553cde06c4f6cc6_b.png)

它的构造方法 BlurMaskFilter(float radius, BlurMaskFilter.Blur style) 中， radius 参数是模糊的范围， style 是模糊的类型。一共有四种：

* NORMAL: 内外都模糊绘制
* SOLID: 内部正常绘制，外部模糊
* INNER: 内部模糊，外部不绘制
* OUTER: 内部不绘制，外部模糊

![](https://pic4.zhimg.com/v2-0b928e0dd74a6a430df6b545e5032acb_b.png)

### EmbossMaskFilter

浮雕效果的 MaskFilter。


![](https://pic3.zhimg.com/v2-a6433fdaad3b24e4a9d62fe4a58fc82e_b.png)

它的构造方法 EmbossMaskFilter(float[] direction, float ambient, float specular, float blurRadius)的参数里， direction 是一个 3 个元素的数组，指定了光源的方向； ambient 是环境光的强度，数值范围是 0 到 1； specular 是炫光的系数； blurRadius 是应用光线的范围。

## 获取绘制的 Path

### getFillPath(Path src, Path dst)

实际 Path」。所谓实际 Path ，指的就是 drawPath() 的绘制内容的轮廓，要算上线条宽度和设置的 PathEffect。

默认情况下（线条宽度为 0、没有 PathEffect），原 Path 和实际 Path 是一样的；而在线条宽度不为 0 （并且模式为 STROKE 模式或 FLL_AND_STROKE ），或者设置了 PathEffect的时候，实际 Path 就和原 Path 不一样了：

![](https://pic4.zhimg.com/v2-46ed45eb381ec1b723ab2d58b3e31023_b.png)

通过 getFillPath(src, dst) 方法就能获取这个实际 Path。方法的参数里，src 是原 Path ，而 dst 就是实际 Path 的保存位置。 getFillPath(src, dst) 会计算出实际 Path，然后把结果保存在 dst 里。

### getTextPath

>getTextPath(String text, int start, int end, float x, float y, Path path)
>getTextPath(char[] text, int index, int count, float x, float y, Path path)

「文字的 Path」。文字的绘制，虽然是使用 Canvas.drawText()方法，但其实在下层，文字信息全是被转化成图形，对图形进行绘制的。 getTextPath() 方法，获取的就是目标文字所对应的 Path 。这个就是所谓「文字的 Path」

![](https://pic3.zhimg.com/v2-07a86031bd9d471a8fae0f6e4148d0f6_b.png)

这两个方法， getFillPath() 和 getTextPath() ，就是获取绘制的 Path 的方法。之所以把它们归类到「效果」类方法，是因为它们主要是用于图形和文字的装饰效果的位置计算,比如[自定义的下划线效果](https://medium.com/google-developers/a-better-underline-for-android-90ba3a2e4fb)。

![](https://pic4.zhimg.com/v2-75bd8384345da721b2d36a13e9d4da03_b.png)

# drawText() 相关

Paint 有些设置是文字绘制相关的，即和 drawText() 相关的。

比如设置文字大小：

![](https://pic1.zhimg.com/v2-b6d745429c425efda98e2e5626f6202c_b.png)

比如设置文字间隔：

![](https://pic1.zhimg.com/v2-8aae726a7af7680b464841e3632f5364_b.png)

比如设置各种文字效果：

![](https://pic2.zhimg.com/v2-90e3d7a3a8227f7c823168565dbb88c9_b.png)

# 初始化类

## reset()

重置 Paint 的所有属性为默认值。相当于重新 new 一个，不过性能当然高一些啦。

## set(Paint src)

把 src 的所有属性全部复制过来。相当于调用 src 所有的 get 方法，然后调用这个 Paint的对应的 set 方法来设置它们。

## setFlags(int flags)

批量设置 flags。相当于依次调用它们的 set 方法。例如：

```Java
paint.setFlags(Paint.ANTI_ALIAS_FLAG | Paint.DITHER_FLAG); 
```

等价于

```Java
paint.setAntiAlias(true);  
paint.setDither(true);
```




