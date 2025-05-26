---
layout: post
title: 服务器上创建 git 远程仓库
date: 2024-09-23 11:12:56+0800
description:
tags:
  - git
categories: Potpourri
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

本篇文章将会介绍如何在服务器上创建一个 `git` 中央服务器并且配置 `ssh` 连接。

# 必要工具
* `git`
* `ssh`

# 创建 `git` 用户
我们一般不会使用 `root` 用户来管理 `git` 仓库，所以我们可以创建一个用户专门用来管理 `git` 仓库，通常
这个用户的名字叫 `git`。可以通过以下命令创建一个 `git` 用户：

```bash
# -m 用于创建用户的家目录：/home/git
useradd -m git
```

创建成功后， 我们需要为 `git` 用户设置密码：

```bash
# 第一次可能需要输入 `sudo` 的密码，然后输入两次新密码
sudo passwd git
```

# 创建仓库
仓库的创建非常简单，我们首先通过以下命令切换到 `git` 用户：

```bash
# 需要输入 `git` 用户的密码，或者可以使用 `sudo su git` 切换，此时需要输入 `sudo` 的密码
su git
```

然后我们可以通过以下命令创建一个仓库 `a`：

```bash
# 将仓库放置在 /home/git/a.git 目录下
git init --bare /home/git/a.git
```

# 设置远程 `ssh`
我们需要为 `git` 用户设置 `ssh`，这样我们才能通过 `ssh` 连接到 `git` 服务器。

我们只需要通过以下命令创建相应的目录即可：

```bash
# 切换到 git 用户，以保证目录的权限正确
su git
# ~/.ssh 目录的权限必须是 700，~/.ssh/authorized_keys 的权限必须是 600
cd ~
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

# 上传公钥
我们需要将我们的公钥上传到 `git` 服务器，这样我们才能通过 `ssh` 连接到 `git` 服务器。

公钥的创建这里不多介绍。我们只需要将本地的公钥内容全部追加到 `git` 用户的 `~/.ssh/authorized_keys`
文件中即可。

# 克隆仓库
如果一切正常，假定服务器域名为 `1.1.1.1`，端口为 `123` 的情况下，用户为 `git`，仓库路径为 `~/a.git`，
我们可以通过以下命令克隆仓库：

```bash
# 会在当前目录下创建一个 a 目录
git clone ssh://git@1.1.1.1:123/~/a.git
```

通过克隆命令获取的仓库，我们可以通过 `git remote -v` 查看远程仓库的地址。

如果想要手动添加远程仓库，可以通过以下命令：

```bash
# 添加一个名为 origin 的远程仓库
git remote add origin ssh://git@1.1.1.1:123/~/a.git
```

# 配置 `git-shell`
如果我们不希望 `git` 用户能够登录到服务器，我们可以将 `git` 用户的 `shell` 设置为 `git-shell`，这样
`git` 用户只能通过 `git` 命令来操作仓库。

首先查看 `/etc/shells` 中是否有 `git-shell`，如果没有，我们需要将 `git-shell` 添加到 `/etc/shells` 中：

```bash
# 查看 /etc/shells 中是否有 git-shell
cat /etc/shells | grep git-shell
# 如果没有，我们需要将 git-shell 添加到 /etc/shells 中
echo $(which git-shell) | sudo tee -a /etc/shells
```

接下来，我们可以通过以下命令将 `git` 用户的 `shell` 设置为 `git-shell`：

```bash
sudo chsh -s $(which git-shell) git
```

设置完成后，`git` 用户将不能登录到服务器，只能通过 `git` 命令来操作仓库。

# 禁止转发
即使 `git` 用户目前不能登录，但是其依然可以执行一些端口转发的操作，我们可以通过在
`~/.ssh/authorized_keys` 对想要禁止的公钥添加
`no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty` 来禁止转发。

例如，对第一行的公钥进行禁止转发：

```bash
# 在第一行的前面添加后，内容可能如下：
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAAA...
```

# 配置快速连接
通常租用的服务器只有一个 `ip` 地址，没有相应的域名解析，不过我们可以通过 `~/.ssh/config` 文件来配置
快速连接。

以之前的例子为例，我们需要在本地的 `~/.ssh/config` 中添加以下内容：

```bash
Host gitserver
  HostName 1.1.1.1
  User git
  Port 123
```

这样操作后，我们的 `clone` 命令可以简化为：

```bash
# 这个命令会获取 /home/git/a.git 仓库，因为 ssh 连接时默认路径为 ~
git clone gitserver:a.git
```

# 巨人的肩膀
* [Git on the Server - Setting Up the Server](https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server)
