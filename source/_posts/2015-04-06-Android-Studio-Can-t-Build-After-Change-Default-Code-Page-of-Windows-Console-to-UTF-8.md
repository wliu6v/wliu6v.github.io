title: Android Studio Can't Build After Change Default Code Page of Windows Console to UTF-8
date: 2015-04-06 16:16:50
tags: [Android, Windows]

---

在 Windows 环境下进行 Android 开发的时候，如果修改了 CMD 的默认代码页，将会导致 Android Studio 无法正常编译。因为修改了默认代码页后会导致 `C:\Windows\System32\find.exe` 失效。

将 CMD 的默认代码页修改回默认值即可。在简体中文环境下该默认值通常是 `chcp 936`。

如果就是希望每次使用 CMD 的时候都是 UTF-8 代码页，可以建一个 bat 文件，如 `cmd_65001`，并将其添加至环境变量中。bat 文件的内容为 ：

	C:\Windows\System32\cmd.exe /k CHCP 65001

这样每次只要在运行里面输入 cmd_65001 ，就可以开启一个 UTF-8 代码页的 CMD 窗口了。

不过好像不能改字体- =


[Ref : superuser - Change default code page of Windows console to UTF-8](http://superuser.com/questions/269818/change-default-code-page-of-windows-console-to-utf-8)
[Ref : superuser - Why find.exe not work in Windows 7?](http://superuser.com/questions/176737/why-find-exe-not-work-in-windows-7)