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

### Variables

You can use `name=[value]` (square braces here mean that the `value` is optional) to define a
variable and set its value. If the `value` is omitted, the empty string is assigned to it.

**NOTE**: You can not put a space between `name` and `=`. For example, `name = value` is invalid.

You can also use `declare` command to define a variable and set its value.
Those below are available `declare` attributes options:

| Option | Meaning |
| --- | --- |
| `-a` | indexed array |
| `-A` | associative array |
| `-i` | integer |
| `-l` | lowercase value automatically |
| `-u` | uppercase value automatically |
| `-r` | readonly |
| `-x` | export |
| `-n` | name reference to another variable |
| `-t` | trace, rarely used |

**NOTE**: You can use `+` instead of `-` to unset an attribute.

If you use `declare -i` to set the integer attribute of a variable, the value will be evaluated
as an arithmetic expression automatically, for example:

```bash
a=1+2
echo $a # 1+2
# You can use $((...)) to evaluate an arithmetic expression
echo $((a)) # 3
declare -i a=1+2
echo $a # 3
```

You can use `+=` to expand a variable:

```bash
declare -i num=5
num+=3
echo $num # 8
num+=2+5
echo $num # 15

str="Hello"
str+=" World"
echo "$str" # Hello World

arr=("apple" "banana")
arr+=("cherry" "date")
echo "${arr[@]}" # apple banana cherry date

arr=([0]="a" [2]="b") # Non-continuous index
arr+=("c") # Appends at index 3 (next max index + 1)
echo "${!arr[@]}" # 0 2 3 (indexes of the array)

# Must use declare -A aarr to define aarr as an associative array,
# which is similar with a dictionary or map
declare -A aarr
aarr=([name]="Alice" [age]=30)
aarr+=([city]="Paris" [job]="Engineer") # Add new keys
echo "${aarr[@]}" # Alice 30 Paris Engineer (unordered)
```

The `*` in the value of a variable is not expanded, but is treated as a normal character.

**NOTE**: The variable can be unset by the `unset` command.

### Positional Parameters

A positional parameter is a parameter denoted by one or more digits,
other than the single digit 0.

Positional parameters are assigned from the shell's arguments when it is invoked,
and may be reassigned using the `set` builtin command.
Positional parameters may not be assigned to with assignment statements.
The positional parameters are temporarily replaced when a shell function is executed.

When a positional parameter consisting of more than a single digit is expanded,
it must be enclosed in braces.

You can update positional parameters with `set` command, for example:

```bash
# Set all the positional parameters with 1 2 3 4
set -- 1 2 3 4

args=("$@") # Copy positional args into an array
args[1]="mango" # Change the second element (index 1)
set -- "${args[@]}" # Reset positional arguments

set -- "$@" "bird" # Append "bird"

set -- "fish" "$@" # Prepend "fish"

# Remove the first positional parameter
# After this, the $1 is the value of original $2
shift # or shift 1

args=("$@")
unset 'args[2]' # Remove the second positional parameter
set -- "${args[@]}"
```

**NOTE**: `0` is not a positional parameter, it is a special parameter, which will be expanded to
the name of the shell or shell script.

### Special Parameters

* `$*`: All positional parameters, each of which expands to a separate word.
* `"$*"`: A single string with all positional parameters
separated by the first character of the `IFS` variable.
If `IFS` is unset, the parameters are separated by spaces.
If `IFS` is null, the parameters are joined without intervening separators.
* `$@`: All positional parameters, each of which expands to a separate word.
* `"$@"`: Equivalent to `"$1" "$2" ... "$N"` (where N is the number of positional parameters).
`prefix"$@"suffix` will be parsed as `prefix"$1" "$2" ... "$N"suffix`.
* `$#`: The number of positional parameters in decimal.
* `$?`: The exit status of the most recently executed foreground pipeline.
* `$$`: The process ID of the shell. In a sub-shell,
it expands to the process ID of the current shell, not the sub-shell.
* `$!`: The process ID of the job most recently placed into the background.
* `$0`: The name of the shell or shell script.
If bash is invoked with a file of commands, `$0` is set to the name of that file.
If bash is started with the `-c` option, 
then `$0` is set to the first argument after the string to be executed, if one is present.
Otherwise, it is set to the filename used to invoke bash, as given by argument zero.
* `$_`: The last argument of the previous command or script path.
* `$-`: The current option flags as specified upon invocation,
by the set builtin command, or those set by the shell itself (such as the -i option).

The characters of `$-` and their meanings:

| Flag | Meaning |
| --- | --- |
| `h` | hashall (remembers command locations in `$PATH`) |
| `i` | interactive shell |
| `m` | monitor mode (job control enabled) |
| `H` | history expansion enabled (e.g., `!!` expands to the last command) |
| `B` | brace expansion enabled (e.g., `{a, b}` expands to `a b`) |
| `s` | compounds read from stdin (e.g., `bash -s` |


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
