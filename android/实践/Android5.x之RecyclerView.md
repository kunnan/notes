[TOC]

RecyclerView是Android官方推出的旨在取代ListView、GridView的控件，可以通过导入support-V7进行使用。据官方介绍，该控件用于在有限的窗口中展示大量数据集，同样能实现此效果的有ListView、GridView。

那么有了ListView、GridView之后为什么还需要RecyclerView这样的控件呢？整体上看RecyclerView架构，它提供了一种插拔式的体验，高度的解耦，异常的灵活，通过设置它提供不同的LayoutManager、ItemDecoration、ItemAnimator来实现各种各样的效果。

*	控制其显示的布局方式，请设置布局管理器LayoutManager
*	控制Item间的间隔，请实现ItemDecoration
*	控制Item间增删的动画，请实现ItemAnimator
*	控制其点击、长按事件，不好意思请自己写

下面，我们开始进入RecyclerView的体验之旅

#1.build.gradle

要想使用RecyclerView，首先我们要导入support-V7包，Android Studio中需要的build.gradle文件中加入下面代码自动导入V7包。

```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.1.1'  //v7
    compile 'com.android.support:design:23.1.1'
    compile 'com.android.support:cardview-v7:23.1.1'
    compile 'com.android.support:recyclerview-v7:23.1.1'
    compile 'com.android.support:palette-v7:23.1.1'
}
```

#2.RecyclerView实现ListView效果

1. 实例化RecyclerView
2. 设置RecyclerView的布局（有ListView、GridView和瀑布流三种效果，水平和垂直两个方向）
3. 设置RecyclerView的Item分割线
4. 设置RecyclerView的Item增删时的动画
5. 实例化一个RecyclerViewAdapter
6. 设置RecyclerView的Adapter


```
mRecyclerView = (RecyclerView) findViewById(R.id.recyclerview);
//设置布局管理器
mRecyclerView.setLayoutManager(new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false)); //垂直方向的list
//mRecyclerView.setLayoutManager(new LinearLayoutManager(this,LinearLayoutManager.HORIZONTAL,false)); //水平方向的list

//增加Item分割线，必须在setAdapter方法之前设置
mRecyclerView.addItemDecoration(new DividerItemDecoration(this, DividerItemDecoration.VERTICAL_LIST));

//设置增加和删除条目时的动画
mRecyclerView.setItemAnimator(new DefaultItemAnimator());

mAdapter = new RecyclerViewAdapter(this,mDatas);

mRecyclerView.setAdapter(mAdapter);
```

##2.1 主界面布局

RecyclerView的类名全称为android.support.v7.widget.RecyclerView

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true"
    tools:context="com.example.michael.recyclerviewdemo.MainActivity">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <android.support.design.widget.AppBarLayout
            android:id="@+id/appbar"
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


        <android.support.v7.widget.RecyclerView
            android:id="@+id/recyclerview"
            android:layout_width="match_parent"
            android:layout_height="match_parent">

        </android.support.v7.widget.RecyclerView>

    </LinearLayout>


    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        android:src="@android:drawable/ic_dialog_email" />

</android.support.design.widget.CoordinatorLayout>

```

##2.2 分割线DividerItemDecoration

DividerItemDecoration是实现的是RecyclerView作为ListView效果时的分割线。

```
package com.example.michael.recyclerviewdemo;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Rect;
import android.graphics.drawable.Drawable;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;

/**
 * 横向和纵向的分割线
 * Created by Michael on 2016/2/25.
 */
public class DividerItemDecoration extends RecyclerView.ItemDecoration {

    private static final int[] ATTRS = new int[]{
            android.R.attr.listDivider
    };

    public static final int HORIZONTAL_LIST = LinearLayoutManager.HORIZONTAL;

    public static final int VERTICAL_LIST = LinearLayoutManager.VERTICAL;

    private Drawable mDivider;

    private int mOrientation;

    public DividerItemDecoration(Context context, int orientation) {
        final TypedArray a = context.obtainStyledAttributes(ATTRS);
        mDivider = a.getDrawable(0);
        a.recycle();
        setOrientation(orientation);
    }

    public void setOrientation(int orientation) {
        if (orientation != HORIZONTAL_LIST && orientation != VERTICAL_LIST) {
            throw new IllegalArgumentException("invalid orientation");
        }
        mOrientation = orientation;
    }

    @Override
    public void onDraw(Canvas c, RecyclerView parent) {
        if (mOrientation == VERTICAL_LIST) {
            drawVertical(c, parent);
        } else {
            drawHorizontal(c, parent);
        }

    }


    public void drawVertical(Canvas c, RecyclerView parent) {
        final int left = parent.getPaddingLeft();
        final int right = parent.getWidth() - parent.getPaddingRight();

        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            android.support.v7.widget.RecyclerView v = new android.support.v7.widget.RecyclerView(parent.getContext());
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int top = child.getBottom() + params.bottomMargin;
            final int bottom = top + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    public void drawHorizontal(Canvas c, RecyclerView parent) {
        final int top = parent.getPaddingTop();
        final int bottom = parent.getHeight() - parent.getPaddingBottom();

        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int left = child.getRight() + params.rightMargin;
            final int right = left + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    @Override
    public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent) {
        if (mOrientation == VERTICAL_LIST) {
            outRect.set(0, 0, 0, mDivider.getIntrinsicHeight());
        } else {
            outRect.set(0, 0, mDivider.getIntrinsicWidth(), 0);
        }
    }


}

```

##2.3 实现Adapter

RecyclerViewAdapter用于RecyclerView作为ListView和GridView效果时Adapter，通过其构造函数传入Context和需要绑定的数据集合。一个基本的Adapter里面有需要实现如下功能：

*	创建RecyclerView.ViewHolder的子类
*	视图与数据绑定
*	点击事件回调接口
*	Item的增加和删除更新


```
package com.example.michael.recyclerviewdemo;

import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import java.util.List;

/**
 * Created by Michael on 2016/2/25.
 */
public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerViewAdapter.MyViewHolder> {

    private Context mContext;
    private List<String> mDatas;

    private OnItemClickListener mListener;

    /**
     * 点击事件回调接口
     */
    public interface OnItemClickListener {

        void onItemClick(View view, int position);

        void onItemLongClick(View view, int position);
    }

    public void setOnItemClickListener(OnItemClickListener listener) {

        this.mListener = listener;
    }

    public RecyclerViewAdapter(Context context, List<String> datas) {

        this.mContext = context;
        mDatas = datas;
    }

	//实例化ViewHolder
    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {

        View view = LayoutInflater.from(mContext).inflate(R.layout.item_recyclerview, parent, false);

        return new MyViewHolder(view);
    }

	//绑定数据
    @Override
    public void onBindViewHolder(final MyViewHolder holder, final int position) {


        System.out.println(position);

        holder.mClickBtn.setText(mDatas.get(position));

        if (mListener != null) {

            holder.mClickBtn.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {

                    //获取实际的位置
                    int pos = holder.getLayoutPosition();

                    mListener.onItemClick(holder.mClickBtn,pos);

                }
            });

            holder.mClickBtn.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {

                    int pos = holder.getLayoutPosition();

                    mListener.onItemLongClick(holder.mClickBtn,pos);

                    return true;  //屏蔽单击  false点击事件仍会响应
                }
            });

        }
    }

    @Override
    public int getItemCount() {

        return mDatas.size();

    }

    /**
     * 在当前Item之前添加Item
     */
    public void addPreItem(String data,int position) {

        mDatas.add(position,data);
        notifyItemInserted(position);
    }

    /**
     * 在当前Item之后添加Item
     */
    public void addNextItem(String data,int position) {

        mDatas.add(position+1,data);
        notifyItemInserted(position+1);
    }

    /**
     * 删除Item
     * @param position
     */
    public void removeItem(int position) {
        mDatas.remove(position);
        notifyItemRemoved(position);
    }

    class MyViewHolder extends RecyclerView.ViewHolder {

        TextView mClickBtn;

        public MyViewHolder(View itemView) {
            super(itemView);

            mClickBtn = (TextView) itemView.findViewById(R.id.btn_click);
        }
    }
}

```

RecyclerView的Item布局item_recycleview.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="60dp"
    android:orientation="vertical">

    <!--
    android:background="?android:attr/selectableItemBackground设置点击时水波纹效果 Android5.0以上有效果
    -->

    <TextView
        android:id="@+id/btn_click"
        android:clickable="true"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="?android:attr/selectableItemBackground"
        android:gravity="center"
        android:text="点击"/>


</LinearLayout>
```

##2.4 实现RecyclerView

```
package com.example.michael.recyclerviewdemo;

import android.content.DialogInterface;
import android.content.Intent;
import android.os.Bundle;
import android.support.design.widget.FloatingActionButton;
import android.support.design.widget.Snackbar;
import android.support.v7.app.AlertDialog;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.DefaultItemAnimator;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.StaggeredGridLayoutManager;
import android.support.v7.widget.Toolbar;
import android.view.Menu;
import android.view.MenuItem;
import android.view.View;
import android.widget.Toast;

import java.util.ArrayList;
import java.util.List;


public class MainActivity extends AppCompatActivity {

    private RecyclerView mRecyclerView;
    private RecyclerViewAdapter mAdapter;
    private StaggeredAdapter mStaggeredAdapter;
    private List<String> mDatas;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Snackbar.make(view, "Replace with your own action", Snackbar.LENGTH_LONG)
                        .setAction("Action", null).show();
            }
        });
        
        initView();
        initData();
        initEvent();
    }

    private void initView() {

        mRecyclerView = (RecyclerView) findViewById(R.id.recyclerview);

    }

    private void initData() {

		
        mDatas = new ArrayList<String>();

        for(int i = 0;i < 20;i++) {

            mDatas.add(String.valueOf(i));
        }

        setListView();
    }

    /**
     * ListView效果
     */
    private void setListView() {

        //设置布局管理器
        mRecyclerView.setLayoutManager(new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false)); //垂直方向的list
//        mRecyclerView.setLayoutManager(new LinearLayoutManager(this,LinearLayoutManager.HORIZONTAL,false)); //水平方向的list

        //增加Item分割线，必须在setAdapter方法之前设置
        mRecyclerView.addItemDecoration(new DividerItemDecoration(this, DividerItemDecoration.VERTICAL_LIST));

        //设置增加和删除条目时的动画
        mRecyclerView.setItemAnimator(new DefaultItemAnimator());

        mAdapter = new RecyclerViewAdapter(this,mDatas);

        mRecyclerView.setAdapter(mAdapter);

    }

    private void initEvent(){

		//实现RecyclerView的点击事件
        mAdapter.setOnItemClickListener(new RecyclerViewAdapter.OnItemClickListener() {
            @Override
            public void onItemClick(View view, final int position) {

                Toast.makeText(MainActivity.this,"onItemClick: " +position,Toast.LENGTH_SHORT).show();

                AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this);
                builder.setTitle("添加Item");
                builder.setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {

                        mAdapter.addNextItem("add pre",position);
                    }
                });
                builder.show();
            }

            @Override
            public void onItemLongClick(View view, final int position) {

                Toast.makeText(MainActivity.this,"onItemLongClick " + position,Toast.LENGTH_SHORT).show();

                AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this);
                builder.setTitle("删除Item");
                builder.setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {

                        mAdapter.removeItem(position);
                    }
                });
                builder.show();
            }
        });

    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {

        getMenuInflater().inflate(R.menu.menu_main,menu);

        return super.onCreateOptionsMenu(menu);
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {

        switch (item.getItemId()) {

            case R.id.action_settings:

                Intent intent = new Intent(MainActivity.this,CardViewActivity.class);
                startActivity(intent);

                break;
        }

        return super.onOptionsItemSelected(item);
    }

}

```

##2.5 效果

![ALT TEXT](http://img.blog.csdn.net/20160226170143271)

#3.RecyclerView实现GridView效果

RecyclerView实现GridView与实现ListView效果的不同之处有两点：

*	布局管理器为StaggeredGridLayoutManager
*	Item间隔类为DividerGridItemDecoration

##3.1 DividerGridItemDecoration

```
package com.example.michael.recyclerviewdemo;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Rect;
import android.graphics.drawable.Drawable;
import android.support.v7.widget.GridLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.StaggeredGridLayoutManager;
import android.view.View;

/**
 * GridView分割线
 * Created by Michael on 2016/2/25.
 */
public class DividerGridItemDecoration extends RecyclerView.ItemDecoration {

    //系统theme.xml中,可以自定义
    private static final int[] ATTRS = new int[] { android.R.attr.listDivider };
    private Drawable mDivider;

    public DividerGridItemDecoration(Context context)
    {
        final TypedArray a = context.obtainStyledAttributes(ATTRS);
        mDivider = a.getDrawable(0);
        a.recycle();
    }

    @Override
    public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state)
    {

        drawHorizontal(c, parent);
        drawVertical(c, parent);

    }

    private int getSpanCount(RecyclerView parent)
    {
        // 列数
        int spanCount = -1;
        RecyclerView.LayoutManager layoutManager = parent.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager)
        {

            spanCount = ((GridLayoutManager) layoutManager).getSpanCount();
        } else if (layoutManager instanceof StaggeredGridLayoutManager)
        {
            spanCount = ((StaggeredGridLayoutManager) layoutManager)
                    .getSpanCount();
        }
        return spanCount;
    }

    public void drawHorizontal(Canvas c, RecyclerView parent)
    {
        int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++)
        {
            final View child = parent.getChildAt(i);
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int left = child.getLeft() - params.leftMargin;
            final int right = child.getRight() + params.rightMargin
                    + mDivider.getIntrinsicWidth();
            final int top = child.getBottom() + params.bottomMargin;
            final int bottom = top + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    public void drawVertical(Canvas c, RecyclerView parent)
    {
        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++)
        {
            final View child = parent.getChildAt(i);

            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int top = child.getTop() - params.topMargin;
            final int bottom = child.getBottom() + params.bottomMargin;
            final int left = child.getRight() + params.rightMargin;
            final int right = left + mDivider.getIntrinsicWidth();

            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    private boolean isLastColum(RecyclerView parent, int pos, int spanCount,
                                int childCount)
    {
        RecyclerView.LayoutManager layoutManager = parent.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager)
        {
            if ((pos + 1) % spanCount == 0)// 如果是最后一列，则不需要绘制右边
            {
                return true;
            }
        } else if (layoutManager instanceof StaggeredGridLayoutManager)
        {
            int orientation = ((StaggeredGridLayoutManager) layoutManager)
                    .getOrientation();
            if (orientation == StaggeredGridLayoutManager.VERTICAL)
            {
                if ((pos + 1) % spanCount == 0)// 如果是最后一列，则不需要绘制右边
                {
                    return true;
                }
            } else
            {
                childCount = childCount - childCount % spanCount;
                if (pos >= childCount)// 如果是最后一列，则不需要绘制右边
                    return true;
            }
        }
        return false;
    }

    private boolean isLastRaw(RecyclerView parent, int pos, int spanCount,
                              int childCount)
    {
        RecyclerView.LayoutManager layoutManager = parent.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager)
        {
            childCount = childCount - childCount % spanCount;
            if (pos >= childCount)// 如果是最后一行，则不需要绘制底部
                return true;
        } else if (layoutManager instanceof StaggeredGridLayoutManager)
        {
            int orientation = ((StaggeredGridLayoutManager) layoutManager)
                    .getOrientation();
            // StaggeredGridLayoutManager 且纵向滚动
            if (orientation == StaggeredGridLayoutManager.VERTICAL)
            {
                childCount = childCount - childCount % spanCount;
                // 如果是最后一行，则不需要绘制底部
                if (pos >= childCount)
                    return true;
            } else
            // StaggeredGridLayoutManager 且横向滚动
            {
                // 如果是最后一行，则不需要绘制底部
                if ((pos + 1) % spanCount == 0)
                {
                    return true;
                }
            }
        }
        return false;
    }

    @Override
    public void getItemOffsets(Rect outRect, int itemPosition,
                               RecyclerView parent)
    {
        int spanCount = getSpanCount(parent);
        int childCount = parent.getAdapter().getItemCount();
        if (isLastRaw(parent, itemPosition, spanCount, childCount))// 如果是最后一行，则不需要绘制底部
        {
            outRect.set(0, 0, mDivider.getIntrinsicWidth(), 0);
        } else if (isLastColum(parent, itemPosition, spanCount, childCount))// 如果是最后一列，则不需要绘制右边
        {
            outRect.set(0, 0, 0, mDivider.getIntrinsicHeight());
        } else
        {
            outRect.set(0, 0, mDivider.getIntrinsicWidth(),
                    mDivider.getIntrinsicHeight());
        }
    }

}

```


##3.2 实现

```
/**
 * GridView效果
 */

private void setGridView() {

    mRecyclerView.setLayoutManager(new GridLayoutManager(this,4));
    mRecyclerView.addItemDecoration(new DividerGridItemDecoration(this));
    mRecyclerView.setItemAnimator(new DefaultItemAnimator());
    mAdapter = new RecyclerViewAdapter(this,mDatas);
    mRecyclerView.setAdapter(mAdapter);

}

或者

private void setGridView() {

    mRecyclerView.setLayoutManager(new StaggeredGridLayoutManager(4, StaggeredGridLayoutManager.VERTICAL));
    mRecyclerView.addItemDecoration(new DividerGridItemDecoration(this));
    mRecyclerView.setItemAnimator(new DefaultItemAnimator());
    mAdapter = new RecyclerViewAdapter(this,mDatas);
    mRecyclerView.setAdapter(mAdapter);

}
```

**注意：**StaggeredGridLayoutManager构造的第二个参数传一个orientation，如果传入的是StaggeredGridLayoutManager.VERTICAL代表有多少列；那么传入的如果是StaggeredGridLayoutManager.HORIZONTAL就代表有多少行。

##3.3 效果

![ALT TEXT](http://img.blog.csdn.net/20160226170212631)

#4.RecyclerView实现瀑布流效果

RecyclerView实现瀑布流效果与实现ListView效果的不同之处有三点：

*	布局管理器为StaggeredGridLayoutManager
*	Item间隔类为DividerGridItemDecoration
*	Adapter不同，实现瀑布流需要在adaper写一个随机的高度来控制每个item的高度就可以了，通常这个高度是由服务端返回的数据高度来控制的。

##4.1 实现

```
/**
 * 瀑布流效果
 */
private void setWaterfallView() {

    mRecyclerView.setLayoutManager(new StaggeredGridLayoutManager(4,StaggeredGridLayoutManager.VERTICAL));
    mRecyclerView.addItemDecoration(new DividerGridItemDecoration(this));
    mRecyclerView.setItemAnimator(new DefaultItemAnimator());
    mStaggeredAdapter = new StaggeredAdapter(this,mDatas);
    mRecyclerView.setAdapter(mStaggeredAdapter);

}
```

**注意：**StaggeredGridLayoutManager构造的第二个参数传一个orientation，如果传入的是StaggeredGridLayoutManager.VERTICAL代表有多少列；那么传入的如果是StaggeredGridLayoutManager.HORIZONTAL就代表有多少行。

##4.2 StaggeredAdapter

StaggeredAdapter写一个随机的高度来控制每个item的高度。

```
package com.example.michael.recyclerviewdemo;

import android.content.Context;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.List;

/**
 * 实现瀑布流的Adapter,增加一个高度列表，随机生成View的高度
 * Created by Michael on 2016/2/25.
 */
public class StaggeredAdapter extends RecyclerView.Adapter<StaggeredAdapter.MyViewHolder> {

    private Context mContext;
    private List<String> mDatas;

    private List<Integer> mHeights;

    private OnItemClickListener mListener;

    /**
     * 点击事件回调接口
     */
    public interface OnItemClickListener {

        void onItemClick(View view, int position);

        void onItemLongClick(View view, int position);
    }

    public void setOnItemClickListener(OnItemClickListener listener) {

        this.mListener = listener;
    }

    public StaggeredAdapter(Context context, List<String> datas) {

        this.mContext = context;
        mDatas = datas;

        mHeights = new ArrayList<Integer>();

		//生成随机高度
        for(int i = 0;i < mDatas.size();i++) {

            mHeights.add((int) (100 + Math.random() * 300));
        }
    }

    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {

        View view = LayoutInflater.from(mContext).inflate(R.layout.item_recyclerview, parent, false);

        return new MyViewHolder(view);
    }

    @Override
    public void onBindViewHolder(final MyViewHolder holder, final int position) {

		//设置Item的高度
        ViewGroup.LayoutParams layoutParams = holder.itemView.getLayoutParams();
        layoutParams.height = mHeights.get(position);
        holder.itemView.setLayoutParams(layoutParams);


        holder.mClickBtn.setText(mDatas.get(position));

        if (mListener != null) {

            holder.mClickBtn.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {

                    //获取实际的位置
                    int pos = holder.getLayoutPosition();

                    mListener.onItemClick(holder.mClickBtn,pos);

                }
            });

            holder.mClickBtn.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {

                    int pos = holder.getLayoutPosition();

                    mListener.onItemLongClick(holder.mClickBtn,pos);

                    return true;  //屏蔽单击  false点击事件仍会响应
                }
            });

        }
    }

    @Override
    public int getItemCount() {

        return mDatas.size();

    }

    /**
     * 在当前Item之前添加Item
     */
    public void addPreItem(String data,int position) {

        mDatas.add(position,data);
        mHeights.add( (int) (100 + Math.random() * 300));
        notifyItemInserted(position);
    }

    /**
     * 在当前Item之后添加Item
     */
    public void addNextItem(String data,int position) {

        mDatas.add(position+1,data);
        mHeights.add((int) (100 + Math.random() * 300));
        notifyItemInserted(position + 1);
    }

    /**
     * 删除Item
     * @param position
     */
    public void removeItem(int position) {
        mDatas.remove(position);
        mHeights.remove(position);
        notifyItemRemoved(position);
    }

    class MyViewHolder extends RecyclerView.ViewHolder {

        TextView mClickBtn;

        public MyViewHolder(View itemView) {
            super(itemView);

            mClickBtn = (TextView) itemView.findViewById(R.id.btn_click);
        }
    }
}

```

##4.3 效果

![ALT TEXT](http://img.blog.csdn.net/20160226170313991)


#5.修改分割线

DividerItemDecoration和DividerGridItemDecoration的实现类都是通过读取系统主题中的android.R.attr.listDivider来作为Item间的分割线，并且支持纵向和横向，获取到listDivider熟悉后，该属性的值是个Drawable，在getItemOffsets中，outRect去设置了绘制的范围，onDraw中实现了真正的绘制。

该分割线是系统默认的，你可以在theme.xml文件中找到该属性的使用情况。那么，使用系统的listDivide有什么好处呢？就是方便我们去随意的改变，该属性我们可以直接声明在：

```
 <!-- Application theme. -->
    <style name="AppTheme" parent="AppBaseTheme">
      <item name="android:listDivider">@drawable/divider_bg</item>  
    </style>
```

然后自己写个Drawable即可，下面我们换一种分隔符

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle" >

    <gradient
        android:centerColor="#ff00ff00"
        android:endColor="#ff0000ff"
        android:startColor="#ffff0000"
        android:type="linear" />
    <size android:height="4dp"/>

</shape>
```

效果如下:

![ALT TEXT](http://img.blog.csdn.net/20150415150026088)


#参考资料

[Android RecyclerView 使用完全解析 体验艺术般的控件](http://blog.csdn.net/lmj623565791/article/details/45059587#t7)
[Android5.x RecyclerView 应用解析](http://blog.csdn.net/itachi85/article/details/50036285)

