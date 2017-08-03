**Android 5.x之Toolbar和Palette**

Toolbar是Android5.0后应用的内容的标准工具栏，可以说是ActionBar的升级版，两者不是独立的关系，要使用Toolbar还是得跟ActionBar有关系的。相比ActionBar，Toolbar最明显的一点就是变得很自由，可以随处放置，具体的使用方法和ActionBar很类似。

#1.Toolbar引入

首先还是得引入v7包，Android studio在build.gradle配置如下代码

```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.0.1'
    compile 'com.android.support:palette-v7:23.0.1'
}
```

接下来为了显示Toolbar控件，先要将style里的ActionBar去掉:

```
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>
```

设置各个部分属性的图：

![ALT TEXT](http://img.blog.csdn.net/20151202204454385)

接下来我们引入Toolbar：

```
<android.support.v7.widget.Toolbar
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    android:background="?attr/colorPrimary"
    app:popupTheme="@style/AppTheme.PopupOverlay" />
```

##2.主布局

在主布局中我们使用DrawerLayout来完成侧滑效果

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context="com.deason.library.mytoobar.MainActivity">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:popupTheme="@style/AppTheme.PopupOverlay" />

    </android.support.design.widget.AppBarLayout>

    <android.support.v4.widget.DrawerLayout
        android:id="@+id/id_drawerlayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <!--内容界面-->
        <LinearLayout
            android:id="@+id/ll_content"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical"
            android:background="@drawable/ic_launcher">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:text="内容界面"
                android:textColor="@android:color/white"/>


        </LinearLayout>
        <!--侧或菜单界面-->
        <LinearLayout
            android:id="@+id/ll_tabs"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@android:color/darker_gray"
            android:orientation="vertical"
            android:layout_gravity="start">

            <TextView
                android:id="@+id/tv_close"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:clickable="true"
                android:text="侧滑界面,点击收回侧滑"
                android:textColor="@android:color/white"/>
        </LinearLayout>

    </android.support.v4.widget.DrawerLayout>

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        android:src="@android:drawable/ic_dialog_email" />

</android.support.design.widget.CoordinatorLayout>

```

#3.Toolbar自定义Item布局

我们在menu/main.xml中去声明将在Toobar的Menu item,MenuItem的设置与ActionBar类似

```
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context="com.deason.library.mytoobar.MainActivity">

    <item
        android:id="@+id/ab_search"
        android:orderInCategory="80"
        android:title="搜索"
        app:actionViewClass="android.support.v7.widget.SearchView"
        app:showAsAction="ifRoom"/>
    <item
        android:id="@+id/action_share"
        android:orderInCategory="90"
        android:title="分享"
        app:actionProviderClass="android.support.v7.widget.ShareActionProvider"
        app:showAsAction="ifRoom"/>
    
    <item
        android:id="@+id/action_settings"
        android:orderInCategory="100"
        android:title="设置"
        app:showAsAction="never" />
</menu>

```

#4.java代码实现

```
package com.deason.library.mytoobar;

import android.content.Intent;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.drawable.ColorDrawable;
import android.os.Bundle;
import android.support.design.widget.FloatingActionButton;
import android.support.design.widget.Snackbar;
import android.support.v4.view.MenuItemCompat;
import android.support.v4.widget.DrawerLayout;
import android.support.v7.app.ActionBarDrawerToggle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.graphics.Palette;
import android.support.v7.widget.SearchView;
import android.support.v7.widget.ShareActionProvider;
import android.support.v7.widget.Toolbar;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {
    
    
    private DrawerLayout mDrawerLayout;
    private ActionBarDrawerToggle mDrawerToggle;
    private Toolbar toolbar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        //是否给左上角图标的左边加上一个返回的图标
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
//        getSupportActionBar().setLogo(R.mipmap.ic_launcher);
        
        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
                        .setAction("Action", null).show();
            }
        });
        
        toolbar.setOnMenuItemClickListener(new Toolbar.OnMenuItemClickListener() {
            @Override
            public boolean onMenuItemClick(MenuItem item) {

                Toast.makeText(MainActivity.this,"searchView"+item.getTitle(),Toast.LENGTH_SHORT).show();
                
                return true;
            }
        });
        
        initView();
        
    }

    private void initView() {
        
		//实现侧滑栏的切换
        mDrawerLayout = (DrawerLayout) findViewById(R.id.id_drawerlayout);
        mDrawerToggle = new ActionBarDrawerToggle(this, mDrawerLayout, toolbar, R.string.open, R.string.close);
        mDrawerToggle.syncState();
        mDrawerLayout.setDrawerListener(mDrawerToggle);
        
        //关闭抽屉栏
//        mDrawerLayout.closeDrawer(Gravity.LEFT);
        
        //Palette取色器
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(),R.drawable.ic_launcher);

        Palette.from(bitmap).generate(new Palette.PaletteAsyncListener() {
            @Override
            public void onGenerated(Palette palette) {
                Palette.Swatch swatch = palette.getVibrantSwatch();
                
                //设置Toolbar颜色
                getSupportActionBar().setBackgroundDrawable(new ColorDrawable(swatch.getRgb()));
                
                //设置系统状态栏颜色
                getWindow().setStatusBarColor(swatch.getRgb());
            }
        });

    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.menu_main, menu);

        //在菜单中找到对应控件的item
        MenuItem menuItem = menu.findItem(R.id.ab_search);
        
        //获取SearchView
        SearchView searchView = (SearchView) menuItem.getActionView();
        
        searchView.setOnSearchClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this,"searchView",Toast.LENGTH_SHORT).show();
            }
        });
        
        MenuItem shareItem = menu.findItem(R.id.action_share);
        ShareActionProvider shareActionProvider = (ShareActionProvider)MenuItemCompat.getActionProvider(shareItem);
        Intent intent = new Intent(Intent.ACTION_SEND);
        intent.setType("image/*");
        shareActionProvider.setShareIntent(intent);
        
        shareActionProvider.setOnShareTargetSelectedListener(new ShareActionProvider.OnShareTargetSelectedListener() {
            @Override
            public boolean onShareTargetSelected(ShareActionProvider source, Intent intent) {

                Toast.makeText(MainActivity.this,"onShareTargetSelected:"+ intent.getAction(),Toast.LENGTH_SHORT).show();
                
                return false;
            }
        });
        
        
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        // Handle action bar item clicks here. The action bar will
        // automatically handle clicks on the Home/Up button, so long
        // as you specify a parent activity in AndroidManifest.xml.
        int id = item.getItemId();

        //noinspection SimplifiableIfStatement
        if (id == R.id.action_settings) {

            Toast.makeText(MainActivity.this,"action_settings",Toast.LENGTH_SHORT).show();
            
            return true;
        }


        return super.onOptionsItemSelected(item);
    }
}

最后来看看效果:


![Toolbar](http://img.blog.csdn.net/20160227234641589)

```
#5.Paletta的应用

Android5.x的Paletta作用是提取图片的颜色，从而让主题能够动态适应当前界面的色调，做到整个App颜色的颜色基调和谐统一。

Android内置了几种提取色调的种类：

*	Vibrant：充满活力的
*	Vibrant dark 活力黑
*	Vibrant	light 活力亮
*	Muted 柔和的
*	Muted dark 柔和的黑
*	Muted light 柔和的亮

要使用Palette，我们需要引入com.android.support:palette-v7:23.0.1包。

实现提取颜色的步骤：

1. 获取一个Bitmap对象
2. 将Bitmap对象传递给Palette，然后调用generate方法
3. 在onGenerated回调中得到图片的色调，最后我们把Toolbar和系统状态栏的背景设置为该图片的色调

```
//Palette取色器
Bitmap bitmap = BitmapFactory.decodeResource(getResources(),R.drawable.ic_launcher);

Palette.from(bitmap).generate(new Palette.PaletteAsyncListener() {
    @Override
    public void onGenerated(Palette palette) {

		//这里我们获取的是图片充满活力的黑的色调
        Palette.Swatch swatch = palette.getDarkVibrantSwatch();
        
        //设置Toolbar颜色
        getSupportActionBar().setBackgroundDrawable(new ColorDrawable(swatch.getRgb()));
        
        //设置系统状态栏颜色
        getWindow().setStatusBarColor(swatch.getRgb());
    }
});
```

最后来看看效果:

![Toolbar_Paletta](http://img.blog.csdn.net/20160227234750966)

#参考文章

[ Android5.x Toolbar和Palette应用解析](http://blog.csdn.net/itachi85/article/details/50150747)

