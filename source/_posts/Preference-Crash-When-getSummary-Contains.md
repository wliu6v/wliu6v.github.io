title: Preference Crash When getSummary() Contains '%'
date: 2015-04-06 16:11:48
tags: [Android]

---

当 ListPreference 中执行 getSummary 的时候碰到了字符 '%' 时会导致应用 crash。

### 解决方式

重写 ListPreference 的 getSummary 方法，避免在遇到 '%' 的时候 crash。

<!-- more -->

``` JAVA  
private class NoneFormatSummaryListPreference extends ListPreference {

	public NoneFormatSummaryListPreference(Context context) {
		super(context);
	}

	@Override
	public CharSequence getSummary() {
		try {
			// for call super.super.getSummary()
			CharSequence[] entries = super.getEntries();
			super.setEntries(null);
			CharSequence result = super.getSummary();
			super.setEntries(entries);
			return result;
			// return super.getSummary();
		} catch (Exception e) {
			e.printStackTrace();
			return "";
		}
	}
}
```

###	原理

以下内容皆引用自 [Preference Summary or Secondary Text](http://gmariotti.blogspot.sg/2013/02/preference-summary-or-secondary-text.html) :

> __A note about ListPreference.__ In [android doc](http://developer.android.com/reference/android/preference/ListPreference.html#setSummary%28java.lang.CharSequence%29) we can read:

> > If the summary has a String formatting marker in it (i.e. “%s” or “%1$s”), then the current entry value will be substituted in its place when it’s retrieved.

> As a result, if you set “summary” that contains “%” char (like "5%"), you can have `java.util.UnknownFormatConversionException`: Conversion: because it may be an unknown format.
The reason is here (standard method in ListPreference)

> ```
@Override
public CharSequence getSummary() { 
	final CharSequence entry = getEntry(); 
	if (mSummary == null || entry == null) { 
		return super.getSummary(); 
	} else { 
		return String.format(mSummary, entry); 
	} 
}
```

> If you want to use value like "5%" in ListPreference, in API Level 11 or higher, you need to be careful.
> I suggest you create a subclass of ListPreference, override getSummary() method and return whatever you need, skipping the call to String.format().
> Alternatively you can obtain percent character in String.format() by specifying "%%". 