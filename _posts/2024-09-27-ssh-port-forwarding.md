---
layout: post
title: SSH Port Forwarding
date: 2024-09-27 19:42:56+0800
last_updated: 2025-05-17 16:19:12+0800
description: This post introduces the three types of SSH port forwarding.
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

`SSH` supports three types of port forwarding:
local forwarding, remote forwarding, and dynamic forwarding.

**NOTE**: The terms "local" and "remote" are relative concepts. In this post, you can
think of "local" as a host without a public IP address,
and "remote" as a host with a public IP address unless otherwise specified.

## Local Forwarding

The local forwarding refers to forwarding requests received on a local port
to a specific port on the destination host through the remote machine as a jump host.

The most common example of using local forwarding is
when you are using `VSCode` to connect to a remote host via `ssh-remote`
for development, and you start a `web` service on your remote machine in `VSCode`.
The `ssh-remote` plugin will automatically open a port forwarding for you,
so you can access the `web` service on your remote server through `localhost` in your browser.
Actually, I haven't researched whether the port forwarding in `VSCode` is implemented
through the `ssh` command, but I know that the `ssh` command can achieve this functionality.

### Local Forwarding between Two Hosts

The example mentioned above is a port forwarding between two hosts.
We can achieve this with the following command (executed locally):

```bash
ssh -f -N -L 8080:localhost:80 user@remote-host
# or
ssh -f -N -L 8080:remote-host:80 user@remote-host
```

`remote-host` in the above command refers to the public `ip`,
and the two commands are equivalent because the first command's
`localhost` is relative to `remote-host`.

The above command means that requests received on the local `8080` port
will be forwarded to the `80` port of `remote-host` (or `localhost`) through `remote-host`.

After running this command, if there is a `HTTP` server running on your remote host, and it's
listening on `80` port, you can access it through `http://localhost:8080`
on your local host's browser.

The options in the command are explained as follows:

* `-f` : Run in the background after authentication.
* `-N` : Do not execute remote commands.
* `-L` : Specify the local port forwarding.

It is worth noting that if your local host also has a public `ip`,
the requests that requests your public `ip` will not be forwarded to the remote host.
If you want these requests to be forwarded to the remote host,
you can use `-g` option, or set `GatewayPorts` in the configuration file (`/etc/ssh/ssh_config`),
or specify `*:8080:localhost:80` or `:8080:localhost:80` in the command.
I will explain this later in the [GatewayPorts](#gatewayports) section.

**NOTE**：Using `*:8080:localhost:80` is equivalent to using `:8080:localhost:80`, from the `MAN`
page of `SSH` we have those below:

> The bind_address of “localhost” indicates that the listening port be bound for use only, while
an empty address or ‘*’ indicates that the port should be available from all interfaces.

### Local Forwarding among Three Hosts

In the previous section, I mentioned in the explanation of the command:

> The requests received on the local `8080` port will be forwarded to the `80` port of `remote-host`
> through `remote-host`.

So if we change the `remote-host` in the above command `8080:remote-host:80` to
the `ip` address of the third host, it will be a local forwarding among three hosts.
For example:

```bash
ssh -f -N -L 8080:2.2.2.2:80 user@1.1.1.1
```

The above command means that requests received on the local `8080` port
will be forwarded to the `80` port of `2.2.2.2` through `1.1.1.1`.

An example of this is when you are using `SSH` to connect to a remote host,
and you want to access a service running on another host in the same network as the remote host.
In this case, you can use the above command to access the service. Note that you should execute the
command on your local machine.

## Remote Forwarding

The remote forwarding refers to forwarding requests received on a remote host's port
to a specific port on the destination host through the local machine as a jump host.

### Remote Forwarding between Two Hosts

We still start from the simplest part, remote forwarding between two hosts:

```bash
ssh -f -N -R 80:localhost:8080 user@remote-host
```

In the above command, `remote-host` refers to the public `ip`,
and the `localhost` in `80:localhost:8080` is relative to the local host.

After running the command, if someone requests the `http://remote-host:80` on his or her browser,
the request will be forwarded to the `8080` port of your local host.

### Remote Forwarding among Three Hosts

Similarly, we can achieve remote forwarding among three hosts by changing the `localhost`
in the above command to the `ip` address of the third host. For example (executed locally):

```bash
ssh -f -N -R 80:198.168.0.2:8080 user@remote-host
```

After running the command, you may think that
if someone requests the `http://remote-host:80` on his or her browser,
the request will be forwarded to the `8080` port of `198.168.0.2` through your local host.
This is almost right, but not exactly. Only the requests that come from the `remote-host`'s
`localhost` will be forwarded by default.

In order to allow other hosts' requests to be forwarded, you must set the `GatewayPorts` option
on the server side. Use the command below:

```bash
ssh -f -N -R :80:198.168.0.2:8080 user@remote-host
# or
ssh -f -N -R *:80:198.168.0.2:8080 user@remote-host
```

See the [GatewayPorts](#gatewayports) section for more details.

### Interesting Thing

The remote forwarding is a bit like a reverse proxy. With remote forwarding, we can achieve
some interesting things.

For example, if your local machine has a very powerful performance but does not have a public `ip`,
and you want to deploy a website that can support many people to access.
Then you can rent a very cheap cloud server, and then use remote forwarding to forward the requests
to the `80` port of the cloud server to a certain port on your local machine,
and deploy your website on your local machine.

To learn more about this, you can refer to [GatewayPorts](#gatewayports).

## Dynamic Forwarding

The local forwarding or remote forwarding are both to forward requests from a certain port
to a specific port on the destination host.
Dynamic forwarding is a bit different. It can forward requests to different locations.
With dynamic forwarding, we can achieve a proxy server.
For example:

```bash
ssh -f -N -D 1234 user@remote-host
```

After running the command, if you set the proxy address to `socks5://localhost:1234`,
all your requests will be proxied by `remote-host`.

In `CLI`, you can use `curl --proxy socks5://localhost:1234 http://www.google.com`
to access `google.com` through `remote-host` after starting the dynamic forwarding.

## `GatewayPorts`

`GatewayPorts` is a configuration option in `SSH`. It is different in
`ssh-client` (`/etc/ssh/ssh_config`) and `ssh-server` (`/etc/ssh/sshd_config`).

### `ssh-client`

We can see the meaning of `GatewayPorts` in `ssh-client` through `man ssh_config`:

> By default, `ssh` binds local port forwardings to the loopback address. This prevents other
remote hosts from connecting to forwarded ports. `GatewayPorts` can be used to specify that `ssh`
should bind local port forwardings to the wildcard address, thus allowing remote hosts to connect
to forwarded ports. The argument must be `yes` or `no` (the default).

For local forwarding, if we set the `GatewayPorts` as `yes`, it means that `8080:localhost:80` will
be parsed as `*:8080:localhost:80`, which means that the port can be accessed by all hosts.
If we set it as `no`, it means that `8080:localhost:80` will be parsed as
`localhost:8080:localhost:80`, which means that the port can only be accessed by the local host.
No matter how we set it, we can manually specify `*:8080:localhost:80` to allow all hosts
to access the local port forwarding.

### `ssh-server`

For the remote forwarding, it is a bit different. We can see this below in `man ssh`:

> Specifying a remote bind_address will only succeed if the server's `GatewayPorts` option is
enabled.

This means if we want use `*:80:localhost:8080` to forward all requests when using remote
forwarding, we must make sure the `GatewayPorts` option is enabled on the remote host.

This is very reasonable, because the remote forwarding is done by the `remote-host`,
so it should be the `remote-host` that decides whether to allow other hosts' requests to be
forwarded.

From `man sshd_config`, we have the following description of `GatewayPorts`:

> Specifies whether remote hosts are allowed to connect to ports forwarded for the client. By
default, `sshd` binds remote port forwardings to the loopback address. This prevents other remote
hosts from connecting to forwarded ports. `GatewayPorts` can be used to specify that `sshd` should
allow remote port forwardings to bind to non-loopback addresses, thus allowing other hosts to
connect. The argument may be `no` to force remote port forwardings to be available to the local
host only, `yes` to force remote port forwardings to bind to the wildcard address, or
`clientspecified` to the client to select the address to which the forwarding is bound. The default
is `no`.

The above description is a bit complicated, but it can be summarized as follows:

* `no` : By default, remote port forwardings are bound to the loopback address,
which means that only the local host can access the forwarded port.
* `yes` : Remote port forwardings are bound to the wildcard address,
which means that all hosts can access the forwarded port.
* `clientspecified` : The client can specify the address to which the forwarding is bound.

## References

* [How to Set up SSH Tunneling (Port Forwarding)](https://linuxize.com/post/how-to-setup-ssh-tunneling/)
* [SSH port forwarding \| SSH Tunnel (Forward & Reverse)](https://www.golinuxcloud.com/setup-ssh-port-forwarding/)
