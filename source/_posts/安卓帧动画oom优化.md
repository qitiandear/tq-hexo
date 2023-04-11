---
title: 安卓帧动画oom优化
date: 2023-04-11 13:43:07
tags:
  - Android
categories:
  - 帧动画
---

帧动画非常容易理解，其实就是简单的由N张静态图片收集起来，然后我们通过控制依次显示 这些图片，因为人眼"视觉残留"的原因，会让我们造成动画的"错觉"，跟放电影的原理一样！

而Android中实现帧动画，一般我们会用到前面讲解到的一个Drawable：AnimationDrawable 先编写好Drawable，然后代码中调用start()以及stop()开始或停止播放动画~

当然我们也可以在Java代码中创建逐帧动画，创建AnimationDrawable对象，然后调用 addFrame(Drawable frame,int duration)向动画中添加帧，接着调用start()和stop()而已~

## 普通实现

实现一个帧动画，最先想到的就是用animation-list将全部图片按顺序放入，并设置时间间隔和播放模式。然后将该drawable设置给ImageView或Progressbar就OK了。
首先创建帧动画资源文件drawable/anim.xml，oneshot=false为循环播放模式，ture为单次播放；duration为每帧时间间隔，单位毫秒。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false">
    <item
        android:drawable="@mipmap/compose_00000"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00001"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00002"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00003"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00004"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00005"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00006"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00007"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00008"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00009"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00010"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00011"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00012"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00013"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00014"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00015"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00016"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00017"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00018"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00019"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00020"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00021"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00022"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00023"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00024"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00025"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00026"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00027"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00028"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00029"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00030"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00031"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00032"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00033"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00034"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00035"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00036"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00037"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00038"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00039"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00040"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00041"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00042"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00043"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00044"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00045"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00046"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00047"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00048"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00049"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00050"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00000"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00051"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00052"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00053"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00054"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00055"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00056"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00057"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00058"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00059"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00060"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00061"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00062"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00063"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00064"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00065"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00066"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00067"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00068"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00069"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00070"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00071"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00072"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00073"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00074"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00075"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00076"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00077"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00078"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00079"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00080"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00081"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00082"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00083"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00084"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00085"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00086"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00087"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00088"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00089"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00090"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00091"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00092"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00093"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00094"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00095"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00096"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00097"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00098"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00099"
        android:duration="33" />
    <item
        android:drawable="@mipmap/compose_00100"
        android:duration="33" />
</animation-list>
```

然后在activity_main.xml中写入

``` xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ImageView
        android:id="@+id/img_show"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@drawable/miao_gif" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

在MainActivity中写入

``` java
package com.roli.lunbo;

import androidx.appcompat.app.AppCompatActivity;

import android.graphics.drawable.AnimationDrawable;
import android.os.Bundle;
import android.widget.ImageView;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ImageView img_show = (ImageView) findViewById(R.id.img_show);
        AnimationDrawable anim = (AnimationDrawable) img_show.getBackground();
        anim.start();
    }
}
```

这样一个简单的帧动画就可以了

## OOM问题及优化

###  内存溢出咋办

用普通方法实现帧动画用到普通场景是没问题的，如果碰到几十甚至几百帧图片，而且每张图片几百K的情况，呵呵。例如，做一个很炫的闪屏帧动画，要保证高清且动作丝滑，就需要至少几十张高清图片。这时，OOM问题就出来了，闪屏进化成了一闪~

这里使用[StackOverflow](http://stackoverflow.com/questions/8692328/causing-outofmemoryerror-in-frame-by-frame-animation-in-android)提供的方法

### 解决思路

先分析下普通方法为啥会OOM，从xml中读取到图片id列表后就去硬盘中找这些图片资源，将图片全部读出来后按顺序设置给ImageView，利用视觉暂留效果实现了动画。一次拿出这么多图片，而系统都是以Bitmap位图形式读取的（作为OOM的常客，这锅Bitmap来背）；而动画的播放是按顺序来的，大量Bitmap就排好队等待播放然后释放，然而这个排队的地方只有10平米，呵呵~发现问题了吧。。

按照大神的思路，既然来这么多Bitmap，一次却只能临幸一个，那么就翻牌子吧，轮到谁就派个线程去叫谁，bitmap1叫到了得叫上下一位bitmap2做准备，这样更迭效率高一些。为了避免某个bitmap已被叫走了线程白跑一趟的情况，加个Synchronized同步下数据信息，实现代码如下：

``` java
public synchronized void start() {
    mShouldRun = true;
    if (mIsRunning)
        return;

    Runnable runnable = new Runnable() {
        @Override
        public void run() {
            ImageView imageView = mSoftReferenceImageView.get();
            if (!mShouldRun || imageView == null) {
                mIsRunning = false;
                if (mOnAnimationStoppedListener != null) {
                    mOnAnimationStoppedListener.AnimationStopped();
                }
                return;
            }

            mIsRunning = true;
            //新开线程去读下一帧
            mHandler.postDelayed(this, mDelayMillis);

            if (imageView.isShown()) {
                int imageRes = getNext();
                if (mBitmap != null) { // so Build.VERSION.SDK_INT >= 11
                    Bitmap bitmap = null;
                    try {
                        bitmap = BitmapFactory.decodeResource(imageView.getResources(), imageRes, mBitmapOptions);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    if (bitmap != null) {
                        imageView.setImageBitmap(bitmap);
                    } else {
                        imageView.setImageResource(imageRes);
                        mBitmap.recycle();
                        mBitmap = null;
                    }
                } else {
                    imageView.setImageResource(imageRes);
                }
            }

        }
    };

    mHandler.post(runnable);
}
```

### 进一步优化

为了快速读取SD卡中的图片资源，这里用到了Apache的IOUtils。在位图处理上使用了BitmapFactory.Options()相关设置,InBitmap,当图片大小类型相同时，虚拟机就对位图进行内存复用，不再分配新的内存，可以避免不必要的内存分配及GC。

``` java
if (Build.VERSION.SDK_INT >= 11) {
    Bitmap bmp = ((BitmapDrawable) imageView.getDrawable()).getBitmap();
    int width = bmp.getWidth();
    int height = bmp.getHeight();
    Bitmap.Config config = bmp.getConfig();
    mBitmap = Bitmap.createBitmap(width, height, config);
    mBitmapOptions = new BitmapFactory.Options();
    //设置Bitmap内存复用
    mBitmapOptions.inBitmap = mBitmap;//Bitmap复用内存块
    mBitmapOptions.inMutable = true;//解码时返回可变Bitmap
    mBitmapOptions.inSampleSize = 1;//缩放比例
}
```

### 实现过程

复制粘贴实现功能，[具体代码](https://gitee.com/qitiandear/oomlunbo)

使用很简单：

``` java
/**
 * 将帧动画资源id以字符串数组形式写到values/arrays.xml中
 * FPS为每秒播放帧数，FPS = 1/T，（T--每帧间隔时间秒）
 */

AnimationsContainer.FramesSequenceAnimation animation 
        = AnimationsContainer.getInstance(R.array.XXX, FPS).createProgressDialogAnim(imageView);

animation.start();//动画开始
animation.stop();//动画结束
```

注意图片资源ID需要以String数组形式放入xml中，然后再利用TypedArray将字符串转为资源ID。如果直接用@drawable/img1这样的形式放入Int数组中，是没法读取到正真的资源ID的。

从xml中读取资源ID数组代码：

``` java
/**
 * 从xml中读取帧数组
 * @param resId
 * @return
 */
private int[] getData(int resId){
    TypedArray array = mContext.getResources().obtainTypedArray(resId);

    int len = array.length();
    int[] intArray = new int[array.length()];

    for(int i = 0; i < len; i++){
        intArray[i] = array.getResourceId(i, 0);
    }
    array.recycle();
    return intArray;
}
```