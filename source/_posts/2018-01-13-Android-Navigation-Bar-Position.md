---
title: Android 虚拟按键在不同版本上的位置问题
date: 2018-01-13 11:42:47
tags: [Android]
---

> 当 Android 手机设备进行横屏状态时，其虚拟按键的位置在不同版本上位置可能不同。

<!-- more -->

# 问题描述

应用里面为了实现全局的右滑返回效果而引入了一个 SwipeBack 库 （[GitHub - SwipeBackLayout](https://github.com/ikew0ng/SwipeBackLayout)），然后将基类的 Activity 直接继承自 SwipeBackActivity。这之后就发现在 Android 7.0 的手机上横屏时，整个布局都向屏幕左侧移动了一些，导致左边被虚拟按键给挡住了。效果如下图：

![左边被挡住了](/img/Android-Navigation-Bar/Android-Navigation-Bar-1.png)

经反复测试后发现，只有在 Android 7.0 及以上版本的手机设备上，将设备顺时针旋转 90 度后才会出现该问题。如果是逆时针旋转、6.0 及以下设备、或者平板设备都不会有这个问题。这是因为从 Android 7.0 开始，虚拟按键的位置在横屏状态下发生了变化。在以前版本的手机设备中，横屏时的虚拟按键总是在屏幕右侧，而 Android 7.0 将虚拟按键在横屏状态下的位置改为了“总是接近设备底部”。就是说，如果手机顺时针旋转 90 度，此时设备底部在左侧，虚拟按键也就随之出现在左侧。而如果是逆时针旋转 90 度则在右侧，跟以前一样。

见下图，上面是 Android 6.0 设备，下图是 7.0。

![对比图](/img/Android-Navigation-Bar/Android-Navigation-Bar-2.png)

此外，平板设备上因为虚拟按键总在屏幕底部，所以也不存在该问题。

# 解决方案

这个 SwipeBackActivity 的实现方式是在 `Activity.onPostCreate` 的时候，获取布局根层的 DecorView，将其子 view 作为 swipeBackLayout 的子 view，然后再将 swipeBackLayout 插进 DecorView 中。从而做到在不修改 layout 文件的情况下将 swipeBackLayout 变成根 view。从这个实现方式上很显然可以猜到，这个替换的过程导致了布局的异常。因此要解决该问题，容易想到的一个方式就是直接在每个 acitivty 的布局里面写上 swipeBackLayout，不使用这种 onPostCreate 时强行修改根布局的方式即可。但是那样的话要修改的地方太多，我们还是希望在原先的基础上做尽量少的改动将问题解决。因此我们考虑在 SwipeBackLayout 上将布局移动到正确的位置。

首先是考虑如何获取虚拟按键当前在屏幕上的位置，在 [AndroidXRef](http://androidxref.com/7.0.0_r1/xref/frameworks/base/core/java/com/android/internal/policy/DecorView.java#998) 上从 Android 源码一层层跟进去之后发现实现过于复杂，因此直接用现有的信息进行条件判断： Android 7.0 以上、手机设备，顺时针旋转 90 度。

知道了虚拟按键的位置之后就要计算虚拟按键的高度。我们可以通过 WindowManager 中 getdefaultDisplay 之后的 `getRealSize` 和 `getSize` 这两个方法进行计算。

最后主要的代码如下：

```
    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        mInLayout = true;
        if (mContentView != null) {
            mContentLeft = getLeftPadding(mActivity);
            mContentView.layout(mContentLeft, mContentTop,
                    mContentLeft + mContentView.getMeasuredWidth(),
                    mContentTop + mContentView.getMeasuredHeight());
        }
        mInLayout = false;
    }

    private int getLeftPadding(Context context) {

        // 1. Android Version >= 7.0
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.N) {
            return 0;
        }

        // 2. Is Mobile Phone
        if (context.getResources().getConfiguration().smallestScreenWidthDp >= 600) {
            return 0;
        }

        // 3. is 270 rotation
        int screenAngle = getScreenRotationAngle(context);
        if (screenAngle != Surface.ROTATION_270) {
            return 0;
        }

        // 4. get navigation height
        return getNavigationBarHeight(context);
    }

    public int getNavigationBarHeight(Context context) {
        WindowManager wm = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
        if (wm == null) {
            return 0;
        }

        Display display = wm.getDefaultDisplay();

        Point devicePoint = new Point();
        display.getRealSize(devicePoint);

        Point appPoint = new Point();
        display.getSize(appPoint);

        if (appPoint.x < devicePoint.x) { // landscape
            return devicePoint.x - appPoint.x;
        } else if (appPoint.y < devicePoint.y) { // portrait
            return devicePoint.y - appPoint.y;
        }

        return 0;
    }

    public int getScreenRotationAngle(Context context) {
        WindowManager wm = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
        if (wm == null) {
            return Surface.ROTATION_0;
        }

        Display display = wm.getDefaultDisplay();
        return display.getRotation();
    }


```

BTW，__设备__顺时针旋转 90 度意味着__屏幕上的内容__要顺时针旋转 270 度，因此上面对于旋转角度的判断是 ROTATION_270 而非 ROTATION_90。

# Ref

- [Detect Android Navigation Bar orientation](https://stackoverflow.com/questions/21057035/detect-android-navigation-bar-orientation)
- [How to REALLY get the navigation bar height in Android](https://stackoverflow.com/questions/36514167/how-to-really-get-the-navigation-bar-height-in-android)
