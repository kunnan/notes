**TabLayout实现网易新闻滑动标签效果**

TabLayout是Android Design Support Library库中的控件。Google在2015年的IO大会上，给我们带来了更加详细的Material Design设计规范，同时也给我们带来了全新的Android Design Support Library，在这个support库中,Google给我们提供了更加规范的MD设计风格的控件。最重要的是，Android Design Support Library可以向下兼容到Android 2.2。

接下来，我们开始熟悉TabLayout的使用，并完成一个类似网易新闻客户端滑动标签的效果，它的滑动标签由Toolbar+TabLayout实现，内容显示由ViewPager+Fragment实现。

#1.配置build.gradle

```
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.1.1'
    compile 'com.android.support:recyclerview-v7:23.1.1'
    compile 'com.android.support:support-v4:23.1.1'
    compile 'com.android.support:cardview-v7:23.1.1'
    compile 'com.android.support:design:23.2.0'
}
```
com.android.support:design:23.2.0就是我们需要引入的Android Design Support Library，其次我们还引入了RecyclerView和CardView两个Android 5.X的新控件。

#2.主界面的布局

主界面的布局由AppBarLayout、Toolbar和TabLayout，以及ViewPager组成，主界面布局文件activity_main.xml如下:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    android:orientation="vertical">

    <android.support.design.widget.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:theme="@style/AppTheme.AppBarOverlay">

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            android:background="?attr/colorPrimary"
            app:layout_scrollFlags="scroll|enterAlways"
            app:popupTheme="@style/AppTheme.PopupOverlay" />

        <android.support.design.widget.TabLayout
            android:id="@+id/tabs"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:tabIndicatorColor="#ADBE107E"
            app:tabMode="scrollable">

        </android.support.design.widget.TabLayout>

    </android.support.design.widget.AppBarLayout>

    <android.support.v4.view.ViewPager
        android:id="@+id/viewpager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">
        
    </android.support.v4.view.ViewPager>


</LinearLayout>
```
这里用到AppBarLayout和Toolbar，AppBarLayout是Android Design Support Library新加的空间继承自LinearLayout，它用来将Toolbar和TabLayout组合起来成为一个整体。ViewPager用来实现Fragment页面的切换

布局文件中最关键的一点是android.support.design.widget.TabLayout的app:tabMode="scrollable"，它表示tab标签的模式是可滑动的，如果不设置次模式的话，标签将不可滑动。

#3.主界面的实现

1. 实例化TabLayout，给TabLayout标签设置文字
3. 实例化FragmentAdapter，加载Fragment
4. 实例化ViewPager，并设置ViewPager的适配器FragmentAdapter
5. TabLayout与ViewPager关联
6. TabLayout设置适配器FragmentAdapter

MainActivity.java如下所示

```
package com.deason.library.mytablayout;

import android.os.Bundle;
import android.support.design.widget.TabLayout;
import android.support.v4.app.Fragment;
import android.support.v4.view.ViewPager;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.view.Menu;
import android.view.MenuItem;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {
    
    private ViewPager mViewPager;
    private TabLayout mTabLayout;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        mViewPager = (ViewPager) findViewById(R.id.viewpager);
        initViewPager();
    }

    private void initViewPager() {

        mTabLayout = (TabLayout) findViewById(R.id.tabs);

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
        titles.add("校园");
        
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
}

```

###3.1 实例化FragmentAdapter

FragmentAdapter.java

```
package com.deason.library.mytablayout;

import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentPagerAdapter;

import java.util.List;

/**
 * Created by liuguoquan on 2016/2/24.
 */
public class FragmentAdapter extends FragmentPagerAdapter {
    
    private List<Fragment>  mFragments;
    private List<String>    mTitles;

    public FragmentAdapter(FragmentManager fm,List<Fragment> fragments,List<String> titles) {
        super(fm);
        
        this.mFragments = fragments;
        this.mTitles = titles;
    }

    @Override
    public Fragment getItem(int position) {
        return mFragments.get(position);
    }

    @Override
    public int getCount() {
        return mFragments.size();
    }

	//Tabs标签名称
    @Override
    public CharSequence getPageTitle(int position) {
        return mTitles.get(position);
    }
}

```
###3.2 TabFragment

```
package com.deason.library.mytablayout;


import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;


/**
 * A simple {@link Fragment} subclass.
 */
public class TabFragment extends Fragment {

    private RecyclerView mRecyclerView;


    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        View view = inflater.inflate(R.layout.fragment_list,container,false);
        return view;
    }


    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);

        
    }
}

```

###4 运行效果

![ALT TEXT](http://upload-images.jianshu.io/upload_images/1014339-823ac9ef8636d01f.gif?imageMogr2/auto-orient/strip)


