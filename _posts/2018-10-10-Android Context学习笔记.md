---
layout:     post
title:      Android Context学习笔记
subtitle:   
date:       2018-10-10
author:     HelloMin
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - Android
---
## Context继承关系
Context是一个抽象类，其中函数真正的实现在ContextImpl类。

ContextWrapper类是一个包装类，包含了一个真正的ContextImpl引用。

ContextThemeWrapper类包含了与ui相关的接口。
![context.png](https://upload-images.jianshu.io/upload_images/311072-16b6516ab6c15b58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一个程序包含的context数目 = activity数目+service数目+application数目（进程数）

## Context作用
弹出toast/启动activity/发广播/操作数据库...
```
TextView tv = new TextView(getContext());

ListAdapter adapter = new SimpleCursorAdapter(getApplicationContext(), ...);

AudioManager am = (AudioManager) getContext().getSystemService(Context.AUDIO_SERVICE);getApplicationContext().getSharedPreferences(name, mode);

getApplicationContext().getContentResolver().query(uri, ...);

getContext().getResources().getDisplayMetrics().widthPixels * 5 / 8;

getContext().startActivity(intent);

getContext().startService(intent);

getContext().sendBroadcast(intent);

```
## Context引起的内存泄露
#### 错误的单例模式
```
public class Singleton {
    private static Singleton instance;
    private Context mContext;

    private Singleton(Context context) {
        this.mContext = context;
    }

    public static Singleton getInstance(Context context) {
        if (instance == null) {
            instance = new Singleton(context);
        }
        return instance;
    }
}
```
instance作为静态对象，生命周期比普通对象长。即使activity被销毁，但它的引用还保存着，就不会被gc，造成内存泄漏。
#### View持有Activity引用
```
public class MainActivity extends Activity {
    private static Drawable mDrawable;

    @Override
    protected void onCreate(Bundle saveInstanceState) {
        super.onCreate(saveInstanceState);
        setContentView(R.layout.activity_main);
        ImageView iv = new ImageView(this);
        mDrawable = getResources().getDrawable(R.drawable.ic_launcher);
        iv.setImageDrawable(mDrawable);
    }
}
```
static的drawable间接引用了activity
#### 正确使用Context
1. 生命周期长的对象，优先使用application的context
2. 不要让生命周期长于activity的对象持有其引用
3. 尽量不要在Activity中使用非静态内部类，因为非静态内部类会隐式持有外部类实例的引用，如果使用静态内部类，将外部实例引用作为弱引用持有。
## 应用程序APP各种Context访问资源的唯一性分析
```
class ContextImpl extends Context {
    ......
    private final ResourcesManager mResourcesManager;
    private final Resources mResources;
    ......
    @Override
    public Resources getResources() {
        return mResources;
    }
    ......
}
```
上图看出，资源保存在resources变量中，而该变量的赋值如下：
```
private ContextImpl(ContextImpl container, ActivityThread mainThread,
            LoadedApk packageInfo, IBinder activityToken, UserHandle user, boolean restricted,
            Display display, Configuration overrideConfiguration) {
        ......
        //单例模式获取ResourcesManager对象
        mResourcesManager = ResourcesManager.getInstance();
        ......
        //packageInfo对于一个APP来说只有一个，所以resources 是同一份
        Resources resources = packageInfo.getResources(mainThread);
        if (resources != null) {
            if (activityToken != null
                    || displayId != Display.DEFAULT_DISPLAY
                    || overrideConfiguration != null
                    || (compatInfo != null && compatInfo.applicationScale
                            != resources.getCompatibilityInfo().applicationScale)) {
                //mResourcesManager是单例，所以resources是同一份
                resources = mResourcesManager.getTopLevelResources(packageInfo.getResDir(),
                        packageInfo.getSplitResDirs(), packageInfo.getOverlayDirs(),
                        packageInfo.getApplicationInfo().sharedLibraryFiles, displayId,
                        overrideConfiguration, compatInfo, activityToken);
            }
        }
        //把resources赋值给mResources
        mResources = resources;
        ......
    }
```

由此可以看出在设备其他因素不变的情况下我们通过不同的Context实例得到的Resources是同一套资源。

https://www.jianshu.com/p/94e0f9ab3f1d
https://blog.csdn.net/yanbober/article/details/45967639
