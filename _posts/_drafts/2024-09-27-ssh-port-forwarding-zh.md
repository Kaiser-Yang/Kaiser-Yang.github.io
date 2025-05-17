---
layout: post
title: ssh 端口转发简介
date: 2024-09-27 19:42:56+0800
last_updated: 2025-04-15 16:19:12+0800
description:
tags:
  - Unix/Linux
categories: Potpourri
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

`ssh` 一共支持三种类型的端口转发：本地转发、远程转发和动态转发。

**注意**：由于本地和远程是一个相对的概念，
本文中提到的本地如果没有特殊说明可以理解成没有公网 `ip` 的主机，
而提到的远程如果没有特殊说明可以理解成有公网 `ip` 的主机。

# 本地转发

本地转发指的是将本地端口收到的请求通过某个跳板机转发到目的主机的某个端口上。

先介绍一个最常见的使用例子，如果你正在使用 `VSCode` 通过 `ssh-remote` 连接到某个远程主机上进行开发，
并且在 `VSCode` 中启动了一个 `web` 服务，那么 `VSCode` 会自动为你开启一个端口转发，
这样你可以在浏览器中通过 `localhost` 来访问你远程服务器的 `web` 服务。
我没有去研究过 `VSCode` 的端口转发是否是通过 `ssh` 命令实现的，
但是我知道 `ssh` 命令是可以实现这个功能的。

## 两个主机之间的本地转发

上面提到的 `VSCode` 的例子就是一个两个主机之间的端口转发。我们可以通过以下命令实现 (命令在本地执行)：

```bash
ssh -f -N -L 8080:localhost:80 user@remote-host
# or
ssh -f -N -L 8080:remote-host:80 user@remote-host
```

上面的 `remote-host` 指的是公网 `ip`，
上面两条命令等价是因为第一条命令的 `localhost` 是相对于 `remote-host` 而言的。

上面的命令的意思是将本地 `8080` 端口收到的请求通过 `remote-host` 转发到 `remote-host` (或 `localhost`) 的 `80` 端口上。

**注意**：`-f` 选项表示后台运行。

**注意**：`-N` 选项表示不执行远程命令。

**注意**：你需要保证本地能够和 `user@remote-host` 建立 `ssh` 连接。

**注意**：假设你的本地也有公网 `ip`，那么本地转发默认是不会放行其他主机访问的。

如果你想要其他主机访问你的本地转发，你可以使用 `-g` 选项，
或者在配置中开启 `GatewayPorts` (`/etc/ssh/ssh_config`) 选项，
或者使用 `*:8080:localhost:80` 或 `:8080:localhost:80` 来指定本地转发，
后面的 [GatewayPorts](#gatewayports) 会进行详细介绍。

**注意**：`*:8080:localhost:80` 与 `:8080:localhost:80` 是等价的，`man ssh` 中有这样一句话：

> The bind_address of “localhost” indicates that the listening port be bound for use only, while
an empty address or ‘*’ indicates that the port should be available from all interfaces.

## 三个主机之间的本地转发

在上一部分我对命令的解释中提到了：

> 将本地 `8080` 端口收到的请求 **通过** `remote-host` 转发到 `remote-host` 的 `80` 端口上。

所以当上面命令的 `8080:remote-host:08` 中的 `remote-host` 为第三个主机的 `ip` 地址时，
就是三个主机之间的本地转发。例如 (依然在本地执行)：

```bash
ssh -f -N -L 8080:2.2.2.2:80 user@1.1.1.1
```

上述命令的意思是将本地 `8080` 端口收到的请求通过 `user@1.1.1.1` 转发到 `2.2.2.2` 的 `80` 端口上。
这条命令执行之前需要保证 `1.1.1.1` 可以访问 `2.2.2.2` 的 `80` 端口。

这种情况我能想到的一个例子便是代理服务器。当在公司外部需要访问公司内部的资源的时候，为了保证安全性，
可以使用 `ssh` 的本地端口转发功能。
这样经过外网部分的数据包将被 `ssh` 加密 (上面例子中的 `localhost` <--> `1.1.1.1` 部分)。

**注意**：你需要保证本地能够和 `user@1.1.1.1` 建立 `ssh` 连接。

# 远程转发

远程转发指的是将远程主机的某个端口收到的请求通过本机作为跳板转发到目的主机的某个端口上。

远程转发的一个常见的使用场景是内网穿透。如果你的公司只有一个公网 `ip`，
你不可能把所有的服务全部部署在这一个拥有公网 `ip` 的主机上。
这时候你可以在没有公网 `ip` 的主机上面部署各种不同的服务，
然后通过远程转发将这些服务映射到拥有公网 `ip` 的主机上。

## 两个主机之间的远程转发

我们依然从最简单的部分开始，两个主机之间的远程转发。例如 (在本地执行)：

```bash
ssh -f -N -R 80:localhost:8080 user@remote-host
```

上面的命令依然可以使用 `localhost`，因为此时 `localhost` 是相对与本地主机而言的 (实际上两者都是在相对跳板机而言的)。
上面的命令的意思是将 `remote-host` 的 `80` 端口收到的请求通过本地端转发到 `localhost` 的 `8080` 端口上。

### 三个主机之间的远程转发

当上面的命令中的 `80:localhost:8080` 中的 `localhost` 变成第三台主机的 `ip` 地址时，
就是三个主机之间的远程转发。例如 (在本地执行)：

```bash
ssh -f -N -R 80:198.168.0.2:8080 user@remote-host
```

上面的命令的意思是将 `remote-host` 的 `80` 端口收到的请求通过本地端转发到 `198.168.0.2` 的 `8080` 端口上。

## 题外话

通过远程转发你可以完成一些有趣的东西。例如你的本地的电脑性能非常强悍，但是没有公网 `ip`，
你想要部署一个可以支持很多人访问的网站，那么你可以租一个很便宜的云服务器，
然后通过远程转发将云服务器的 `80` 端口的请求转发到你本地的某个端口上，并把你的网站部署在本地。

**注意**：要实现这个功能，你可能需要了解 `GatewayPorts`，
可以查看后文的 [GatewayPorts](#gatewayports)。

# 动态转发

前面介绍的本地 (远程) 转发都是将本地 (远程) 的某个端口的请求转发到某个固定的目的主机的某个端口上。
而动态转发则是可以实现转发到不同的位置。

我想到关于动态转发的一个例子依然是代理服务器。例如下面的命令 (在本地执行)：

```bash
ssh -f -N -D 1234 user@remote-host
```

上面的命令会让所有到达本地 `1234` 端口的数据包都通过 `remote-host` 转发到目的主机上。
这样你就可以在执行上述命令后，设置代理地址为 `socks5://localhost:1234`，
这样所有的网络请求都会通过 `remote-host` 转发到目的主机上。

例如在完成动态转发的设置后 `curl --proxy socks5://localhost:8080 http://www.google.com` 就会通过 `remote-host` 访问 `google`。

**注意**：`-D` 选项后面的端口号可以设置为任意空闲的非特权 (特权端口需要 `root` 权限) 端口。

**注意**：需要保证本机可以访问 `remote-host`；`remote-host` 可以访问目的主机。

# `GatewayPorts`

`GatewayPorts` 在 `ssh-client` (`/etc/ssh/ssh_config`) 和 `ssh-server` (`/etc/ssh/sshd_config`) 中有不同的含义。

## `ssh-client`

通过 `man ssh_config` 我们可以看到 `GatewayPorts` 在 `ssh-client` 中的含义：

> By default, `ssh` binds local port forwardings to the loopback address. This prevents other
remote hosts from connecting to forwarded ports. `GatewayPorts` can be used to specify that `ssh`
should bind local port forwardings to the wildcard address, thus allowing remote hosts to connect
to forwarded ports. The argument must be `yes` or `no` (the default).

也就是说对于动态转发和本地转发，当设置 `GatewayPorts yes` 时，使用 `8080:localhost:80` 会被解析成
`*:8080:localhost:80` 而当设置成 `GatewayPorts no` 时，
使用 `8080:localhost:80` 会被解析成 `localhost:8080:localhost:80`，也就是是否使用 `-g` 选项的区别。
但是不管设置成 `yes` 还是 `no`，
都可以通过手动指定 `*:8080:localhost:80` 来实现允许所有主机访问本地端口转发。

在 `man ssh` 中有这样一句话：

> Specifying a remote bind_address will only succeed if the server's `GatewayPorts` option is
enabled.

也就是对于远程转发，如果我们想使用类似 `*:80:localhost:8080` 这样的的转发，
我们必须开启 `remote-host` 的 `GatewayPorts` 选项而不是像本地转发一样可以直接使用。
这也是很合理的，
因为远程转发是由 `remote-host` 进行转发，所以应该由 `remote-host` 来决定是否为其他主机访问进行转发，
而不是由本地来决定。

## `ssh-server`

在 `man sshd_config` 中写到了 `GatewayPorts` 的配置方式：

> Specifies whether remote hosts are allowed to connect to ports forwarded for the client. By
default, `sshd` binds remote port forwardings to the loopback address. This prevents other remote
hosts from connecting to forwarded ports. `GatewayPorts` can be used to specify that `sshd` should
allow remote port forwardings to bind to non-loopback addresses, thus allowing other hosts to
connect. The argument may be `no` to force remote port forwardings to be available to the local
host only, `yes` to force remote port forwardings to bind to the wildcard address, or
`clientspecified` to the client to select the address to which the forwarding is bound. The default
is `no`.

简单总结一下：

* `no`：默认值，只允许本地 (这里指远端的本地，也就是远端) 访问转发的端口。
* `yes`：允许其他主机访问转发的端口。
* `clientspecified`：由客户端决定是否允许其他主机访问转发，只有当设置成 `clientspecified` 时，
客户端才能通过 `*:80:localhost:8080` (也可以替换 `*` 为其他想要允许的地址) 的方式来设置转发。

# 巨人的肩膀

* [How to Set up SSH Tunneling (Port Forwarding)](https://linuxize.com/post/how-to-setup-ssh-tunneling/)
* [SSH port forwarding \| SSH Tunnel (Forward & Reverse)](https://www.golinuxcloud.com/setup-ssh-port-forwarding/)
