title: Multi-Flavor in Android Studio
date: 2016-05-22 15:34:41
tags: [Android]
---

## 如何为多个 Flavor 配置不同权限

Multi-Flavor 的一个重要功能是针对于不同的 Flavor 加载不同的代码。通过此功能，我们可以实现为多个 Flavor 配置不同应用权限的效果。

具体方式可参考这篇文章：[Android: How to Implement ProductFlavor-Dependent Permissions with Gradle](https://futurestud.io/blog/how-to-implement-product-flavor-dependent-permissions) .

通过这种方式，我们可以将所有 Flavor 共有的部分放在默认的 AndroidManifest 文件中，然后针对不同的 Flavor 再实现各自额外的内容。在编译对应 Flavor 的时候，其各自的 AndroidManifest 文件会跟默认的 AndroidManifest 文件合并，相同节点的内容会合并到一起。非常方便。

<!-- more -->

但是上述教程的代码结构是针对于 Android Studio 的代码结构的。如果是 Eclipse 结构的代码，我们需要将目录结构调整为如下所示：

	module 
		- AndroidManifest.xml
		- src
		- res
		- flavor1
			└ AndroidManifest.xml 
		- flavor2
			└ AndroidManifest.xml

然后还要在 build.gradle 中添加 sourceSets 字段：

	sourceSets {
	    main {
	        manifest.srcFile 'AndroidManifest.xml'
	        java.srcDirs = ['src']
	        resources.srcDirs = ['src']
	        aidl.srcDirs = ['src']
	        renderscript.srcDirs = ['src']
	        res.srcDirs = ['res']
	        assets.srcDirs = ['assets']
	    }
	
	    flavor1Flavor {
	        manifest.srcFile 'flavor1/AndroidManifest.xml'
	    }
	
	    flavor2Flavor {
	        manifest.srcFile 'flavor2/AndroidManifest.xml'
	        java.srcDirs = ['flavor2/src']
	    }
	
	    // Move the tests to tests/java, tests/res, etc...
	    instrumentTest.setRoot('tests')
	
	    debug.setRoot('build-types/debug')
	    release.setRoot('build-types/release')
	}

## 如何为多个 Flavor 配置不同的 url-scheme

原理跟上面一样，只要在各自的 AndroidManifest 写上不同的 url-scheme 即可。基本思路如下：

### 1. 默认 AndroidManifest.xml

	<manifest>
		<uses-permission />
		<application>
			<activity>
				<intent-filter> intent 1 </intent-filter>
			</activity>
		</application>
	</manifest>

### 2. Flavor1/AndroidManifest.xml

	<manifest>
		<uses-permission />
		<application>
			<activity>
				<intent-filter> intent 2 </intent-filter>
			</activity>
		</application>
	</manifest>

### 3. Flavor2/AndroidManifest.xml

	<manifest>
		<uses-permission />
		<application>
			<activity>
				<intent-filter> intent 3 </intent-filter>
				<intent-filter> intent 4 </intent-filter>
				<intent-filter> intent 5 </intent-filter>
				<intent-filter> intent 6 </intent-filter>
			</activity>
		</application>
	</manifest>

## 如何切换当前编译的 Flavor

在 Android Studio 的 Menu 中找到 View -> Tool Windows -> Build Variants， 然后将对应 Module 的 BuildVariant 进行修改即可。对于每个 Flavor，都会生成 Debug 和 Release 两种编译方式。选择时需注意。