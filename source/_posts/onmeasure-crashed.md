title: onmeasure crashed
date: 2014-08-12 00:30:24
tags: [Android]

---

RelativeLayout 在 Android 4.1.1 及其之前版本执行 measure 有时会 crash。

可尝试在 inflate 的时候指定其 parentView 参数。

详见 http://blog.csdn.net/wangfei584521/article/details/25987377 中的 solution 1