title: Integer.getInteger()
date: 2015-04-06 16:09:55
tags: [Android]

---

Integer 类中包含三个名字比较相近的静态方法 : `getInteger()`, `valueOf()`, `parseInt()` 。这三个方法名字差不多，但实际上，getInteger 方法是个坑。

valueOf() 与 parseInt() 这两个方法效果比较接近。都是将某个 String 转为数值类型。其中，valueOf() 方法将会把 int 或 String 转成 Integer 类型，parseInt() 方法将会把 String 转成 int 类型。

但对于 getInteger() 并非如此。JAVADoc 对 getInteger() 方法的描述是：

>	Returns the Integer value of the system property identified by string.
>	Returns null if string is null or empty, if the property can not be found or if its value can not be parsed as an integer.

>	Parameters
>	
>	string
>		the name of the requested system property.
>		
>	Returns
>		the requested property's value as an Integer or null.

因此如果对一个任意数值字符串使用 getInteger() 方法，结果通常为 `null`。

类似的， `Boolean.getBoolean("true");` 将得到结果 `Boolean.FALSE`。

[Ref : Integer.getInteger. Are you kidding me?](http://konigsberg.blogspot.com/2008/04/integergetinteger-are-you-kidding-me.html)