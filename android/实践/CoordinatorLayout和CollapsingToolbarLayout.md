**Android Design Support Library之CoordinatorLayout和CollapsingToolbarLayout**


CoordinatorLayout是Android Design Support Library中比较难的控件，它是用来组织它的子View之间协助的一个父View，它直接继承于ViewGroup。

CoordinatorLayout默认情况下可理解是一个FrameLayout，它的布局方式是一层一层叠加上去的，这里我们来介绍它常用的两种情况。

##1.CoordinatorLayout实现ToolBar的隐藏效果

首先看看效果:

![CoordinatorLayout实现ToolBar的隐藏效果](http://upload-images.jianshu.io/upload_images/1014339-7d3bd2e5b2a4f58e.gif?imageMogr2/auto-orient/strip)

接下来开始实现，首先配置build.gradle

###1.1 build.gradle

```
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.2.0'
    compile 'com.android.support:design:23.2.0'
    compile 'com.android.support:recyclerview-v7:23.2.0'
}

```

com.android.support:design:23.2.0就是我们需要引入的兼容库

###1.2 主界面

主界面跟前面的Android Design Support Library之NavigationView一致，不同之处在于layout="@layout/app_bar_main"引入的布局实现的不同

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/drawer_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:openDrawer="start">

    <include
        layout="@layout/app_bar_main"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />


    <android.support.design.widget.NavigationView
        android:id="@+id/nav_view"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        android:fitsSystemWindows="true"
        app:headerLayout="@layout/nav_header_main"
        app:menu="@menu/activity_main_drawer" />


</android.support.v4.widget.DrawerLayout>

```

下面我们来看看app_bar_main.xml这个布局：

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main_content"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">


        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="100dp"
            android:background="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|enterAlways"
             />

        <android.support.design.widget.TabLayout
            android:id="@+id/tabs"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="?attr/colorPrimary"
            app:tabIndicatorColor="#ffffffff"
            app:tabMode="scrollable">
            
            </android.support.design.widget.TabLayout>
        

    </android.support.design.widget.AppBarLayout>
    
    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
		//必须设置，AppBarLayout才能接收到滚动事件
        app:layout_behavior="@string/appbar_scrolling_view_behavior"/>
    

    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        android:src="@android:drawable/ic_dialog_email"
        />

</android.support.design.widget.CoordinatorLayout>

```

CoordinatorLayout中不应该再设置android:fitsSystemWindows="true"这个配置，因为外层已经设置好，如果设置会出现不和谐的效果。

Toolbar能隐藏的关键在于app:layout_scrollFlags="scroll|enterAlways"这个事件，设置滚动事件，属性里面至少启用scroll这个flag，这样view才会滚动出屏幕，否则它将一直固定在头部。

###1.3 Java代码实现

```
package com.deason.library.mycoordinatorlayout;

import android.content.Intent;
import android.os.Bundle;
import android.support.design.widget.NavigationView;
import android.support.design.widget.TabLayout;
import android.support.v4.app.Fragment;
import android.support.v4.view.GravityCompat;
import android.support.v4.view.ViewPager;
import android.support.v4.widget.DrawerLayout;
import android.support.v7.app.ActionBarDrawerToggle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.view.Menu;
import android.view.MenuItem;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity
        implements NavigationView.OnNavigationItemSelectedListener {
    
    
    private ViewPager mViewPager;
    private TabLayout mTabLayout;
    

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);


        DrawerLayout drawer = (DrawerLayout) findViewById(R.id.drawer_layout);
        ActionBarDrawerToggle toggle = new ActionBarDrawerToggle(
                this, drawer, toolbar, R.string.navigation_drawer_open, R.string.navigation_drawer_close);
        drawer.setDrawerListener(toggle);
        toggle.syncState();

        NavigationView navigationView = (NavigationView) findViewById(R.id.nav_view);
        navigationView.setNavigationItemSelectedListener(this);
        
        mViewPager = (ViewPager) findViewById(R.id.viewpager);
        
        initViewPager();
    }

    private void initViewPager() {

        mTabLayout = (TabLayout) findViewById(R.id.tabs);

        mTabLayout.setSelectedTabIndicatorHeight(10);

        List<String> titles = new ArrayList<String>();
        titles.add("新闻");
        titles.add("财经");
        titles.add("娱乐");
        titles.add("体育");
        titles.add("军事");
        titles.add("科技");
        titles.add("教育");
        titles.add("历史");
        titles.add("文化");
        titles.add("深圳");

        for(int i= 0;i < titles.size();i++) {
            //设置TabIndicator文字
            mTabLayout.addTab(mTabLayout.newTab().setText(titles.get(i)));
        }

        List<Fragment> fragments = new ArrayList<Fragment>();

        for (int i = 0; i < titles.size(); i++) {

            fragments.add(new TabFragment());
        }

        //实例化适配器
        FragmentAdapter mAdapter = new FragmentAdapter(getSupportFragmentManager(),fragments,titles);

        //给ViewPager设置适配器
        mViewPager.setAdapter(mAdapter);

        //将TabLayout和ViewPager关联起来
        mTabLayout.setupWithViewPager(mViewPager);

        //给TabLayout设置适配器
       mTabLayout.setTabsFromPagerAdapter(mAdapter);
    }

    @Override
    public void onBackPressed() {
        DrawerLayout drawer = (DrawerLayout) findViewById(R.id.drawer_layout);
        if (drawer.isDrawerOpen(GravityCompat.START)) {
            drawer.closeDrawer(GravityCompat.START);
        } else {
            super.onBackPressed();
        }
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.main, menu);
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

            Intent intent = new Intent(this,CollapsingActivity.class);
            startActivity(intent);
            
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

    @SuppressWarnings("StatementWithEmptyBody")
    @Override
    public boolean onNavigationItemSelected(MenuItem item) {
        // Handle navigation view item clicks here.
        int id = item.getItemId();

        if (id == R.id.nav_camera) {
            // Handle the camera action
        } else if (id == R.id.nav_gallery) {

        } else if (id == R.id.nav_slideshow) {

        } else if (id == R.id.nav_manage) {

        } else if (id == R.id.nav_share) {

        } else if (id == R.id.nav_send) {

        }

        DrawerLayout drawer = (DrawerLayout) findViewById(R.id.drawer_layout);
        drawer.closeDrawer(GravityCompat.START);
        return true;
    }
}

```
其他代码如ViewPager、RecyclerView的实现请参考Android Design Support Library之TabLayout一文。

###1.4 效果

![CoordinatorLayout实现ToolBar的隐藏效果](http://upload-images.jianshu.io/upload_images/1014339-7d3bd2e5b2a4f58e.gif?imageMogr2/auto-orient/strip)

##2.CoordinatorLayout+CollapsingToolbarLayout实现Toolbar折叠效果 

首先看看效果：

![CoordinatorLayout+CollapsingToolbarLayout实现Toolbar折叠效果](http://upload-images.jianshu.io/upload_images/1014339-21388cdc3cacbd6d.gif?imageMogr2/auto-orient/strip)

要实现折叠效果我们需要引入一个新的布局CollapsingToolbarLayout，它作用是提供了一个可以折叠的Toolbar，它继承至FrameLayout，给它设置layout_scrollFlags，它可以控制包含在CollapsingToolbarLayout中的控件比如mageView、Toolbar在响应layout_behavior事件时作出相应的scrollFlags滚动事件。

###2.1 布局

布局文件用CollapsingToolbarLayout将ImageView和Toolbar包含起来作为一个可折叠的Toolbar，再用AppBarLayout包裹起来作为一个Appbar的整体，当然，AppBarLayout目前必须是第一个嵌套在CoordinatorLayout里面的子view。

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/main_content"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:fitsSystemWindows="true"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginEnd="64dp"
            app:expandedTitleMarginStart="48dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <ImageView
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:fitsSystemWindows="true"
                android:scaleType="centerCrop"
                android:src="@drawable/mao" />


            <android.support.v7.widget.Toolbar
                android:id="@+id/toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />


        </android.support.design.widget.CollapsingToolbarLayout>


    </android.support.design.widget.AppBarLayout>
    <!--
        <include layout="@layout/content_main" />
        
    -->

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scrollbars="none"
		//必须设置，AppBarLayout才能接收到滚动事件
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />

</android.support.design.widget.CoordinatorLayout>

```

CollapsingToolbarLayout的几个关键属性需要说明一下：

*	app:contentScrim="?attr/colorPrimary"，用来设置CollapsingToolbarLayout收缩后最顶部的颜色

*	app:expandedTitleGravity= "left|bottom"，表示将此CollapsingToolbarLayout完全展开后，title所处的位置，默认是left|bottom

*	app:collapsedTitleGravity= "left"，表示当头部的图片ImageView消失后，此title将回归到ToolBar的位置，默认是left

*	app:layout_scrollFlags="scroll|exitUntilCollapsed"，这个属性我们上面讲过用来设置滚动事件，属性里面必须至少启用scroll这个flag，这样这个view才会滚动出屏幕，否则它将一直固定在顶部。这里我们设置的是app:layout_scrollFlags=”scroll|exitUntilCollapsed”这样能实现折叠效果，如果想要隐藏效果我们可以设置app:layout_scrollFlags=”scroll|enterAlways”

我们需要定义AppBarLayout与滚动视图之间的联系，Design Support Library包含了一个特殊的字符串资源@string/appbar_scrolling_view_behavior，它和AppBarLayout.ScrollingViewBehavior相匹配，用来通知AppBarLayout何时发生了滚动事件，这个behavior需要设置在触发事件的view之上，所以我们应该在RecyclerView或者任意支持嵌套滚动的view比如NestedScrollView上添加app:layout_behavior=”@string/appbar_scrolling_view_behavior这个属性，当然AppBarLayout 中的子view需要设置app:layout_scrollFlags这个属性，否则接收到RecyclerView滚动事件，AppBarLayout 也不会有什么变化。

###2.2 Java代码实现

```
package com.deason.library.mycoordinatorlayout;

import android.os.Bundle;
import android.support.design.widget.CollapsingToolbarLayout;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.Toolbar;

public class CollapsingActivity extends AppCompatActivity {

    private RecyclerView mRecyclerView;
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.collapsing_main);
        
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        CollapsingToolbarLayout collapsingToolbarLayout = (CollapsingToolbarLayout) findViewById(R.id.collapsing_toolbar);
        collapsingToolbarLayout.setTitle("哆啦A梦");

        mRecyclerView = (RecyclerView) findViewById(R.id.recyclerView);
        mRecyclerView.setLayoutManager(new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false));
        mRecyclerView.setAdapter(new RecyclerViewAdapter(this));
        
        
        
    }
}

```

###2.3 效果

![CoordinatorLayout+CollapsingToolbarLayout实现Toolbar折叠效果](http://upload-images.jianshu.io/upload_images/1014339-21388cdc3cacbd6d.gif?imageMogr2/auto-orient/strip)

##3.参考文章

[ Android Design Support Library（三）用CoordinatorLayout实现Toolbar隐藏和折叠](http://blog.csdn.net/itachi85/article/details/50492695)

