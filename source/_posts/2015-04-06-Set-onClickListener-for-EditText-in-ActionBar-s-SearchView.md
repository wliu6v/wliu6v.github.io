title: "Set onClickListener for EditText in ActionBar's SearchView"
date: 2015-04-06 16:14:05
tags: [Android]

---

要为 ActionBar 上的 SearchView 的 EditText 设置 onClickListener，可以考虑通过递归的方式为 SearchView 的所有子 View / ViewGroup 设置 onClickListener。代码如下：

<!-- more -->

``` JAVA
public static void setOnClickListenerForViewGroup(View v, View.OnClickListener listener) {
	if (v instanceof ViewGroup) {
		ViewGroup group = (ViewGroup)v;
		int count = group.getChildCount();
		for (int i = 0; i < count; i++) {
			View child = group.getChildAt(i);
			if (child instanceof LinearLayout || child instanceof RelativeLayout) {
				setOnClickListenerForViewGroup(child, listener);
			}

			if (child instanceof TextView) {
				TextView text = (TextView)child;
				text.setFocusable(false);
			}
			child.setOnClickListener(listener);
		}
	}
}
```

当然也可以通过类似的方式获取到 EditText 对象，并对其设置 onClickListener() 

``` JAVA
public static EditText getEditTextFromSearchView(View v) {
	EditText et = null;
	if (v instanceof ViewGroup) {
		ViewGroup group = (ViewGroup)v;
		int count = group.getChildCount();
		for (int i = 0; i < count; i++) {
			View child = group.getChildAt(i);
			if (child instanceof LinearLayout || child instanceof RelativeLayout) {
				EditText tmpEt = getEditTextFromSearchView(child);
				if (tmpEt != null) {
					et = tmpEt;
				}
			}

			if (child instanceof EditText) {
				et = (EditText) child;
			}
		}
	}
	return et;
}
```