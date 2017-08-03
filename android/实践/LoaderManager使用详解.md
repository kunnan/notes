LoaderManager使用详解
==

# LoaderManager是什么？

LoaderManager用来负责管理与Activity或者Fragment联系起来的一个或多个Loaders对象。每个Activity或者Fragment都有唯一的一个LoaderManager实例，用来启动、停止、保持、重启、关闭它的Loaders。这些事件有时直接在客户端通过调用initLoader()/restartLoader()/destroyLoader()函数来实现。通常这些事件通过主要的Activity/Fragment声明周期事件来触发，而不是手动（当然也可以手动调用）。比如，当一个Activity关闭时（destroyed），改活动将指示它的LoaderManager来销毁并且关闭它的Loaders（当然也会销毁并关闭与这些Loaders关联的资源，比如Cursor）。  

LoaderManager并不知道数据如何装载以及何时需要装载。相反地，LoaderManager只需要控制它的Loaders们开始、停止、重置他们的Load行为，在配置变换（比如横竖屏切换）时保持loaders们的状态，并提供一个简单的接口来获取load结果到客户端中。



# 实现LoaderManager.LoaderCallbacks<D>接口

LoaderManager.LoaderCallbacks<D>接口LoaderManager用来向客户返回数据的方式。每个Loader都有自己的回调对象供与LoaderManager进行交互。该回调对象在实现LoaderManager中地位很高，告诉LoaderManager如何实例化Loader(onCreateLoader)，以及当载入行为结束或者重启（onLoadFinished或者onLoadReset）之后执行什么操作。大多数情况，你需要把该接口实现为组件的一部分，比如说让你的Activity或者Fragment实现LoadManager.LoaderCallbacks<D>接口。

```Java
public class SampleActivity extends Activity implements LoaderManager.LoaderCallbacks<D> {  

  public Loader<D> onCreateLoader(int id, Bundle args) { ... }  

  public void onLoadFinished(Loader<D> loader, D data) { ... }  

  public void onLoaderReset(Loader<D> loader) { ... }  

  /* ... */  
}  
```

一旦实现该接口，客户端将回调对象（本例中为“this”）作为LoaderManager的initLoader函数的第三个参数传输。
总的来说，实现回调接口非常直接明了。每个回调方法都有各自明确的与LoaderManager进行交互的目的：

- onCreateLoader是一个工厂方法，用来返回一个新的Loader。LoaderManager将会在它第一次创建Loader的时候调用该方法。
- onLoadFinished方法将在Loader创建完毕的时候自动调用。典型用法是，当载入数据完毕，客户端（译者注：调用它的Activity之类的）需要更新应用UI。客户端假设每次有新数据的时候，新数据都会返回到这个方法中。记住，检测数据源是Loader的工作，Loader也会执行实际的同步载入操作。一旦Loader载入数据完成，LoaderManager将会接受到这些载入数据，并且将将结果传给回调对象的onLoadFinished方法，这样客户端（比如Activity或者Fragment）就能使用该数据了。
- 最后，当Loader们的数据被重置的时候将会调用onLoadReset。该方法让你可以从就的数据中移除不再有用的数据。

# 示例

```Java
package com.michael.loadermanagerdemo;

import android.database.Cursor;
import android.os.Looper;
import android.provider.MediaStore;
import android.support.v4.app.LoaderManager;
import android.support.v4.content.CursorLoader;
import android.support.v4.content.FileProvider;
import android.support.v4.content.Loader;
import android.support.v4.content.PermissionChecker;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import com.orhanobut.logger.Logger;

public class MainActivity extends AppCompatActivity implements LoaderManager.LoaderCallbacks<Cursor>{

  private static final String[] PROJECTION = new String[] { MediaStore.Images.Media._ID, MediaStore.Images.Media.DATA };

  final String[] DOC_PROJECTION = {
      MediaStore.Files.FileColumns.DATA,
      MediaStore.Files.FileColumns.MIME_TYPE,
      MediaStore.Files.FileColumns.TITLE

  };

  private static final int LOADER_ID = 1;
  private static final int LOADER_FILE_ID = 2;

  LoaderManager mLoaderManager;

  @Override protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    Logger.init("loader");

    //1.获取LoadManager实例
    mLoaderManager = getSupportLoaderManager();
    //2.设置LoadManager传输的参数
    Bundle bundle = new Bundle();
    bundle.putString("liu","liuguoquan");
    //3.初始化LoadManager
    mLoaderManager.initLoader(LOADER_FILE_ID,bundle,this);
  }

  @Override public Loader<Cursor> onCreateLoader(int id, Bundle bundle) {
    Logger.d("bundle: " + bundle.getString("liu"));
    if (id == LOADER_ID) {
      //获取图片信息
      return new CursorLoader(MainActivity.this, MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
          PROJECTION, null, null, null);
    } else if (id == LOADER_FILE_ID) {
      //获取文件名
      return new CursorLoader(MainActivity.this,MediaStore.Files.getContentUri("external"),DOC_PROJECTION,null,null,null);
    }

    return null;
  }

  @Override public void onLoadFinished(Loader<Cursor> loader, Cursor cursor) {
    //创建完毕后调用
    switch (loader.getId()) {
      case LOADER_ID:

        while (cursor.moveToNext()) {
          Long id = cursor.getLong(0);
          String data = cursor.getString(1);
          Log.d("loader","id:" + id + "; data: " + data);
        }

        break;

      case LOADER_FILE_ID:

        while (cursor.moveToNext()) {
          String title =  cursor.getString(2);
          Log.d("loader","title: " + title);
        }
        cursor.close();
    }

  }

  @Override public void onLoaderReset(Loader<Cursor> loader) {
    //数据重置时调用
  }
}

```

[LoaderManager使用详解（三）---实现Loaders](http://blog.csdn.net/murphykwu/article/details/35288477)


