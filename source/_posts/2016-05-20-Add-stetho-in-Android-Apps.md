title: Add stetho in Android Apps
date: 2016-05-20 18:01:59
tags: [Android]
---


[Stetho](http://facebook.github.io/stetho/) 是 Facebook 提供的一个面向 Android App 的调试工具。在配置好 Stetho 之后，我们可以在 Chrome 中对特定应用进行调试，功能包括：查看数据库与 SharedPerferences、查看网络请求、查看当前视图布局（hierarchy view）等等，甚至可以通过 JavaScript 调用 Android App 中的部分命令。

<!-- more -->

## 如何集成

可以参照 [Stetho 官方网页](http://facebook.github.io/stetho/)。
步骤基本上就两三步：
1. 在 build.gradle 添加 stetho 的依赖。
2. 在 Application 初始化的时候添加 stetho 的初始化代码。

这两步完成之后可以实现除抓包之外的大部分功能。
要实现抓包，需要根据使用的网络请求库进行特定的配置。比如用的如果是 OkHttp 的话，就需要添加 stetho-okhttp 的依赖，并且在初始化 OkHttp 的时候添加对 Stetho 的 Interceptor。

具体的步骤可以参照其他人的教程：

http://stormzhang.com/android/2015/03/05/android-debug-use-chrome/
http://blog.csdn.net/sbsujjbcy/article/details/45420475

## 如何仅在 Debug 下启用 Stetho

之前已经有很多人写过关于基本的如何集成 Stetho 及其使用方式。这里我着重描述如何仅在 Debug 版本中使用 Stetho。

Stetho 是一个调试工具，其大部分使用场景都仅限于开发阶段。如果在 Release 中也导入 Stetho 的话，会导致应用体积变大，并有可能使应用的安全性降低。因此我们需要仅在 Debug 版本中使用 Stetho。

基本方法分为两步：

1. 在 build.gradle 添加依赖的时候，使用 debugCompile 而不是 compile
2. 分别在 debug 和 release 代码目录中创建 Stetho 初始化类，然后在 Application 初始化的时候使用该 Stetho 初始化类进行初始化。

如果项目采用的是 Android Studio 的代码结构，并且 OkHttp 直接在项目中，那么只要参照这篇教程即可： http://stackoverflow.com/questions/30172308/include-stetho-only-in-the-debug-build-variant/31483962#31483962

但我在我的项目中按照此教程进行操作的时候碰到三个问题，下面分别讲述

## 问题一：debug 目录可以识别，但 release 目录无法正常识别出 java 代码

当我们创建 debug 目录和 release 目录时，有时候会出现其中一个目录中的 java 文件无法被正常识别为 java class 的情况。此时需要注意，该目录与 build variant 的编译方式是同步的。当我们以 debug 方式编译的时候，Android studio 可以识别 debug 目录，但是无法识别 release 目录。反之亦然。

## 问题二：OkHttp 是以 Library 方式添加，无法以 debug 方式进行编译

对于 Library 的 Module，默认总是会以 Release 进行编译，即使在编译的时候选择的 Debug 也没用。为了防止这一情况，我们需要在 Library 的 build.gradle 中添加以下内容：

	publishNonDefault true

然后在主 Module 中将对改 library 的依赖改为：

	debugCompile project(path: ':foo_library', configuration: 'debug')
    releaseCompile project(path: ':foo_library', configuration: 'release')

即可正常的以 debug 方式编译 library 了。其他的要点同上

## 问题三：项目结构是 Eclipse 结构而非 Android Studio 结构

在 Android Studio 结构的代码中，debug 目录是 `src/debug/java/`，release 目录是 `src/release/java/` 

而在 Eclipse 的代码结构中，debug 目录是 `build-types/debug/java/`, release 目录是 `build-types/release/java/`

基本结构如下图所示：

	module
		- build-types
			- debug
				- java
					- com
						- package.name
			- release
				- ...
		- src
			- com
				- package.name

----

## 其他的问题

我会将使用 Stetho 过程中碰到的其他问题陆续更新在这里。

###	Database 中内容显示不全

使用 Stetho 查看 Database 时，显示的行数被限制在 250 行，代码在 [stetho/stetho/src/main/java/com/facebook/stetho/inspector/protocol/module/Database.java](https://github.com/facebook/stetho/blob/master/stetho/src/main/java/com/facebook/stetho/inspector/protocol/module/Database.java#L54) 中：
	
	  /**
	   * The protocol doesn't offer an efficient means of pagination or anything like that so
	   * we'll just cap the result list to some arbitrarily large number that I think folks will
	   * actually need in practice.
	   * <p>
	   * Note that when this limit is exceeded, a dummy row will be introduced that indicates
	   * truncation occurred.
	   */
	  private static final int MAX_EXECUTE_RESULTS = 250;

因此我们要查看 database 的时候可能需要自己编写 SQL 语句进行查询。方法是选中某个 database（而非某个 Table）并在右侧光标处输入语句进行查询。