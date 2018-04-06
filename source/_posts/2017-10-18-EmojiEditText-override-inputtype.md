---
title: EmojiEditText 控件会覆盖 XML 中定义的 inputType 属性
date: 2017-10-18 14:58:47
tags: [Android]
---


最近项目中引入了 support 26 中的 EmojiEditText (`com.android.support:support-emoji:26.1.0`)，我们自定义的 BaseEditText 也继承了 EmojiEditText。后来突然发现，在使用原生键盘输入密码的时候会出现单词联想提示了。而按理说，在 xml 里面设置了 `inputType="textPassword" ` 的话，是不应该有联想提示的。

<!-- more -->

![正常的密码输入键盘与使用了 EmojiEditText 之后的密码键盘区别](/img/Emoji-Et-demo.png)

*只有左图是正常的，中图和右图都使用了 EmojiEditText，可以看到无论是否添加 textNoSuggestion，结果都是错误的*

经排查后发现，只有 EmojiEditText 会存在该问题，如果直接使用原生的 EditText 则正常。

当我们设置 textPassword 之后，EditText.getInputType() 的值是 129。然而 EmojiTextView 会在 init 过程中执行一句 `setKeyListener(super.getKeyListener)` （如下图），这行代码会导致 EditText 的 inputType 变为 1。

![EmojiEditText init 方法中导致 inputType 出错的语句](/img/Emoji-Et-init.png)

EmojiEditText 虽然重写了 `setKeyListener()` 方法，但是其内部实现与父类相比并没有什么特别的。我们可以创建一个正常的 TextView，设置其 inputType="textPassword" ，然后执行下列语句：

```JAVA
Log.d("6v", "InputType=" + tv.getInputType()); // inputType=129
tv.setKeyListener(tv.getKeyListener());
Log.d("6v", "InputType=" + tv.getInputType()); // inputType=1
```

可以看到即使什么也不做，只是简单地执行一下 `setKeyListener()` ，也会导致 inputType 被重置。因为 TextView 会在 `setKeyListener()` 方法中会将 TextView 自身的 inputType 设置为 keyListener 的 inputType。相关代码如下：

![Emoji setKeyListener 相关代码](/img/Emoji-Et-Keylistener.png)

而继续往下追踪，我们不难发现，在 TextView 初始化过程中，会根据 inputType 去生成不同类型的 KeyListener：TextKeyListener, DigitsKeyListener, DateKeyListener 等等，通过这些不同类型的 KeyListener 去限制用户输入的内容。在 TextKeyListener 中还会根据 **是否存在大写限制** 与 **是否自动纠正** 产生不同的细分，然而 **是否为密码** 并不在考虑范围内。因此无论是 textPassword 还是默认的 inputType，都会产生同一个类型的 textKeyListener （inputType=1） 。在正常使用中，这里的处理没什么问题。但如果我们重新执行了 `setKeyListener()`，就会导致 TextView 的 inputType 被覆盖为 1，使得谷歌原生键盘产生了单词联想。

因此，如果要解决这个 Bug，只能引入 EmojiEditText 的源码，修改其 init 逻辑，在执行 `setKeyListener()` 之前先保存一下 `TextView.getInputType()` 的值，然后 `setKeyListener()` 之后再重新 `setInputType()`。

然而这么做实在是太复杂，所以最后的解决方案只是简单地将所有存在 `inputType="textPassword"` 的地方改成了 EditText （而不是原来的自定义基类 EditText）。