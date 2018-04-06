title: 一个神奇的 Bug：在 Win10 中使用 MacType 导致 Git rebase 命令出现问题
date: 2016-05-22 16:22:10
tags: [Windows, Git]
---

之前换了浏览器，然后突然想试试看 MacType 。查了一下发现 MacType 并不支持 Win10，但它没有说为什么不支持，我就强行装了一个用了用。刚开始用的时候感觉还行，虽然碰到一些权限问题，但基本上能用。结果用着用着突然发现我的 Git 无法正常的执行 rebase 操作了。比如，在 b 分支上执行 git rebase a，结果实际效果却似乎是在 b 上执行了 git rebase origin/b。而且 rebase 过程中碰到了冲突之后会卡主，即使冲突解决了也无法 git rebase --continue，甚至连 git rebase --abort 也不行。命令行会一直卡在 rebase 的过程状态。

刚碰到这个问题的时候我查了很久也没找到解决方案，而且除了 git 命令行之外，我还尝试了在 git gui 中与 source tree 中使用 rebase 功能，效果都一样。最后想破了头想到，这个问题好像是启用了 MacType 之后才引起的。卸载 MacType 并重启之后，git rebase 就正常了。

<!-- more -->

感觉这问题简直是坑啊。。。 太莫名其妙了。也许跟我 MacType 的管理员权限设置也有一定关系，不知道其他人在 Win10 下用 MacType 的时候有没有遇到同类问题，有的话一定告诉我。

顺便写一下卡在 rebase 的过程是如何解决的：首先确认当前的代码状态是正常的（防止丢代码啊），然后移除 `.git/rebase-merge` 或者 `.git/rebase-apply` 即可。处于冲突状态的文件可以直接 reset 掉。ref ： http://stackoverflow.com/questions/3685001/git-how-to-fix-corrupted-interactive-rebase