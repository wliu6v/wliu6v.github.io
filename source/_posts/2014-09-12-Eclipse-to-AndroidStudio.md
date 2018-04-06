title: Eclipse to AndroidStudio
date: 2014-09-12 00:11:56
tags: [Android]

---

### 1. 从 Eclipse 中 Export 项目

在需要点击的项目上点击右键，选择 Export，选择 Android - Generater Gradle build files，然后 Next。

如果无此选项，说明 ADT 版本过低，需要去 [Google Developer](http://developer.android.com/sdk/index.html) 下载较新的版本。

<!-- more -->

![](/img/Eclipse2AndroidStudio_1.png)

###	2. 勾选需要生成 Gradle 的项目

这里选择有哪些项目需要生成 Gradle。此处应将项目所依赖的其他项目一并勾选。之后一路 Next。成功之后会提示生成的 build.gradle 的位置。

![](/img/Eclipse2AndroidStudio_2.png)

### 3. 用 Android Studio 打开 build.gradle

此处需要注意的是，用 Android Studio 打开的不能使整个工程，而是在上一步中所生成的 build.gradle 的位置。

如下图，此时我们选择的不应该是 demo 或者 library，而是 build.gradle 文件。在 demo 目录与 library 目录下可能也会有 build.gradle 文件，但我们只需要包含外层的这个即可。

在执行此操作时可能会提示选择 Gradle，按照默认选择即可 (Use default gradle wrapper)，之后可能要下载 Gradle，通常需要下载很久（5~30分钟）

![](/img/Eclipse2AndroidStudio_3.png)