title: 为 Android 应用添加 RTL 语言支持
date: 2017-02-25 13:41:18
tags: [Android]
---

对于一款面向全球用户的 Android 应用来说，进行本地化处理是十分重要的。关于本地化涉及的全部内容，可以参考 [Google Developer - Localization Checklist](https://developer.android.com/distribute/tools/localization-checklist.html) 。这里只介绍 RTL 布局中涉及的部分内容。

## 什么是 RTL 布局

我们平常都是以从左到右（Left-to-Right，LTR）的方向显示文本，但是对于阿拉伯语、希伯来语等语言，他们的文字是从右向左排列的，称为 Right-to-Left 布局，简称为 RTL。当我们处理 RTL 语言的时候，需要特别注意的一点是：在 RTL 语言中，并非仅有文本框的内容需要从右到左排列，而是整个页面的内容都要镜像过来，才符合 RTL 用户的使用习惯。

<!-- more -->

我们可以从 iOS 10 的介绍页面上比较一下 RTL 和 LTR 布局的区别。除了所有的文本框从右到左之外，包括状态栏、Tab bar 在内的所有内容都是反向的，甚至有部分能够表达方向含义的图标，如电量和返回键的图标也是反着的。

![阿拉伯语与中文的 iOS 10 介绍页面](/img/RTL-Support-iOS-ar-vs-cn.png)

*从这两幅图中容易看出 RTL 布局与 LTR 布局在大多数元素上都是镜像的效果。*


## Android 对 RTL 布局的支持情况

Android 系统从 4.2 开始引入了对双向文本的完整支持，如果应用支持的 minSdkVersion >= 17 ，则可以很轻易的实现 RTL 效果。由于本人并没有特地为 4.1 及以下设备调试过 RTL 的支持，因此以下内容主要以 4.2 版本进行讲述。

## 如何实现 RTL 布局

实现 RTL 布局的核心思想，就是将所有 Left / Right 这种绝对方向换为 Start / End 这种相对方向。

注：以下内容主要是以 `minSdkVersion = 17` 的情况下进行描述，并且是以 “为一个现有应用添加 RTL 支持” 的视角进行描述的。新创建应用的话当然亦可参考，但对于 `minSdkVersion < 17` 的情况就需要注意了，许多地方要根据设备的系统版本进行额外判断。

### 1. 在 AndroidManifest.xml 标明 `supportsRtl="true"`

首先在 AndroidManifest.xml 文件中的 application 节点内添加 `android:supportsRtl="true"` ，向系统声明当前应用能够支持 RTL 布局，这样当系统语言修改为阿拉伯语等从右到左布局时，应用的布局内容会随之发生改变。

### 2. 在 layout 文件中出现的所有 left / right 属性替换为 start / end

包括但不限于 marginLeft / paddingLeft / alignLeftOf 等内容。通过 Android Studio 的重构功能可以简单的完成这一步骤，如下图：

![利用 Android Studio 的重构功能实现 RTL Support](/img/RTL-Support-refactorInAs.png)

### 3. 需要将代码中包含绝对方向设置的代码改为相对方向

思路跟 layout 中的修改一样。Android Studio 的重构功能似乎只能影响 layout 文件，不能处理代码中的这种内容。因此我们需要手动进行修改，需要注意以下几方面

- 需要将 setMargins(left, top, right, bottom) 方法改为 setMarginStart(), setMarginEnd() 等方法。（这里很奇怪的是，从 SDK 的源码中可以看到，ViewGroup.MaringLayoutParam 类中提供了 setMarginsRelative 方法并设置了 hide 标注，不知道为什么不公开出来）
- TextView 的 setCompoundDrawableXXX 系列方法统一改为 setCompoundDrawableRelativeXXX 系列方法
- 其他内容大多可以通过代码全文搜索 LEFT 以及 RIGHT 字符串之后再单独处理

### 4. 部分资源需要提供 rtl 版本

如果有表示方向作用的图片资源，需要创建 -ldrtl 后缀的响应文件夹，并将提供的资源放到里面。至于有哪些图片需要修改，最典型的是返回箭头。

### 5. 将原生 ViewPager 替换为支持 RTL 布局的第三方 ViewPager

系统原生 ViewPager 有个经典 bug：在 RTL 布局中，其操作手势与滑动效果是反向的。

比如我们先考虑 LTR 布局的三个页面： 1-2-3 ，在 RTL 布局中应该变为 3-2-1。通常当我们处于第 2 个页面的时候，手指从屏幕右侧往左滑动，页面会进入右侧的页面，也就是 LTR 的 3 号页面或 RTL 的 1 号页面。而因为系统原生 ViewPager 的 Bug，导致 RTL 布局中，手指向左滑动时页面也左滑，效果就很怪异。相关的讨论可以参照 [Android Issue](https://code.google.com/p/android/issues/detail?id=56831) 中的讨论，这个问题在 2013 年就被提出来过，但一直没有得到有效地修复。

我们可以通过用第三方的 ViewPager 控件替换掉系统 ViewPgaer 的方式修复掉此问题。目前我在使用的库来自 [https://github.com/ksloginov/RtlViewPager](https://github.com/ksloginov/RtlViewPager) ，使用方式足够简单且有效，能够与 TabLayout 相结合。

### 6. 考察 TextView 的宽度能否用 wrap\_content 替换 match\_parent

系统提供的 TextView 在显示文字时是原生支持 RTL 布局的，也就是说，即使上面的内容都未实现，只是放两个全屏的 TextView，分别显示 LTR 语言文字和 RTL 语言文字，其显示方向也是不同的。反过来考虑，即使上面的内容都实现了，TextView 也是要单独考察的控件。为什么在系统对 TextView 的 RTL 拥有原生支持情况下仍然要对其特别关注呢？

因为即使在 RTL 语言中，也不能保证不出现 LTR 语言内容。我们考虑这样的情况：有两个 TextView，都设置了 `layout_width="match_parent"` 属性，且其中的文字内容都足够短，不能撑满屏幕。那么当这两个 TextView 一个显示 LTR 语言，一个显示 RTL 语言时，会是什么效果呢？

答案如下图左方所示，LTR 语言会在文本框中居左显示，RTL 则居右，导致内容一左一右，非常难看。而将 width 改为 wrap\_content 后则会如下右图样式，尽管控件内的文字显示顺序遵循了 LTR，但控件本身仍然会受到系统整体 RTL 的控制贴紧右侧。

![TextView 的 layout_width 属性会在 LTR 与 RTL 混排情况下影响显示](/img/RTL-Support-textview-width.png)

至于如何在应用内找到使用了 match\_parent 作为宽度属性的 TextView，我们可以通过 Android Studio 的 find in path 功能进行查找。通过构造正则表达式，将其全部列举出来并不难。问题在于不能全局统一的进行替换，而是要每个地方单独考察并尝试替换。如果页面需求的确需要 match\_parent 作为宽度那也就没办法了。

我在 find in path 时候使用的表达式，不一定准确： `<TextView[^>]*?android:layout_width="(match_parent|fill_parent)"[^>]*?/>` 。 需要反选 Case sensitive，勾选 Regular expression，上下文为 anywhere。

----

## 参考内容

同文中链接内容