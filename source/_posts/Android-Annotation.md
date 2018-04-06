title: Android Annotation
date: 2015-12-20 17:57:44
tags: [Android]
---

##	IntDef

通过 `@IntDef` 标注可以限定一个方法的输入输出内容。

枚举类型可以很好的限定值的范围，但是有性能问题（Enum 的性能问题在 [Android Developer](https://developer.android.com/topic/performance/memory.html#Abstractions) 中有提到，在 [胡凯 - Android性能优化典范 - 第3季](http://hukai.me/android-performance-patterns-season-3/) 中的第4小节中有详细解释）。在很多情况下，可以通过自定义的 `@IntDef` 和 `@StringDef` 来替代枚举类型。而且用起来也比在枚举类型里面实现 valueOf 等方法要简洁少许。

<!-- more -->

``` Java
public class IceCreamFlavourManager {
    private int flavour;

    public static final int VANILLA = 0;
    public static final int CHOCOLATE = 1;
    public static final int STRAWBERRY = 2;

    @IntDef({VANILLA, CHOCOLATE, STRAWBERRY})
    public @interface Flavour {
    }

    @Flavour
    public int getFlavour() {
        return flavour;
    }

    public void setFlavour(@Flavour int flavour) {
        this.flavour = flavour;
    }
}
```