---
layout:     post
title:      WeakReference摘要
subtitle:   学习笔记
date:       2018-11-29
author:     HelloMin
header-img: img/post-bg-lemon.jpg
catalog: true
tags:
    - Java
---

> 阅读《Effective Java》的时候读到了weakreference，想起之前曾经做过相关的代码优化，但当时并不太清楚自己在做什么，于是找了些文章读了一遍，总结在这里，方便自己日后查看。

## 为什么要用弱引用
1. 防止内存泄露
2. 清理缓存

## 什么是内存泄露
Java使用有向图机制，通过GC自动检查内存中的对象（什么时候检查由虚拟机决定），如果GC发现一个或一组对象为不可到达状态，则将该对象从内存中回收。也就是说，一个对象不被任何引用所指向，则该对象会在被GC发现的时候被回收；另外，如果一组对象中只包含互相的引用，而没有来自它们外部的引用（例如有两个对象A和B互相持有引用，但没有任何外部对象持有指向A或B的引用），这仍然属于不可到达，同样会被GC回收。

## Android中使用Handler造成内存泄露的原因

```
Handler mHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        mImageView.setImageBitmap(mBitmap);
    }
}
```
上面是一段简单的Handler的使用。当使用内部类（包括匿名类）来创建Handler的时候，Handler对象会隐式地持有一个外部类对象（通常是一个Activity）的引用（不然你怎么可能通过Handler来操作Activity中的View？）。而Handler通常会伴随着一个耗时的后台线程（例如从网络拉取图片）一起出现，这个后台线程在任务执行完毕（例如图片下载完毕）之后，通过消息机制通知Handler，然后Handler把图片更新到界面。然而，如果用户在网络请求过程中关闭了Activity，正常情况下，Activity不再被使用，它就有可能在GC检查时被回收掉，但由于这时线程尚未执行完，而该线程持有Handler的引用（不然它怎么发消息给Handler？），这个Handler又持有Activity的引用，就导致该Activity无法被回收（即内存泄露），直到网络请求结束（例如图片下载完毕）。
另外，如果你执行了Handler的postDelayed()方法，该方法会将你的Handler装入一个Message，并把这条Message推到MessageQueue中，那么在你设定的delay到达之前，会有一条MessageQueue -> Message -> Handler -> Activity的链，导致你的Activity被持有引用而无法被回收。

## 使用Handler导致内存泄露的解决方法

### 方法一：通过程序逻辑来进行保护。

1.在关闭Activity的时候停掉你的后台线程。线程停掉了，就相当于切断了Handler和外部连接的线，Activity自然会在合适的时候被回收。

2.如果你的Handler是被delay的Message持有了引用，那么使用相应的Handler的removeCallbacks()方法，把消息对象从消息队列移除就行了。

### 方法二：将Handler声明为静态类+WeakReference。
静态类不持有外部类的对象，所以你的Activity可以随意被回收。代码如下：
```
static class MyHandler extends Handler {
    @Override
    public void handleMessage(Message msg) {
        mImageView.setImageBitmap(mBitmap);
    }
}
```
但其实没这么简单。使用了以上代码之后，你会发现，由于Handler不再持有外部类对象的引用，导致程序不允许你在Handler中操作Activity中的对象了。所以你需要在Handler中增加一个对Activity的弱引用（WeakReference）
```
static class MyHandler extends Handler {
    WeakReference<Activity > mActivityReference;

    MyHandler(Activity activity) {
        mActivityReference= new WeakReference<Activity>(activity);
    }

    @Override
    public void handleMessage(Message msg) {
        final Activity activity = mActivityReference.get();
        if (activity != null) {
            mImageView.setImageBitmap(mBitmap);
        }
    }
}
```

## WeakReference的实现和使用
假设垃圾收集器在某个时间点决定一个对象是弱可达的(weakly reachable)（也就是说当前指向它的全都是弱引用），这时垃圾收集器会清除所有指向该对象的弱引用，然后垃圾收集器会把这个弱可达对象标记为可终结(finalizable)的，这样它们随后就会被回收。与此同时或稍后，垃圾收集器会把那些刚清除的弱引用放入创建弱引用对象时所登记到的引用队列(Reference Queue)中。

## ReferenceQueue的使用
初始化的时候传进去就可以了～
```
ReferenceQueue<String> rq = new ReferenceQueue<String>();
SoftReference<String> sr = new SoftReference<String>(s, rq);
while (k = rq.remove() != null) { // rq有内容
  // k被回收了
}
```

## 多种Reference的比较

| 引用类型 | 被垃圾回收时间 | 用途 | 生存时间 |
| ------ | ------ | ------ | -----|
| 强引用 | 从来不会 | 对象的一般状态 | JVM停止运行时终止 |
| 软引用 | 在内存不足时 |对象缓存 | 内存不足时终止 |
| 弱引用 |在垃圾回收时 | 对象缓存 | gc运行后终止 |
| 虚引用 | Unknown | Unknown | Unknown |

Refs:

[android中handler使用WeakReference防止内存泄露](https://blog.csdn.net/lanximu/article/details/40522367)

[理解Java中的弱引用](https://www.cnblogs.com/absfree/p/5555687.html)

[ReferenceQueue的使用](https://www.jianshu.com/p/73260a46291c)

[强引用、弱引用、软引用、虚引用](https://www.jianshu.com/p/1fc5d1cbb2d4)
