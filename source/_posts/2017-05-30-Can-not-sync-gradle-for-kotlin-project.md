---
title: 升级到 AndroidStudio 3.0 Canary 之后新建 Kotlin 项目无法编译的问题
date: 2017-05-30 16:49:10
tags: [Android, Kotlin]
---

> 需要确认 kotlin_version 是否最新

<!-- more -->

---

AndroidStudio 最近在 Canary 频道更新了 3.0 版本，其中加入了 Kotlin 的支持，于是就不需要安装 Kotlin 插件了。但是我使用 AndroidStudio 3.0 创建了新的项目并勾选了 Kotlin 的支持之后，gradle 会在 sync 时失败。错误信息如下：

> Error:Unable to find method 'com.android.build.gradle.internal.variant.BaseVariantData.getOutputs()Ljava/util/List;'.
> Possible causes for this unexpected error include:<ul><li>Gradle's dependency cache may be corrupt (this sometimes occurs after a network connection timeout.)
> <a href="syncProject">Re-download dependencies and sync project (requires network)</a></li><li>The state of a Gradle build process (daemon) may be corrupt. Stopping all Gradle daemons may solve this problem.
> <a href="stopGradleDaemons">Stop Gradle build processes (requires restart)</a></li><li>Your project may be using a third-party plugin which is not compatible with the other plugins in the project or the version of Gradle requested by the project.</li></ul>In the case of corrupt Gradle processes, you can also try closing the IDE and then killing all Java processes.

经过很多尝试之后，最终将 root project 的 build.gradle 文件中的 `ext.kotlin_version` 值从 1.1.2-3 改为了 1.1.2-4 之后解决。

这应该会是篇时效性强的文章。之后再碰到同样的问题的话需要自行确认 Kotlin 插件的最新版本并尝试替换。

其他尝试过的方案有：

- 升级 jdk 至最新版本（1.8.0_131）并更新环境变量
- 将 app 的 build.gradle 中的 
	`compile "org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version"`   
	替换为   
	`compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"`   
	（移除 `-jre7`）  
- 确认 AndroidStudio Setting 中的 Gradle 使用的是 default gradle wrapper 而不是自定义的