title: Android Tips
date: 2014-08-06 15:14:12
tags: [Android, Tips]

---

这篇文章会集合 Android 开发过程中遇到的奇怪的各种坑与小技巧。持续更新。

<!-- more -->

#   Android 4.4 及以下版本不支持 StateListDrawable 与 ColorFilter 同时使用

当需要让按钮在按下的过程中显示不一样状态的情况下，我们可以使用 StateListDrawable。而需要构造具有多种不同背景色的按钮时，我们可以使用 ColorFilter 进行实现。但是在 Android 4.4 及以下版本中，这两者不能共存。如果两个一起使用的话，ColorFilter 会失效。

要解决该问题，可以先将需要使用 ColorFilter 的 Drawable 变成一个 BitMapDrawable，然后再传入 StateListDrawable 中。详细代码见 [ref：statckoverflow](https://stackoverflow.com/questions/28717831/setcolorfilter-broken-on-android-4-working-on-android-5/32048294)。 但是看起来这个方案效率太低，考虑到 4.4 及以下的设备较少，可以考虑只使用 ColorFilter 实现背景色的效果，剔除掉 StateListDrawable 的效果。

# Update Android Sdk by Command Line ( update 17.07.20 )

## 1. 更新已经安装的 sdk

```
# 拉取待更新列表，前面会有序号
android-sdk-path/tools/android -v list sdk
# 根据上面的序号进行更新。 -u 表示命令行输出
android-sdk-path/tools/android -v update sdk -u -t 1,2,3,4 
```

## 2. 安装新的 sdk

需要在执行上面命令的时候加入 -a 参数

```
# 拉取待更新列表，前面会有序号
android-sdk-path/tools/android -v list sdk -a
# 根据上面的序号进行更新。 -u 表示命令行输出
android-sdk-path/tools/android -v update sdk -a -u -t 1,2,3,4 
```

## 3. 安装 Constraint Layout

目前（17.07.20）未在 android-sdk-manager 的默认配置中发现 Contraint-layout 包，因此需要手动指定，通过下列命令安装

`android-sdk-path/tools/bin/sdkmanager "extras;m2repository;com;android;support;constraint;constraint-layout;1.0.2"`

### Ref
- [Linux环境下用命令更新Android SDK](http://blog.csdn.net/csusunxgg/article/details/9703789)
- [How to install old version of Android build tools from command line?](https://stackoverflow.com/questions/26016770/how-to-install-old-version-of-android-build-tools-from-command-line)
- [Stackoverflow : How to install android constraint layout tools outside of Android Studio by using the command line?](https://stackoverflow.com/questions/39252102/how-to-install-android-constraint-layout-tools-outside-of-android-studio-by-usin)


#   Android Sp Pt 转换工具

http://angrytools.com/android/pixelcalc/

另外有一篇设计师写的关于 Android 字体的文章：[Android系统字体规范与应用探索](http://www.shejidaren.com/android%E7%B3%BB%E7%BB%9F%E5%AD%97%E4%BD%93%E8%A7%84%E8%8C%83%E4%B8%8E%E5%BA%94%E7%94%A8%E6%8E%A2%E7%B4%A2.html)

里面写的内容有不少不太准确，但是是我找到的少有的从设计师角度去分析 Android 字号相关的文章。**不能**作为实际使用时的标准，但可以了解一下。


#	GridView 中使用正方形布局

http://blog.chengyunfeng.com/?p=465
重写 onMeasure 方法，将其高度设置为宽度。而宽度是可以去动态变化的。

#	Enable Scroll in ScrollView

比如 EditText，经常会需要在 ScrollView 或者 ListView 中实现滚动效果，需要添加如下代码。

	EditText EtOne = (EditText) findViewById(R.id.comment1);
	EtOne.setOnTouchListener(new OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                if (v.getId() == R.id.comment1) {
                    v.getParent().requestDisallowInterceptTouchEvent(true);
                    switch (event.getAction() & MotionEvent.ACTION_MASK) {
                    case MotionEvent.ACTION_UP:
                        v.getParent().requestDisallowInterceptTouchEvent(false);
                        break;
                    }
                }
                return false;
            }
        });

#	Android 中文字符串排序

http://blog.csdn.net/luoboo525/article/details/8594561

http://blog.csdn.net/p106786860/article/details/9811929

#	远程连接 ADB

1.	用 USB 连接手机，开启调试模式
2.	adb tcpip 5555
3.	拔掉数据线。
4.	adb connect 192.168.1.3:5555

其中端口号任选，IP地址为手机所在的IP地址。

[原文](http://j796160836.pixnet.net/blog/post/29108155-%5Bandroid%5D-debug%E4%B8%8D%E7%94%A8%E7%B7%9A%EF%BC%8C%E7%94%A8adb%E9%80%A3%E6%8E%A53g-wifi%E6%89%8B%E6%A9%9F)

[官方文档](http://developer.android.com/guide/topics/connectivity/usb/index.html)

#	Github Rebase

B分支对A分支用过Rebase之后，如果以后还想对A分支Rebase，就不能对C分支再Rebase了。

#	Android Studio 导入第三方依赖包

http://www.cnblogs.com/neozhu/p/3458759.html

#	依赖注入

Roboguice / 依赖注入 / @InjectView / ...

http://www.cnblogs.com/keyindex/p/3366666.html	

#	ListView

##	getChildAt(index)

在 `getChildAt(index)` 方法中，index 是当前可视范围中的位置。  
通常使用 `position - getFirstVisiblePosition()` 作为 index 参数。

##	HeaderView position

[当 ListView 有 Header 时， onItemClick 里的 position 不正确](http://blog.chengbo.net/2012/03/09/onitemclick-return-wrong-position-when-listview-has-headerview.html)

应以如下方式调用：

	@Override
	public void onItemClick(AdapterView<?> parent, View v, int position, long id) {
	    doSomething(parent.getAdapter().getItem(position));
	}

#	通过 return 在测试过程中避免执行后续的语句

如果在一个方法中，需要避免后续语句的执行，直接在前面加 return 是不行的，因为这样会报错。但用 `if (true) return;` 就可以了。