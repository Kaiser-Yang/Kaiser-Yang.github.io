---
layout: post
title: Linux Potpourri
date: 2024-09-10 20:30:06+0800
last_updated: 2025-05-19 19:42:51
description:
tags:
  - Linux
  - CLI
categories: Potpourri
featured: true
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

**This post is being continuously updated.**

## Shell

**NOTE**: All the contents are based on `bash` shell.

### Shortcuts

* `Ctrl + A`: Move to the beginning of the line.
* `Ctrl + E`: Move to the end of the line.
* `Ctrl + U`: Delete from the cursor to the beginning of the line.
* `Ctrl + K`: Delete from the cursor to the end of the line.
* `Ctrl + W`: Delete the word before the cursor.
* `Ctrl + L`: Clear the screen.
* `Ctrl + Z`: Send SIGTSTP to the foreground process (suspend).
* `Ctrl + C`: Send SIGINT to the foreground process (terminate).
* `Ctrl + \`: Send SIGQUIT to the foreground process (terminate and create core dump).
* `Ctrl + D`: Send EOF to the foreground process (end of file).

### Wildcards

Wildcards are symbols that can be used to match multiple files or directories.
Those symbols are supported by the shell, and they can be used in any command
as long as the shell supports them.

Different shells may have different wildcard symbols,
there are some wildcards in the `bash` shell:

* `*`: matches any number of characters (including 0).
* `?`: matches a single character (excluding `/`).
* `[]`：matches any character in the brackets. For example, `[abc]` matches `a` or `b` or `c`.
* `[!]` or `[^]`: matches any character not in the brackets. For example, `[!abc]` matches any
character except `a` or `b` or `c`.

In the square brackets, you can use `-` to represent a range, for example, `[0-9]`
matches any digit, and `[a-z]` matches any lowercase letter.

There is a symbol `{}` which is not a wildcard but is often used in the same way.
It is used to represent multiple strings, for example, `a{b,c}d` will be parsed as `abd` or `acd`.
For example, `mv file_{old,new}` will be parsed as `mv file_old file_new`. You can use
nested curly brackets, for example, `echo a{b{c,d},e}f` will be parsed as `echo abcf abdf aef`,
which works like distributive law in mathematics. `..` is used to represent a range,
for example, `echo {a..z}` will be parsed as `echo a b c ... z`.

**Note**: these wildcards are not all same with regex. In regex, `*` means 0 or more times,
`?` means 0 or 1 time, and `^` also means the beginning of a line.

### Quoting

Quoting is to cancel the special meaning of some characters in the shell.
For example,
`|`, `&`, `;`, `(`, `)`, `<`, `>`, space, tab, newline and `!` are special characters in the shell.
You must quote them if you want to use them as normal characters.

#### Escape Character

`\` is the escape character in the shell. It can be used to cancel the special meaning of
the next character.

It is worth noting that if the next character is a newline, the newline will be treated
as a line continuation. We use this to avoid writing long commands in one line.

#### Single Quotes

Characters in enclosing single quotes are preserved literally.

**NOTE**: You can not put a single quote between enclosing single quotes
even if preceded by a backslash.
You can put a double quote between enclosing single quotes directly.

#### Double Quotes

Characters in enclosing double quotes are preserved literally,
except for `$`, `` ` ``, `\`, and `!` (when history expansion is enabled).

**NOTE**: When the shell is in POSIX mode, the `!` has no special meaning within  double  quotes,
even when  history  expansion  is  enabled.

`$` and `\`` retain their special meaning within double quotes. For example:

```bash
# $variable will be replaced by the value of the variable
echo "Hello, $USER" # Hello, kaiser
# `command` will be replaced by the output of the command
echo "Today is `date +%A`" # Today is Monday
# \ will be preserved literally unless followed by $ or ` or " or \ or newline
echo "Hello, \$USER" # Hello, $USER
echo "Hello, \"" # Hello, "
echo "Hello, \ no used" # Hello, \ no used
echo "Heloo, \
World" # Hello, World
```

When history expansion is enabled, `!` will be interpreted as a history expansion character:

```bash
ls
# !!: the last command
echo "!!" # ls
```

You can use `$"string"` to translate the string according to the current locale.
If there is no translation available, the string `$` will be ignored.

## `git`

### `.gitignore`

You can use `.gitignore` to specify files or directories that do not need to be tracked by `git`.

Most wildcards in `.gitignore` are similar to those in `bash`.
Check [Wildcards in Linux](#wildcards-in-linux) for more information about `wildcards`.

By default, the items in `.gitignore` will be ignored recursively.
If you don't want to ignore recursively, you can add `/` before the item to indicate
that it only takes effect in the current directory.
For example, `/foo` means to ignore only the `foo` file or directory in the current directory,
while `foo` means to ignore all `foo` files or directories.

By default, the items in `.gitignore` match both directories and files.
If you only want to match directories,
you can add `/` at the end of the item to indicate that it only matches directories.
For example, `foo/` means to match only the directory `foo`,
while `foo` means to match all `foo` files or directories.

You can configure the ignore rules in `~/.config/git/ignore` to ignore files globally.

There can be a `.gitignore` file in any directory in a repository,
and this file will take effect on files and directories in the current directory.
The priority of `.gitignore` files is from subdirectory to parent directory,
and the global ignore file has the lowest priority.

## References

* [10 Practical Examples Using Wildcards to Match Filenames in Linux](https://www.tecmint.com/use-wildcards-to-match-filenames-in-linux/)
* [Ignoring files](https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files)
