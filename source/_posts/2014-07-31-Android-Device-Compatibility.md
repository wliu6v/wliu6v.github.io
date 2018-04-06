title: Android 分辨率适配
date: 2014-07-31 01:57:45
tags: [Android]

---

## ldpi / mdpi / hdpi / xhdpi / xxhdpi

密度。means how much pixels can fit into a single inch.


    ldpi = 1:0.75
    mdpi = 1:1
    hdpi = 1:1.5
    xhdpi = 1:2
    xxhdpi = 1:3
    xxxhdpi = 1:4

<!-- more -->

如果一个图片是 100*100  的话

    for mdpi it should be 100X100
    for ldpi it should be 75X75
    for hdpi it should be 150X150
    for xhdpi it should be 200X200
    for xxhdpi it should be 300X300
    for xxxhdpi it should be 400X400


##	-w / -sw

设备尺寸。

-sw : smallest width。指当前屏幕的最短的边。不随屏幕方向而变。

-w : width。指屏幕的宽度。会随着屏幕方向变化而变。

-swNNNdp indicates "use these resources if A is greater than or equal to NNN dp in length"

-wNNNdp indicates "use these resources if the width of the device, as presently held, is greater than or equal to NNN dp"

通常来说，设置 3 种尺寸就足够了。

```
res/layout/main_activity.xml           # For handsets (smaller than 600dp available width)
res/layout-sw600dp/main_activity.xml   # For 7” tablets (600dp wide and bigger)
res/layout-sw720dp/main_activity.xml   # For 10” tablets (720dp wide and bigger)
```

PS: -h 与 -w 同理。

## -land / -port

横竖屏

##	-ldrtl / -ldltr

ld stands for layout direction. 像阿拉伯文字等，就算是 Right-to-Left 的布局。默认为 ldltr

-ldrtl : Layout Direction is Right-to-Left

-ldltr : Layout Direction is Left-to-Right

PS： 要使用此值，需 SDK 17 或以上。且需设置 supportsRtl。

##	-large

针对于 Android 3.2 之前版本。在 3.2 之后应使用 -sw600dp.

##	-finger / -notouch

有无触摸屏

##	-en / -fr / -en-rUS / ...

Language and region。

基本组成是：两位小写语言代码 + `-r` + 两位大写区域代码。 
语言代码使用 ISO 639-1 标准中的代码。
区域代码，采用 ISO 3166-1 标准中的代码。
[ISO 639-1](http://zh.wikipedia.org/wiki/ISO_639-1)
[ISO 3166-1](http://zh.wikipedia.org/wiki/ISO_3166-1)

##	-long / -notlong

long: Long screens, such as WQVGA, WVGA, FWVGA
notlong: Not long screens, such as QVGA, HVGA, and VGA

##	-car / -desk / ....


-	car: Device is displaying in a car dock
-	desk: Device is displaying in a desk dock
-	television: Device is displaying on a television, providing a "ten foot" experience where its UI is on a large screen that the user is far away from, primarily oriented around DPAD or other non-pointer interaction
-	appliance: Device is serving as an appliance, with no display
-	watch: Device has a display and is worn on the wrist

Added in API level 8, television added in API 13, watch added in API 20.

##	-night / -notnignt

-	night: Night time
-	notnight: Day time


##	通常可能用到的组合

-xhdpi , -sw600dp (for tablet) , -land (for Landscape)

## Ref

- [http://theground.jimdo.com/android/android-screen-size-ldpi-mdpi-hdpi/](http://theground.jimdo.com/android/android-screen-size-ldpi-mdpi-hdpi/)

- [About finger](http://stackoverflow.com/questions/1677346/difference-between-layout-finger-and-layout-directory)

- [Android 自适应多屏幕支持](http://xiaomi4980.blog.163.com/blog/static/215945196201391411840729/)

- [Android 常见分辨率](http://blog.csdn.net/sarsscofy/article/details/9249397)

- [官方文档](http://developer.android.com/guide/topics/resources/providing-resources.html)