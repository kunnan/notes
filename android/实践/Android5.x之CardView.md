[TOC]

Android 5.x版本中增加了CardView控件，CardView继承自FrameLayout类，它的功能是实现在一个卡片布局中显示相同的内容，卡片布局可以设置圆角和阴影，还可以布局其他的View。CardView即可作为一般的布局使用，也可以作为RecyclerView的Item使用。

接来下，我们进入CardView学习之旅

#1.build.gradle

首先，和RecyclerView一样，导入v7兼容包

```
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.1.1'
    compile 'com.android.support:design:23.1.1'
    compile 'com.android.support:cardview-v7:23.1.1' //cardview
    compile 'com.android.support:recyclerview-v7:23.1.1'
    compile 'com.android.support:palette-v7:23.1.1'
}
```

#2.CardView

##2.1 布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:card_view="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="5dp">

    <!--android:foreground="?android:attr/selectableItemBackground"点击CardView时前景色水波纹效果-->

    <android.support.v7.widget.CardView
        android:id="@+id/cardview"
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:foreground="?android:attr/selectableItemBackground"
        card_view:cardCornerRadius="20dp"
        card_view:cardElevation="20dp"
        card_view:contentPadding="10dp">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="#585858"
            android:orientation="horizontal">


            <ImageView
                android:id="@+id/iv_image"
                android:layout_width="150dp"
                android:layout_height="150dp"
                android:layout_gravity="center_vertical"
                android:src="@drawable/mao" />


            <TextView
                android:id="@+id/tv_msg"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_gravity="center_vertical"
                android:gravity="center_vertical"
                android:text="123456789" />

        </LinearLayout>


    </android.support.v7.widget.CardView>

    <Button
        android:id="@+id/btn_click"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</LinearLayout>
```

card_view:cardCornerRadius="20dp"： 设置CardView的圆角半径

card_view:cardElevation="20dp"： 设置CardView的阴影半径
card_view:contentPadding="10dp"： 设置CardView中子控件和父控件的距离

##2.2 java代码调用

```
package com.example.michael.recyclerviewdemo;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.CardView;
import android.view.View;
import android.widget.Button;
import android.widget.TextView;
import android.widget.Toast;

/**
 * Created by Michael on 2016/2/25.
 */
public class CardViewActivity extends AppCompatActivity {

    private CardView mCardView;
    private Button mButton;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_cardview);

        mButton = (Button) findViewById(R.id.btn_click);
        mCardView = (CardView) findViewById(R.id.cardview);

        mCardView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                Toast.makeText(CardViewActivity.this, "click", Toast.LENGTH_SHORT).show();

            }
        });

        //找到CardView中的子View
        TextView mTextMsg = (TextView) mCardView.findViewById(R.id.tv_msg);
        mTextMsg.setText("text msg");

        mCardView.setRadius(10.0f);  //设置CardView的圆角半径
        mCardView.setCardElevation(10.0f); //设置CardView的阴影半径
        mCardView.setContentPadding(5,5,5,5); //设置CardView的父控件与子控件的距离


    }
}

```

##2.3 运行效果

![cardView](http://img.blog.csdn.net/20160227214617558)

#参考资料

[Android5.x CardView 应用解析](http://blog.csdn.net/itachi85/article/details/50067127)

