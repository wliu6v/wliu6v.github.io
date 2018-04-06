title: 如何为 DFP Ads 自动添加 Test Device Id
date: 2016-06-25 10:45:04
tags: [Android]
---

在使用 DFP Ads 的时候，官方文档中会要求开发人员在测试广告的过程中必须使用测试广告 ID，并需要将测试的设备在代码中添加为测试设备才可以正常使用。但添加的过程有些麻烦：我们需要将测试设备连接至电脑，启动应用拉取广告，同时观察 logcat 中的日志，抓取到形如 `... addTestDevice("123456789ABCDEF0123456789ABCDEF0") ...` 的 log 后，将其中的 Device Id 写到代码中并重新编译才可以。如果测试设备很多，就要对每个设备都抓一遍 log，比较繁琐。因此希望能够在代码中自动抓取 DFP Ads 输出的 log，提取出 Test Device Id，并添加到代码中。这样就可以做到每个新的设备都可以在拉取一次广告后将自身加为测试设备了。

实现基本如下：

<!-- more -->

```
private void loadAd() {
	PublisherAdView adView = (PublisherAdView) findViewById(R.id.ad_view);
	PublisherAdRequest.Builder adBuilder = new PublisherAdRequest.Builder();
	adView.loadAd(adBuilder.build());
	saveTestDevice();
}

private void addTestDevice(PublisherAdRequest.Builder builder) {
	String testDeviceId = PreferenceManager.getDefaultSharedPreferences(context).getString(
				"AD_TEST_DEVICE_ID", "");
	if (!TextUtils.isEmpty(testDeviceId)) {
		builder.addTestDevice(testDeviceId);
	}
}

private void saveTestDevice() {
	try {

		ArrayList<String> loadLogCommand = new ArrayList();
		loadLogCommand.add("logcat");       // 调用 logcat 命令
		loadLogCommand.add("-d");           // 将命令的输出内容直接输出到屏幕上
		loadLogCommand.add("|");            // | grep "Ads" 用来过滤输出的结果，仅需要观察 Ads 相关的 log
		loadLogCommand.add("grep");
		loadLogCommand.add("\"Ads\"");
		
		Process process = Runtime.getRuntime().exec(
				loadLogCommand.toArray(new String[logLogCommand.size()]));
		BufferedReader bufferedReader = new BufferedReader(
				new InputStreamReader( process.getInputStream()), 1024 );
		String line;
		while ((line = bufferedReader.readLine()) != null) {
			if (line.contains("addTestDevice")) {
				int index = line.indexOf("(\"") + 2;
				String testDeviceId = line.substring(index, index + 32);
				if (!TextUtils.isEmpty(testDeviceId)) {
					PreferenceManager.getDefaultSharedPreferences(context).edit().putString(
						"AD_TEST_DEVICE_ID", testDeviceId).apply();
				}
				break;
			}
		}


		Arraylist<String> clearLogCommand = new ArrayList();
		clearLogCommand.add("logcat");      // 清空 logcat 输出内容
		clearLogCommand.add("-c");
		Runtime.getRuntime().exec(clearLogCommand.toArray(new String[clearLogCommand.size()]));

	} catch (Exception e) {
		e.printStackTrace();
	}
}

```

### 备注

- 一些网上的教程说需要添加 READ_LOGS 的 permission 以获取读取日志的权限。但实际上，如果只是获取当前应用所生成的 log，似乎并不需要此权限。（况且读其他应用的日志，在 4.1 以上的 Android 系统中也需要 root 才能实现）
- clearLog 那个命令，据说如果不清理日志的话，任何操作都将产生新的日志并导致代码进入死循环。。不过实际上也没有出现，我猜测是如果要在 while 语句中输出 log 的话需要调用此命令，否则无所谓。

### 参考资料

[博客园 - 马涛 - 如何获取 android 的系统日志 logcat](http://www.cnblogs.com/mataojin/archive/2011/11/07/2239260.html)