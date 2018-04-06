title: EditText 在进行物理键盘输入时失去焦点问题
date: 2014-08-20 14:15:10
tags: [Android]

---

当同屏存在自定义的 TabBar 的时候，偶尔会出现这样一个问题：当光标在 EditText 中，敲击物理键盘的任意键，EditText 会失去焦点。此时按方向键，发现 TabBar 发生了左右移动，说明焦点被其获取。

要解决此问题，只需添加如下语句：

	mTabHost.addOnAttachStateChangeListener(new OnAttachStateChangeListener() {

		    @Override
		    public void onViewDetachedFromWindow(View v) {}

		    @Override
		    public void onViewAttachedToWindow(View v) {
		        mTabHost.getViewTreeObserver().removeOnTouchModeChangeListener(mTabHost);
		    }
		});


问题来源： https://groups.google.com/forum/#!topic/android-developers/8_3R6xcoiIY
解决方法来源：https://code.google.com/p/android/issues/detail?id=2516 中的第16楼。