---
title: Config Multi GitHub Account on Windows
date: 2017-10-26 22:38:37
tags: [Git]
---

> 用 .git-credentials 文件的方式只能对每个站点配置一个账号。如果希望为同一个站点（比如 Github）配置两个账号的话需要用 ssh 实现。

<!-- more -->

当我们用 git 从 github 的私有库拉取代码的时候，需要提供对应的权限。最基础的方式是每次 git fetch 的时候都输入用户名与密码。而我们也可以通过配置 .git-credentials 文件的方式避免每次都重复输入。

但是 .git-credentials 文件的一个缺陷是只能为每个站点配置一个账号。如果是多个站点，比如 github.com 与 bitbucket.org 的话很容易配置，但是如果在同一个站点比如 GitHub 有多个账号的话，那么就只能使用 ssh 的方式进行配置。

# 配置步骤

> 操作系统 Windows，命令行环境为 git 自带的 git bash。 不能用 cmd，会缺少一些必要命令。

## 1. 为各个账号创建 ssh 文件
- `ssh-keygen -t rsa -C "your_email_1@gmail.com"`
    - 此处需输入存放的路径，一般放在 用户文件夹下的 .ssh 目录中。因为要配置多个账号，建议在文件名上进行区别。比如: `~/.ssh/id_rsa_your_email_1`
    - 之后输入密码，不需要的话就直接两下回车。
- `ssh-keygen -t rsa -C "your_email_2@163.com"`

每条命令可以得到一个私钥跟一个公钥文件。公钥文件多了后缀名 pub。

![ssh-keygen -t rsa -C "your_email_1@gmail.com"](/img/Config-Multi-GitHub-Account-on-Windows/Config-Multi-GitHub-Account-1.png)

## 2. 令 git bash 启动的时候自动加载 ssh 文件

找到 .bashrc 文件。在 linux 底下是 ~/.bashrc。 在 windows 下应该是在 git 安装目录的 etc/bash.bashrc 文件。添加以下内容：

``` bash
env=~/.ssh/agent.env
agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }
 
agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }
 
agent_load_env
 
# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2= agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)
 
if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add ~/.ssh/id_rsa_your_email_1   # 这里修改成 ssh-add 接上上面配置的路径
    ssh-add ~/.ssh/id_rsa_your_email_2   # 这里修改成 ssh-add 接上上面配置的路径
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add ~/.ssh/id_rsa_your_email_1   # 这里修改成 ssh-add 接上上面配置的路径
    ssh-add ~/.ssh/id_rsa_your_email_2   # 这里修改成 ssh-add 接上上面配置的路径
fi
 
unset env
```

![ssh-keygen -t rsa -C "your_email_1@gmail.com"](/img/Config-Multi-GitHub-Account-on-Windows/Config-Multi-GitHub-Account-2.png)

修改完成后，关闭 git bash 再重新打开，应该会看到 Identity added 的信息

![ssh-keygen -t rsa -C "your_email_1@gmail.com"](/img/Config-Multi-GitHub-Account-on-Windows/Config-Multi-GitHub-Account-5.png)

## 3. 在 Github 中配置对应的密钥

在两个账号的 GitHub 下分别进行配置。add key 的时候将公钥文件（.pub 结尾的文件）内容全都 copy 上去。通常是以 ssh-rsa 开头，账号结尾。title 随便取，如果有多台电脑的话，可以用电脑名进行区分。

![ssh-keygen -t rsa -C "your_email_1@gmail.com"](/img/Config-Multi-GitHub-Account-on-Windows/Config-Multi-GitHub-Account-3.png)

![ssh-keygen -t rsa -C "your_email_1@gmail.com"](/img/Config-Multi-GitHub-Account-on-Windows/Config-Multi-GitHub-Account-4.png)

## 4. 用 ssh 连接替代 http 连接。

之后在进行 git clone 的时候就可以选择 ssh 地址。如果是本地已经存在的库，可以通过 `git remote set-url` 命令进行修改。 `git remote set-url origin git@github.com:foo/bar.git`

要如何查看当前 git 目录所关联的远程主机，用 `git remote -v`

## 5. 对不同的库使用不同的账号身份

每次拉取代码之后，修改 git config --local 的 user.email 和 user.name，避免提交的时候，用 A 账号的信息往 B 的库里面提交了内容。

# ref

- [working-with-ssh-key-passphrases](https://help.github.com/articles/working-with-ssh-key-passphrases/#auto-launching-ssh-agent-on-git-for-windows)
- [Multiple SSH Keys settings for different github account](https://gist.github.com/jexchan/2351996)
- [could-not-open-a-connection-to-your-authentication-agent](https://stackoverflow.com/questions/17846529/could-not-open-a-connection-to-your-authentication-agent)