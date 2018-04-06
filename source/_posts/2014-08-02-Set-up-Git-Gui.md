title: Git & GitHub
date: 2014-08-02 18:43:20
tags: [Git]

---

这篇文章会集合平常使用 Git & GitHub 的各种坑与小技巧。持续更新。

<!-- more -->

# Git Clean

当切换分支到 gitignore 改动的版本时，通常会导致 git 记录一些 untracked files，导致无法正常返回到原分支状态。此时需要将 untracked files 先移除掉。

可通过 git clean 命令清除 untracked files。

先使用 -n 列出将会被清除的文件列表，进行确认。确认确实需要被删除后，使用 -f 参数执行删除操作。
如果需要清除目录，则应该在相应参数后面加 d，如 -nd 列出文件列表，-fd 执行删除操作。

如果只删除 gitignore 包含的文件，使用 -x

# Rewrite History

参考自 ： [Git Community Book 中文版](http://gitbook.liuhui998.com/4_3.html)

通过在 `git rebase` 后面加 `-i` 或 `--interactive` 调用交互模式。如

`git rebase -i <commit_num>`  

此命令会将需要被 rebase 的提交列出来，并可以在这个时候对每个提交进行修改。PS.此界面默认为 Vim，编辑的时候需要参考 [Vim 常用命令总结](http://pizn.github.io/2012/03/03/vim-commonly-used-command.html)。常用的应该就是 dd 命令和 p 命令。

在编辑结束后（:wq 保存并退出），git 会开始执行 rebase 操作，在遇到 splash、edit 等状态时，会暂停 rebase 过程。

#	How to skip “Loose Object” popup when running 'git gui'

	git config --global gui.gcwarning false

[see stackoverflow](http://stackoverflow.com/questions/1106529/how-to-skip-loose-object-popup-when-running-git-gui)

#	禁用快速合并

[ref:Git分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html)
[ref:Understanding the Git Workflow](https://sandofsky.com/blog/git-workflow.html)

Git 在合并的时候默认的是使用快速合并。这样的话，如果当两个分支完全没有冲突的时候，会自动合并到一起，就不会有一个明确的提交标明这是一次合并操作了。如下图：

![](http://image.beekka.com/blog/201207/bg2012070505.png)

虽然会是提交记录看上去简洁一些，但有时候要 reset 合并操作的时候可能会带来不便。而使用 --no-ff 参数后，就会执行正常合并，效果如下图：

![](http://image.beekka.com/blog/201207/bg2012070506.png)

可以通过如下命令使得 git 全局禁止快速合并 ： 

	git config --global merge.ff no

#	How to remember username and password

在存放 .gitconfig 的文件夹中（通常为 C:/User/xxxx )，新建一个文件名为 `.git-credentials` （不能直接新建或者重命名，需用 cmd）。然后将其内容修改为： 

	https://GITHUB_USERNAME:GITHUB_PASSWORD@github.com

并在 `.gitconfig` 文件中添加

	[credential]
		helper = store

##  如何同时对多个账户进行配置

see [Config Multi GitHub Account on Windows](../Config-Multi-GitHub-Account-on-Windows)


#	在 Git Gui 中添加部分命令

此内容仅适用于 Git Gui 工具。

修改 `.gitconfig` 文件，添加 `[guitool "命令名"]` 字段，并将其值设置为 `cmd = git xxxxxx` 

通常可以如此进行设置：
	
	[guitool "Rebase onto..."]
	    cmd = git rebase $REVISION
	    revprompt = yes
	[guitool "Rebase/Continue"]
	    cmd = git rebase --continue
	[guitool "Rebase/Skip"]
	    cmd = git rebase --skip
	[guitool "Rebase/Abort"]
	    cmd = git rebase --abort
	[guitool "合并 origin\\master"]
		cmd = git merge origin/master

##	其他设置

账户

	[user]
		email = xxx@xxx.com
		name = GITHUB_USERNAME

编码格式

	encoding = utf-8

# 参考资料

- [Git Book](http://git-scm.com/book/en/v2)  
- [Git Book 中文版](http://git-scm.com/book/zh/v2)
- [Git Hub 秘籍](https://snowdream86.gitbooks.io/github-cheat-sheet/content/zh/index.html)