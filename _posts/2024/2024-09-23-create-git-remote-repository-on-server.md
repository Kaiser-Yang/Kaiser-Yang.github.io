---
layout: post
title: Creating a Git Remote Repository on Server
date: 2024-09-23 11:12:56+0800
last_updated: 2025-05-26 10:43:16
description:
tags:
  - English Posts
  - git
categories: Potpourri
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

This article will introduce how to create a central `git` server on a remote server
and configure `ssh` connections.

## Requirements

* `git`
* `ssh`

## Create a `git` User

On remote server, we usually create a user named `git` to manage the `git` repositories.
We can use the following command to create a `git` user:

```bash
# -m to create a home directory for the user
useradd -m git
```

Then we need to set a password for the `git` user:


```bash
# This command may require your `sudo` password the first time,
# then you will be prompted to enter the new password twice
sudo passwd git
```

## Configuration

### Create a `.ssh` Directory

We need to set up `ssh` for the `git` user so that we can connect to the `git` server via `ssh`.

```bash
# Switch to the `git` user, if not already done
su git
# Make sure the ~/.ssh directory and authorized_keys file exist and have the correct permissions
cd ~
mkdir -p .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

**NOTE**: You must make sure that your remote server has `ssh` installed and running.

### Configure Login Shell

We usually do not want the `git` user to have a normal shell login,
so we can set the `git` user's shell to `git-shell`,
which restricts the user to only use `git` commands.

To do this, we first need to check if `git-shell` is listed in `/etc/shells`:

```bash
# Check if git-shell is already in /etc/shells
cat /etc/shells | grep git-shell
# If it is not listed, we can add it
echo $(which git-shell) | sudo tee -a /etc/shells
```

Then, we can update the `git` user's shell to `git-shell`:

```bash
sudo chsh -s $(which git-shell) git
```

### Forbid Forwarding

We usually want to restrict the `git` user from performing port forwarding,
X11 forwarding, agent forwarding, and pseudo-terminal allocation.

We can do this by modifying the `~/.ssh/authorized_keys` file:

```bash
# We may need to add the following line to every public key in the authorized_keys file
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty ssh-rsa AAAAAA...
```

## Upload a Public Key

Now, we need to upload our public key to the `git` server so that we can connect to it via `ssh`.
We just need to append the content of our local public key
to the `git` user's `~/.ssh/authorized_keys` file.

**NOTE**: Do not forget to add `no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty `
before the public key in the `authorized_keys` file.

## Create a Remote Bare Repository

Firstly, we need to switch to the `git` user:

```bash
# You may need to enter the `git` user's password,
# or you can switch to the `git` user using `sudo su git`, which will require the `sudo` password
su git
```

Then, we can create a bare repository named `a.git` in the home directory of the `git` user:

```bash
# Specifying the path as a bare repository
git init --bare /home/git/a.git
```

**NOTE**: A bare repository is a repository that does not have a working directory,
which means it only contains a `.git` directory to store the version control information.

## Clone the Remote Repository

If you have completed the above steps, you can now clone the remote repository.
Supposing the server's domain is `1.1.1.1` and the port is `123`,
the user is `git`, and the repository path is `~/a.git`,
we can clone the repository using the following command:

```bash
git clone ssh://git@1.1.1.1:123/~/a.git
```

We can use `git remote -v` to check the remote repository address after cloning.

We can use the following command to add a remote repository to an existing local repository:

```bash
# The `origin` is the name of the remote repository
git remote add origin ssh://git@1.1.1.1:123/~/a.git
```

## Local `~/.ssh/config` Configuration

We can simplify the `ssh` connection by configuring the `~/.ssh/config` file on our local machine.

For example, if we want to connect to the `git` server at `1.1.1.1`
on port `123` using the user `git`,
we can add the following configuration to our local `~/.ssh/config` file:

```bash
Host gitserver
  HostName 1.1.1.1
  User git
  Port 123
```

After this configuration, we can clone the remote repository using a simpler command:

```bash
# No need to specify the `~`, as the `~` is the default path
git clone gitserver:a.git
```

## References

* [Git on the Server - Setting Up the Server](https://git-scm.com/book/en/v2/Git-on-the-Server-Setting-Up-the-Server)
