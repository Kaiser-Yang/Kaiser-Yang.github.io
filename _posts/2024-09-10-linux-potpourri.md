---
layout: post
title: Linux Potpourri
date: 2024-09-10 20:30:06+0800
last_updated: 2025-06-07 13:36:02+0800
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
* `**`: matches all files and directories recursively.
* `**/`: matches all directories recursively.
* `?`: matches a single character.
* `[]`：matches any character in the brackets. For example, `[abc]` matches `a` or `b` or `c`.
* `[!]` or `[^]`: matches any character not in the brackets. For example, `[!abc]` matches any
character except `a`, `b` and `c`.

**NOTE**: `**` and `**/` are only available when `globstar` shell option is enabled
(use `shopt | grep globstar` to check). You can use `shopt -s globstar` to enable it.

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

When `extglob` is on (use `shopt | grep extglob` to check), those below are supported:

* `?(pattern-list)`: Matches zero or one occurrence of the given patterns.
* `*(pattern-list)`: Matches zero or more occurrences of the given patterns.
* `+(pattern-list)`: Matches one or more occurrences of the given patterns.
* `@(pattern-list)`: Matches one of the given patterns.
* `!(pattern-list)`: Matches anything except one of the given patterns.

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
aarr+=([city]="Paris" [job]="Engineer") # Add new key-value pairs
echo "${aarr[@]}" # Alice 30 Paris Engineer (unordered)
```

The `*` in the value of a variable is not expanded, but is treated as a normal character.
You can use `(*.txt)` to expand to all `.txt` files.

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

### Arrays

An indexed array is created automatically
if any variable is assigned to using the syntax `name[subscript]=value`.
The subscript is treated as an arithmetic expression that must evaluate to a number.

For an indexed array, you can use negative index, which will count back from the end of the array.
For example, `a[-1]` means the last element, and `a[-2]` means the second to last.

You can reference an element of any array with `${name[subscript]}`.
You can not omit the braces.

You can use `${name[@]}` or `${name[*]}` to get all assigned values,
and `${!name[@]}` or `${!name[@]}` to get all assigned indices.
They difference between them when they are double-quoted
is similar with the one between `"$@"` and `"$*"`.

`${#name[subscript]}` expands to the length of `${name[subscript]}`.
If subscript is `*` or `@`, the expansion is the number of elements in the array.

`$name` is equivalent to `${name[0]}`.

`unset ${name[0]}` can destroy the first element of the array.

`unset name` or `unset ${name[@]}` or `unset ${name[*]}` will removes the entire array.

You can use `declare -a name` to create an array called `name`.

You can use `declare -A name` to create an associative array called `name`.

`declare -a -A name` is equivalent to `declare -A name`.

For an associative array,
Using `name=( key1 value1 key2 value2...)` and `name=( [key1]=value1 [key2]=value2 ...)`
to assign value are both OK. But you can not mixed these two types like
`myarr=( key1 value1 [key2]=value2 )`.
If you leave off a value at the end, it's treated as the empty string.
In `declare -A myarr=( key1 value1 key2 )`, `myarr[key2]` is an empty string.

When using a variable name with a subscript as an argument to a command,
such as with unset (`unset arr[i]`),
without using the word expansion syntax described above (`unset ${arr[i]}`),
the argument is subject to pathname expansion (expands to `unset arri`).
If pathname expansion is not desired, the argument should be quoted
(`unset 'arr[i]'` or `unset "arr[i]"`).

### Expansion

The order of expansions is:

* brace expansion;
* tilde expansion, parameter and variable expansion,
arithmetic expansion, command substitution (done in a left-to-right fashion)
and process substitution (if the system supports it);
* word splitting;
* pathname expansion.

After these expansions are performed,
quote characters present in the original word are removed
unless they have been quoted themselves (quote removal).

#### Brace Expansion

This is the expansion of `{}` see [Wildcards](#wildcards).

#### Tilde Expansion

* `~`: Current user's home (`$HOME`).
* `~username`: Home directory of `username`.
* `~+`: Current working directory (`$PWD`).
* `~-`: Previous working directory (`$OLDPWD`).
* `"~"` or `'~'`: Literal `~` (no expansion).
* `~+number`: The `number`-th entry of the output `dirs` (0-indexed).
* `~-number`: The `number`-last entry of the output `dirs` (1-indexed).
* `~number`: Same as `~+number`.

**NOTE:** You can use `pushd` to add directory to `dirs` and `popd` to remove directory from `dirs`.

#### Parameter and Variable Expansion

* `${parameter}`: The value of parameter is substituted. Sometimes, the braces can be omitted.
* `${!parameter}`: Expands to the value of the variable named by `parameter`.
For example:

```bash
foo='Hello'
bar='foo'
echo "${!bar}" # Hello
```

* `${!nameref}`: Expands to the referenced name, for example:

```bash
declare -n nameref_var="target_var"  # nameref_var is a reference to target_var
target_var="Hello"

echo "$nameref_var" # Hello (dereferences automatically)
echo "${!nameref_var}" # target_var (returns the referenced name)
```

* `${!prefix*}` or `${prefix@}`: Expands to the values of variables whose names begin with `prefix`.
For example:

```bash
a=1
aa=11
aaa=111
echo "${!a*}" # 1 11 111 (all variables starting with a)
echo "${!a@}" # 1 11 111 (all variables starting with a)
```

* `${parameter:offset}`: Expands to the substring of the value of `parameter`
starting at the character specified by `offset`.
* `${parameter:offset:length}`: Expands to the substring of the value of `parameter`
starting at the character specified by `offset` and extending for `length` characters.
For examples:

```bash
# Basic usage
str="Hello, World!"
echo "${str:7}" # World! (substring starting at index 7)
echo "${str:7:5}" # World (substring starting at index 7 and length 5)
echo "${str:7:-1}" # World (substring starting at index 7 and end at -1)
# The space between `:` and `-` is required to avoid confusion with the `:-` expansion
echo "${str: -6}" # World! (substring starting at index -6)
echo "${str: -6:5}" # World (substring starting at index -6 and length 5)
echo "${str: -6:-1}" # World (substring starting at index -6 and end at -1)

# For @
set -- A B C D E
echo "${@:0:1}" # the name of the script or the shell
echo "${@:2}" # B C D E (substring starting at index 2)
echo "${@:2:3}" # B C D (substring starting at index 2 and length 3)
echo "${@: -3}" # C D E (substring starting at index -3)
echo "${@: -3:2}" # C D (substring starting at index -3 and length 2)
# echo "${@:2:-1}" # Error, the length can not be negative for @

# For indexed array
arr=(A B C D E)
echo "${arr[@]:2}" # C D E (substring starting at index 2)
echo "${arr[@]:2:3}" # C D E (substring starting at index 2 and length 3)
echo "${arr[@]: -3}" # C D E (substring starting at index -3)
echo "${arr[@]: -3:2}" # C D (substring starting at index -3 and length 2)
# echo "${arr[@]:2:-1}" # Error, the length can not be negative for an indexed array

# Undefined results for associative array
```

* `${!array[@]}` or `${!array[*]}`: Expands to the indices of the array `array`.
* `${#parameter}`: The length of the value of `parameter` is substituted.
* `${#*}` or `${#@}`: Same with `$#`: the number of positional parameters.

For those below, you can remove `:` to make it only work for unset variables:

* `${parameter:-word}`: Expands to `word` if `parameter` is unset or null;
otherwise, it expands to the value of `parameter`.
* `${parameter:=word}`: Assigns `word` to `parameter` and expands to `word`
if `parameter` is unset or null;
otherwise, it expands to the value of `parameter`.
You can not use this to positional parameters.
* `${parameter:?word}`: `word` is written to standard error if `parameter` is unset or null,
if it is not interactive, exits;
otherwise, it expands to the value of `parameter`.
* `${parameter:+word}`: Nothing is substituted if `parameter` is null or unset;
otherwise, the expansion of `word` is substituted.

For those below,
if `parameter` is `@` or `*` or an array subscripted with `@` or `*`,
the pattern removal operation is applied to each element in turn,
and the expansion is the resultant list:

* `${parameter#word}`: Removes the shortest match of `word` from the beginning of `parameter`.
Wildcards are allowed in `word`. See [Wildcards](#wildcards).
* `${parameter##word}`: Removes the longest match of `word` from the beginning of `parameter`.
Wildcards are allowed in `word`. See [Wildcards](#wildcards).
* `${parameter%word}`: Similar to `${parameter#word}`
but removes the suffix instead of the prefix.
* `${parameter%%word}`: Similar to `${parameter##word}`
but removes the suffix instead of the prefix.
* `${parameter@U}`: Converts the value of `parameter` to uppercase.
* `${parameter@u}`: Converts the first character of the value of `parameter` to uppercase.
* `${parameter@L}`: Converts the value of `parameter` to lowercase.
* `${parameter@a}`: Expands to the attributes of `parameter`.
* `${parameter@E}`: Expands to a string with all the escaped characters expanded
(such as `\n` -> newline).
* `${parameter@A}`: Expands to a string whose value,
if evaluated,
will recreate `parameter` with its attributes and value.
If used for array variables, you should use `${a[@]@A}` to get the string.
* `${parameter@Q}`: Expands to a single-Quoted string with any special characters
(such as `\n`, `\t`, etc.) escaped.
For examples:

```cpp
a='Hello World'
b=('Hello' 'World')
declare -A c=([first]='Hello' [second]='World')
echo "${a@Q}" # 'Hello World'
echo "${b[@]@Q}" # 'Hello' 'World'
echo "${c[@]@Q}" # 'World' 'Hello' (unordered)
```

* `${parameter@K}`: Similar to `${parameter@Q}`, but this will
print the values of indexed and associative arrays as a sequence of quoted key-value pairs.
For examples:

```cpp
a='Hello World'
b=('Hello' 'World')
declare -A c=([first]='Hello' [second]='World')
echo "${a@K}" # 'Hello World'
echo "${b[@]@K}" # 0 "Hello" 1 "World"
echo "${c[@]@K}" # first "Hello" second "World"
```

* `${parameter@P}`: Expands as if it were a prompt string.
For examples:

```bash
PS1='\u@\h:\w\$ '
echo "${PS1@P}" # user@host:/path$  (expands prompt codes)
```

* `${parameter/pattern/string}`: Replace the longest match of `pattern` with `string`.
If `pattern` begins with `/`, all matches of `pattern` are replaced with `string`.
If `pattern` begins with `#`, it must match at the beginning of the expanded value of `parameter`.
If `pattern` begins with `%`, it must match at the end of the expanded value of `parameter`.
If `string` is null,
matches of `pattern` are deleted and the `/` following `pattern` may be omitted.
If the `nocasematch` shell option is enabled,
the match is performed without regard to the case of alphabetic characters.
* `${paramter^pattern}`: Convert the first match of `pattern` to uppercase.
The `pattern` can only match one character.
If `pattern` is omitted, it is treated like a `?`, which matches every character.
Wildcards are allowed in `pattern`.
See [Wildcards](#wildcards).
* `${paramter^^pattern}`: Convert all matches of `pattern` to uppercase.
The `pattern` can only match one character.
If `pattern` is omitted, it is treated like a `?`, which matches every character.
Wildcards are allowed in `pattern`.
See [Wildcards](#wildcards).
* `${paramter,pattern}`: Convert the first match of `pattern` to lowercase.
The `pattern` can only match one character.
If `pattern` is omitted, it is treated like a `?`, which matches every character.
Wildcards are allowed in `pattern`.
See [Wildcards](#wildcards).
* `${paramter,,pattern}`: Convert all matches of `pattern` to lowercase.
The `pattern` can only match one character.
If `pattern` is omitted, it is treated like a `?`, which matches every character.
Wildcards are allowed in `pattern`.
See [Wildcards](#wildcards).

#### Arithmetic Expansion

* `$$(expression)`: The value of `expression` is substituted.

#### Command Substitution

* `$(command)` or `` `command` ``: The standard output of `command` is substituted
with trailing newlines deleted.

#### Process Substitution

* `<(command)`: Provides the output of `command` as a file that can be read from.
* `>(command)`: Provides a file that, when written to, becomes the input for `command`

#### Word Splitting

The part depends on the `IFS` variable.

The shell scans the results of parameter expansion, command substitution,
and arithmetic expansion that did not occur within double quotes for word splitting.

You can put variables in double quotes to prevent word splitting, for examples:

```bash
a='1 2   3'
echo $a # 1 2 3
echo "$a" # 1 2   3

set -- $a
echo $# # 3
set -- "$a"
echo $# # 1
```

The `IFS` default to `space`, `tab`, and `newline`.

#### Pathname Expansion

See [Wildcards](#wildcards).

#### Quote Removal

After the preceding expansions,
all unquoted occurrences of the characters `\`, `'`, and `"`
that did not result from one of the above expansions are removed.

## Environment Variables

### Locality

| Variable | Description |
| --- | --- |
| `LC_ALL` | Overrides all locale settings |
| `LC_CTYPE` | Character classification and case conversion |
| `LC_COLLATE` | String collation order |
| `LC_MESSAGES` | Language for system messages |
| `LC_TIME` | Date and time formatting |
| `LC_NUMERIC` | Number formatting |
| `LC_MONETARY` | Currency formatting |
| `LC_PAPER` | Paper size and format |
| `LC_NAME` | Name formatting |
| `LC_ADDRESS` | Address formatting |
| `LC_TELEPHONE` | Telephone number formatting |
| `LC_MEASUREMENT` | Measurement units |
| `LC_IDENTIFICATION` | Locale identification |
| `LANG` | Default locale setting |

The priority of locale settings:

```bash
LC_ALL > LC_* (specific category) > LANG
```

**NOTE**: There is another variable called `LANGUAGE`
which is used to specify the language priority list for messages.

## Commands

### `awk`

| 选项 | 说明 |
| ---  | --- |
| `-F` | 指定输入的分隔符。 |
| `-f` | 指定 `awk` 脚本文件。 |

#### 打印列

* `$0`: 所有列。
* `$1`: 第一列。
* `...`
* `$NF`: 最后一列。

例如可以使用 `awk '{print $1,$2,$NF}'` 打印出每行的第一列、第二列和最后一列。

#### 分隔符

可以通过 `OFS=` (Output Field Separator) 来指定输出的分隔符，例如 `awk '{print $1,$2,$NF}' OFS=","`
表示使用 `,` 作为分隔符。也可以在 `BEGIN` 模式串中指定 `OFS`，例如 `awk 'BEGIN {OFS=","} {print $1,$2,$NF}'`。

除了 `OFS` 以外，还有 `FS` 用于指定输入文件的分隔符。

#### 预定义变量

除了之前提到的 `OFS`, `FS` 之外，还有一些预定义变量：

* `NR`: 当前行的行号。
* `NF`: 当前行的列数。
* `RS`：记录分隔符，默认是换行符，也就是每一行作为一条记录。
* `ORS`：输出的记录分隔符，默认是换行符。
* `FILENAME`：当前文件的文件名。
* `FNR`：当前文件的行号，当时用多个文件的时候，`NR` 记录的是当前的总行号，
`FNR` 记录的是当前文件的行号。

我们可以使用 `awk 'NR > 1'` 从第二行开始打印，也可以使用 `awk 'NF > 0'` 打印列数大于 `0` 的行
(即移除空行)。

如果要打印第一行到第四行，我们可以通过 `awk 'NR == 1, NR == 4'` 来实现。
我们也可以使用 `awk 'NR >= 1 && NR <= 4'` 来实现。

#### 模式

`BEGIN` 和 `END` 模式串：

* `BEGIN`: 在处理输入之前执行。
* `END`: 在处理输入之后执行。

例如可以使用
`awk 'BEGIN {print "Start"} {print} END {print "End"}'`
在处理输入之前和之后打印出 `Start` 和 `End`。

在 `awk` 中可以使用模式来过滤行，
例如 `awk '$1 > 10'` 表示只打印第一列大于 `10` 的行 (当不书写动作时，默认动作是打印整行)。
也可以使用搜索模式，例如 `awk '/pattern/'` 表示只打印包含 `pattern` 的行，搜索模式支持正则表达式。

`,` 也可以和搜索模式一起使用，
例如 `awk '/pattern1/,/pattern2/'` 表示打印找到的 `pattern1` 到 `pattern2` 的行。

`~` 可以用来表示是否与搜索模式匹配，例如 `awk '$1 ~ /pattern/'` 表示第一列是否包含 `pattern`。
`!~` 可以用来表示是否不匹配，例如 `awk '$1 !~ /pattern/'` 表示第一列是否不包含 `pattern`。

#### `awk` 脚本

我们也可以书写 `awk` 脚本实现更加复杂的功能，例如下面的脚本可以实现统计文件中每个单词出现的次数：

```awk
#! /usr/bin/awk -f

BEGIN {
    # 设置输入和输出的分隔符
    FS=":"
    OFS=" "
    tot_count = 0
}
{
    for (i = 1; i <= NF; i++) {
        words[$i]++
        tot_count++
    }
}
END {
    print "Total words:", tot_count
    for (word in words) {
        print word, words[word]
    }
}
```

**注意**：由于在 `awk '{print}' file` 的意思是对某个文件的内容进行处理，
所以要用 `awk -f file` 来表示将文件作为脚本执行。

#### 内置函数

`awk` 中还有一些内置函数，例如 `length` 函数可以返回字符串的长度，`substr` 函数可以返回字符串的子串，
我们可以使用下面的命令计算最后一列 (从第二行起) 的数字和：

```awk
awk 'NR > 1 { printf "%s",$NF"+" }' OFS="" | awk '{ print substr($0, 1, length($0) - 1) }' | bc
```

在 `awk` 中，`print` 会在输出的字符串后面自动添加换行符，而 `printf` 可以进行格式化输出，
上面的例子中我们通过 `printf` 来输出不带换行符的字符串。在 `awk` 中字符串是可以直接拼接的，
例如 `printf "%s", $NF"+"` 表示将最后一列的值和 `+` 拼接在一起。

`printf` 的格式化方法与 `C/C++` 中的类似，这里不再进行详细介绍。

当然要实现同样的功能有更简单的命令：

```awk
awk 'NR > 1 { sum+=$NF } END { print sum }'
# or
awk 'NR > 1 { print sep $NF; sep="+" }' OFS="" ORS="" | bc
```

**注意**：在 `awk` 中，下标是从 `1` 开始的，而不是从 `0` 开始的，且区间是闭区间。

这里再列出一些常用的内置函数：

* `tolower`：将字符串转换为小写。
* `toupper`：将字符串转换为大写。
* `split`：将字符串按照某个分隔符分割为数组。接收三个参数，第一个参数是要分割的字符串，
第二个参数是数组名，第三个参数是分隔符。在分隔符部分，我们可以传入搜索表达式，
例如 `split($0, words, /:+/)` 表示将当前行按照 `:` (或者多个连续的 `:`) 分割为数组。
* `gsub`：全局替换。接收三个参数，第一个参数是查找字符串，第二个参数是替换后的字符串，
第三个参数要替换的文本。
例如 `gsub(/pattern/, "replace", $0)` 表示将当前行中的 `pattern` 替换为 `replace`。
* `system`：执行系统命令。例如 `system("ls")` 会执行 `ls` 命令并将结果输出到标准输出。

#### 改变分隔符

前面介绍到 `FS` 和 `OFS` 可以指定输入和输出的分隔符。如果我们想要直接改变分隔符后输出，
我们可能会写出 `awk '{print}' FS=':' OFS=' '` 这样的命令，试图将 `:` 改成空格。
但是这样并不能正确工作，
在 `awk` 中只有当列被修改后 (或者在 `print` 中打印多个 `fields` 的时候) 才会使用新的输出分隔符，
所以我们可以通过 `awk '($1=$1) || 1' FS=':' OFS=' '` 来实现 (这里省略了动作，因此会打印一整行)。

这里的 `($1=$1) || 1` 是为了保证能够成功输出原始的空行，因为空行在赋值后返回的是空字符串，而空字符串在
`awk` 中被当作 `false`，所以我们需要通过 `|| 1` 来保证输出。这里的括号是必须的，如果没有括号，
`$1=$1 || 1` 会被解释为 `$1=($1 || 1)`，这样会将 `$1` 赋值为 `1`，而不是保留原来的值。

题外话：`awk` 的名字来源于三个创始人的名字 `Alfred Aho`、`Peter Weinberger` 和 `Brian Kernighan` 的首字母。

#### `POSIX` 字符类

`awk` 中支持 `POSIX` 字符类：

* `[:alnum:]`：字母和数字。
* `[:alpha:]`：字母。
* `[:blank:]`：空格和制表符。
* `[:cntrl:]`：控制字符。
* `[:digit:]`：数字。
* `[:graph:]`：可打印字符，不包括空格。
* `[:lower:]`：小写字母。
* `[:print:]`：可打印字符，包括空格。
* `[:punct:]`：标点符号。
* `[:space:]`：空白字符。
* `[:upper:]`：大写字母。
* `[:xdigit:]`：十六进制数字。

例如，我们可以使用 `awk '/[[:digit:]]/'` 来匹配包含数字的行。当然也可以写成 `awk '/[0-9]/'`。

#### 指定文件大小

`-size` 参数有以下几种单位：

* `b`：块，取决于文件系统，默认是 `512` 字节
* `c`：字节
* `k`：千字节 (1024 字节)
* `M`：兆字节 (1024 千字节)
* `G`：吉字节 (1024 兆字节)

知道了上述单位后，查找某个固定大小的文件只需要指定大小和单位即可，
例如 `find . -size 1M` 表示查找大小为 `1` 兆字节的文件。

但是通常我们会查找大于或者小于某个大小的文件，这时候我们可以使用 `+` 和 `-` 来表示大于和小于，
例如 `find . -size +1M` 表示查找大于 `1MB` 的文件，`find . -size -1M` 表示查找小于 `1MB` 的文件。
这两者也可以结合使用，例如 `find . -size +1M -size -2M` 表示查找大于 `1MB` 且小于 `2MB` 的文件。

#### 对查找到的文件执行操作

`-exec` 可以对查找到的文件执行操作，
例如 `find . -name "*.txt" -exec cat {} \;` 表示查找当前目录下的所有 `txt` 文件并将其内容输出到标准输出。
`{}` 会被替换为查找到的文件名，`\;` 表示结束，分号需要进行转义，防止被 `Shell` 解释。

也可以使用 `grep` 命令过滤查找到的文件，
例如 `find . -name "*.txt" -exec grep "pattern" {} \;` 表示查找当前目录下的所有 `txt` 文件并在其中查找 `pattern`。

### `cat`

| Option | Description |
| ---  | --- |
| `-n` | Show line numbers |
| `-b` | Only show non-empty lines numbers |
| `-s` | Suppress repeated empty lines |
| `-v` | Use `^` and `M-` to display non-printable characters, except for tab and newline |
| `-E` | Display `$` at the end of each line |
| `-T` | Display `^I` for tab characters |
| `-A` | Equivalent to `-vET` |
| `-e` | Equivalent to `-vE` |
| `-t` | Equivalent to `-vT` |

You can use `cat` to read from standard input then output to a file.
For example, `cat > file` will read from standard input and write to `file`
(use `^D` to send `EOF`); `cat >> file` can be used to append content to `file`.

You can put `-` as a file name to read from standard input.
For example, `cat file1 - file2` will read from standard input and output the content of `file1`,
then the content of standard input, and finally the content of `file2`;
`cat file1 - file2 - file3` will read from standard input and output the content of `file1`,
then the content of standard input, then the content of `file2`, and finally the content of `file3`.
In this process, you will be required to input the content twice,
you can use `^D` to end the first input,
then input the second content and use `^D` to end it again.

There is another command `zcat`, which is similar to `cat` but used for compressed files.

**NOTE**: `tac` is a command that is similar to `cat`,
but it outputs the contents of files in reverse order.

**Digression**: The name `cat` comes from concatenate.

### `curl`

### `cut`

### `grep`

| Option | Description |
| --- | --- |
| `-A 2` | Show matching lines and the next two lines |
| `-B 2` | Show matching lines and the previous two lines |
| `-C 2` | Show matching lines and two lines before and after |
| `-r`   | Recursively search directories |
| `-n`   | Show line numbers |
| `-i`   | Ignore case |
| `-v`   | Invert match |
| `--include "*.py"` | Search only in files matching the pattern |
| `--exclude "test*"` | Skip files matching the pattern |
| `--exclude-dir "test*"` | Skip directories matching the pattern |
| `-c` | Show the count of match in each file instead of the matching lines |
| `-o` | Only show the matching part |
| `-l` | Show the names of files with matches instead of the matching lines |
| `-L` | Show the names of files without matches instead of the matching lines |
| `-e` | Specify a pattern explicityly |
| `-w` | Only match whole words |
| `-x` | Only match whole lines |
| `-F` | Interpret the pattern as a fixed string, not a regex |
| `-H` | Show file names in output (default when multiple files are searched) |
| `-h` | Do not show file names in output (default when a single file is searched) |
| `-m` | Stop after N matches for each file |
| `--color=auto,always,never` | Highlight rules |
| `-f` | Read patterns from a file, one pattern per line |
| `-E` | Interpret the pattern as an extended regular expression (ERE) |

The difference between extended regular expressions (ERE) and basic regular expressions (BRE)
is that in ERE, `?`, `+`, `{`, `|`, `(`, and `)` are special characters,
while in BRE, they are not special characters unless escaped with a backslash (`\`).

**NOTE**: `-e` is useful when you
want to specify multiple patterns or the pattern starts with a hyphen (`-`).

**NOTE**: We usually use `egrep` as an alias for `grep -E`, and `fgrep` as an alias for `grep -F`.

**NOTE**: Word-constituent characters are letters, digits, and underscores.

**NOTE**: In shell command, you usually need to quote the pattern to prevent pathname expansion.

**Regression**: `grep` comes from the command `g/re/p`,
where `g` stands for `global`, `re` stands for `regular expression`, and `p` stands for `print`.

### `sed`

| Option | Description |
| --- | --- |
| `-i[SUFFIX]` | Edit files in place, optionally with a backup suffix |
| `-e` | Add a script to the commands to be executed |
| `-f` | Add a script file to the commands to be executed |
| `-E` | Use extended regular expressions |
| `-n` | Suppress automatic printing of pattern space |

**NOTE**: With `-n`, `sed` will only print lines explicitly specified with the `p`, `P`, or `w`.

#### Pattern Space and Hold Space

`sed` uses two spaces: the pattern space and the hold space.

The pattern space is where the current line is processed.
`sed` will read one line at a time from the input into the pattern space.

The hold space is a temporary storage area that can be used to store data between commands.
It is empty until you explicitly use some commands to store data in it.
And content in the hold space persists across multiple lines.

#### Commands

| Command | Description |
| --- | --- |
| `s` | Substitute a pattern in the pattern space |
| `p` | Print the pattern space |
| `P` | Print the first part of the pattern space (up to the first newline) |
| `d` | Delete the pattern space |
| `D` | Delete the first part of the pattern space (up to the first newline) |
| `q` | Quit `sed` immediately |
| `h` | Copy the pattern space to the hold space |
| `H` | Append the pattern space to the hold space |
| `g` | Copy the hold space to the pattern space |
| `G` | Append the hold space to the pattern space |
| `n` | Read the next line into the pattern space |
| `N` | Append the next line to the pattern space |
| `x` | Exchange the pattern space and the hold space |
| `=` | Print current line number |
| `w` | Write the pattern space to a file |
| `W` | Write the first part of the pattern space (up to the first newline) to a file |
| `a\` | Append text after the current line (use `\` to continue on the next line) |
| `c\` | Change the current line to the specified text (use `\` to continue on the next line) |
| `i\` | Insert text before the current line (use `\` to continue on the next line) |

**NOTE**: The `a`, `c`, and `i` are GNU `sed` extensions. It can be used without `\` but only
for a single line of text.

#### Examples for `s` command

```bash
# Replace the first occurrence of "old" with "new" in each line
sed 's/old/new/' file.txt
 # Replace all occurrences of "old" with "new" in each line
sed 's/old/new/g' file.txt
# Replace all occurrences of "old" with "new" in each line, ignoring case
sed 's/old/new/gi' file.txt
# Replace the second occurrence of "old" with "new" in each line
sed 's/old/new/2' file.txt
# Replace "old" with "new" in each line from the second occurrence to the end of the line
sed 's/old/new/2g' file.txt
# Replace the first occurrence of "old" with "new" in every 10th line
sed '1~10s/old/new/' file.txt
# Use | as the delimiter instead of /
# Some available delimiters are: /, |, #, @, !, and +
sed 's|/var/log|/var/logs|g' file.txt
# Replace the first occurrence of "old" with "new" in the 5th line
sed '5s/old/new/' file.txt
# Replace the first occurrence of "old" with "new" in lines 5 to 10
sed '5,10s/old/new/' file.txt
# Replace all occurrences of "old" with "new" in lines matching "pattern"
sed '/pattern/s/old/new/g' file.txt
# Replace all occurrences of "old" with "new" in lines matching "pattern" and all following lines
sed '/pattern/,$s/old/new/g' file.txt
# Replace all occurrences of "old" with "new" in lines between "start_pattern" and "end_pattern"
sed '/start_pattern/,/end_pattern/s/old/new/g' file.txt
```

### `find`

| Option | Description |
| --- | --- |
| `-type` | Specify the type of file to search for |
| `-readable` | Search for files or directories that are readable by the current user |
| `-writable` | Search for files or directories that are writable by the current user |
| `-executable` | Search for files or directories that are executable by the current user |
| `-name` | Specify the file name to search for. Can use `wildcards` |
| `-path` | Specify the path to search for. Can use `wildcards` |
| `-iname` | Similar to `-name`, but ignores case |
| `-ipath` | Similar to `-path`, but ignores case |
| `-empty` | Search for empty files or directories |
| `-perm` | Specify the permissions to search for |
| `-user` | Specify the owner of the file or directory |
| `-group` | Specify the group of the file or directory |
| `-maxdepth` | Specify the maximum depth to search |
| `-mindepth` | Specify the minimum depth to search |
| `-depth` | Process each directory's contents before the directory itself |
| `-delete` | Delete the found files or directories |
| `-and` | Logic and |
| `-or` | Logic or |
| `-not` | Logic not |
| `-regex` | Use a regular expression |
| `-iregex` | Use a case-insensitive regular expression |
| `-print0` | Print the found files or directories, separated by a null character |
| `-samefile` | Search for files that are hard links to the specified file |
| `-links` | Search for files with a specific number of hard links |
| `-P` | Neever follow symbolic links (default) |
| `-L` | Follow symbolic links |
| `-H` | Follow symlinks only for the starting directories explicitly passed as arguments |
| `-mtime` | Specify the modification time of the file or directory, the time is in days |
| `-atime` | Specify the access time of the file or directory, the time is in days |
| `-ctime` | Specify the creation time of the file or directory, the time is in days |
| `-mmin` | Specify the modification time of the file or directory, the time is in minutes |
| `-amin` | Specify the access time of the file or directory, the time is in minutes |
| `-cmin` | Specify the creation time of the file or directory, the time is in minutes |

The options for `-type`:

* `b`: block device file
* `c`: character device file
* `p`: pipe file
* `s`: socket file
* `f`: regular file
* `d`: directory
* `l`: soft link file

**NOTE**: When you use `-regex`, the pattern is matched against the entire file name,
which is a little bit different from `grep`. If you want to match a specific part of the file name,
you need to use `.*` to match any characters before and after the pattern. For example,
`find . -regex ".*pattern.*"` will find files that contain `pattern` in their names.

### `ldd`

### `ln`

| Option | Description |
| --- | --- |
| `-s` | Solf link |
| `-f` | Force the creation of the link, removing existing files if necessary |
| `-t` | Specify the target directory for the link |

**NOTE**: When using `ln target link_name`, and `link_name` is an existing directory,
it will create a file named `target` in that directory.

**NOTE**: `rm` and `unlink` commands can both delete symbolic links or hard links.
But `unlink` can only delete one file at a time,

#### Hard Links and Soft Links

You can use `ln` to create hard links and soft links (symbolic links) in `Linux`.
Hard links are files that point to the same `inode` as the original file,
while soft links are files that point to the original file by its path.
Besides, hard links and soft links have the following differences:

* Soft links can point to files in different file systems,
while hard links can only point to files in the same file system.
* Soft links can point to directories, while hard links cannot point to directories.
* When the source file is deleted, hard links will still be valid,
while soft links will become invalid.

In Linux, you can use the `ls -l` command to view the number of hard links to a file.

### `readelf`

### `pgrep`

### `pkill`

### `sort`

| Option | Description |
| --- | --- |
| `-r` | Sort in reverse order |
| `-n` | Sort numerically |
| `-k` | Sort by a specific key (column) |
| `-u` | Unique sort |
| `-t` | Specify the field separator |
| `-c` | Check if the input is already sorted |
| `-f` | Ignore case when sorting |
| `-h` | Sort by human-readable numbers (e.g., 1K, 2M) |
| `-M` | Sort by month names (e.g., Jan, Feb) |
| `--files0-from=-` | Read from standard input with `NUL` as the file name separator |
| `--files0-from=filename` | Read from a file with `NUL` as the file name separator |

**NOTE**: You can use `-k` to specify multiple keys for sorting.
For example, `-k2,2 -k1,1` means sort by the second column first, then by the first column.
And you can specify parts of a column for sorting,
such as `-k2.2,2.3` to sort by the second column from the second character to the third character.
Besides, you can specify the type of the column for sorting,
such as `-k2n,2` to sort the second column numerically
and `-k2r,2` to sort the second column in reverse order.

### `tar`

| Option | Description |
| --- | --- |
| `-c` | Create a new tar archive |
| `-f` | Specify the name of the tar archive file |
| `-v` | Verbose mode |
| `-z` | Use `gzip` compression or decompression |
| `-j` | Use `bzip2` compression or decompression |
| `-J` | Use `xz` compression or decompression |
| `-x` | Extract files from a tar archive |
| `-C` | Change to a directory before performing operations |
| `-t` | List the contents of a tar archive |
| `--wildcards` | Enable wildcard matching for file names |
| `--delete` | Delete files from a tar archive (only works with uncompressed archives) |
| `--exclude` | Exclude files or directories from the tar archive |
| `-r` | Append files to a tar archive |
| `-A` | Append another tar archive to the current one |
| `-u` | Update files in a tar archive, only adding newer files |

**NOTE**: When using `--exclue`, you must use `=` to connect the option,
and `--exclude` must appear before the files or directories to be packed.
For example, `tar -cf a.tar --exclude=*.txt .` is correct,
but `tar -cf a.tar . --exclude=*.txt` is incorrect.

**NOTE**: When using `-u`, it will not overwrite old files, but will append new files directly,
after which the tar archive may contain multiple files with the same name.
But I have not found a way to extract the old files.

### `ssh`

`ssh` 分为 `client` 和 `server` 两部分，`client` 用于连接远程主机，`server` 用于接收远程主机的连接。
`openssh-client` 和 `openssh-server` 是 `ssh` 的两个主要组件。

| 选项 | 说明 |
| ---  | --- |
| `-p` | 指定端口。默认端口是 `22` |
| `-i` | 指定密钥文件。默认情况下，`ssh` 使用 `~/.ssh/id_rsa` 作为密钥文件 |
| `-l` | 指定用户名。默认情况下，`ssh` 使用当前用户名作为登录用户名 |
| `-q` | 静默模式 |
| `-t` | 强制分配终端 |
| `-v` | 显示详细信息。可以使用多个 `v` 来显示更多的信息 |
| `-C` | 压缩传输。使用 `-C` 可以压缩传输的数据 |
| `-X` | 启用 `X11` 受限转发。可以在本地显式远程主机的 `X11` 程序 |
| `-Y` | 启用 `X11` 信赖转发 |
| `-f` | 后台运行 |
| `-N` | 不执行远程命令。通常在端口转发时使用 |
| `-L` | 本地端口转发 |
| `-R` | 远程端口转发 |
| `-D` | 动态端口转发 |
| `-o` | 指定配置选项 |

**注意**：对于 `-l` 选项，也可以不使用 `-l` 而是使用 `user@host` 的形式来指定用户名。

**注意**：关于端口转发可以查看 [`ssh` 端口转发简介](/blog/2024/ssh-port-forwarding)。

**注意**：`-o` 选项常常用于覆盖 `~/.ssh/config` 文件中的配置。
例如 `ssh -o "Port=2222" host_name` 将会使用 `2222` 端口连接 `host_name`，其余配置不变。

### `ssh-keygen`

| 选项 | 说明 |
| ---  | --- |
| `-a` | 指定迭代次数。越高则生成的密钥越安全，但是验证的时候也会越慢 |
| `-t` | 指定密钥类型 |
| `-b` | 指定密钥长度 |
| `-C` | 添加注释 |
| `-c` | 修改密钥的注释 |
| `-f` | 指定密钥路经以及文件名 |
| `-P` | 指定密钥的密码 |
| `-p` | 修改密钥的密码 |
| `-l` | 显示密钥的指纹 |
| `-H` | 对 `known_hosts` 文件进行哈希处理 |
| `-R` | 从 `known_hosts` 文件中删除指定主机的密钥 |

**注意**：从 `open_ssh 9.5` 开始默认生成的密钥是 `ed25519`，而不是 `rsa`。
`ed25519` 相较与 `rsa` 更加安全，且性能更好。

### `~/.ssh/known_hosts` 文件

`known_hosts` 文件存储了远程主机的公钥，当第一次连接远程主机的时候，
`ssh` 会将远程主机的公钥存储到 `known_hosts` 文件中，下次连接远程主机的时候，
`ssh` 会检查远程主机的公钥是否和 `known_hosts` 文件中的公钥一致，
如果一致则连接成功，否则会提示公钥不一致。

`known_hosts` 文件可以很好的防止 `Man-in-the-middle` 攻击。

而默认情况下，`known_hosts` 文件的远程地址部分是以明文的形式存储的，
这样当 `known_hosts` 文件泄露的时候，攻击者可以直接获取到远程主机的地址与对应的公钥，
而使用 `ssh-keygen -H` 可以对 `known_hosts` 的远程地址进行哈希处理，
这样在 `known_hosts` 文件泄露的时候，攻击者无法直接获取到远程主机的地址与公钥的对应关系。

### `~/.ssh/config` 文件

`~/.ssh/config` 文件可以用来配置 `ssh` 的一些选项，例如：

```shell
Host host_name
    HostName host_ip
    Port port
    User user_name
    IdentityFile ~/.ssh/id_rsa

Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes
    CompressionLevel 9
    ForwardAgent yes
    ForwardX11 yes
    ForwardX11Trusted yes
    TCPKeepAlive yes
    ControlMaster auto
    ControlPath ~/.ssh/master-%r@%h:%p
    ControlPersist 600
    UserKnownHostsFile ~/.ssh/known_hosts
    StrictHostKeyChecking yes
    HashKnownHosts yes
    GSSAPIAuthentication yes
    GSSAPIDelegateCredentials yes
    GSSAPITrustDNS yes
    PasswordAuthentication no
    PubkeyAuthentication yes
    PreferredAuthentications publickey
    KexAlgorithms
    ProxyCommand ssh -W %h:%p proxy_address
```

当你拥有上面的配置后，你可以通过 `ssh host_name` 来连接远程主机，
而不需要指定远程主机的 `ip`、`port`、`user` 和 `identity file`。

使用 `*` 可以对所有主机生效，可以达到修改默认配置的目的。

`~/.ssh/config` 中也支持通配符：

* `*`：匹配所有主机
* `?`：匹配一个字符
* `!`：排除主机

### `ssh-copy-id`

| 选项 | 说明 |
| ---  | --- |
| `-i` | 指定密钥文件 |
| `-p` | 指定端口 |

### `ssh-agent`

`ssh-agent` 用来管理用户的密钥，如果密钥被添加到 `ssh-agent` 后，
用户在后续登录过程中可以不输入密钥的密码。

要在当前的 `shell` 中启动 `ssh-agent`，可以使用 `eval $(ssh-agent)`。

### `ssh-add`

| 选项 | 说明 |
| ---  | --- |
| `-l` | 列出所有已加载密钥的指纹 |
| `-L` | 列出所有已加载密钥的公钥 |
| `-d` | 删除某个密钥 |
| `-D` | 删除所有密钥 |
| `-x` | 锁定 `ssh-agent` |
| `-X` | 解锁 `ssh-agent` |

**注意**：如果命令执行过程中提示 `Could not open a connection to your authentication agent`，
那么可能是因为没有启动 `ssh-agent`，可以使用 `eval $(ssh-agent)` 启动 `ssh-agent`。

### `sftp`

`sftp` 是一个交互式的文件传输工具，其通过 `ssh` 保证文件传输的安全性。

`sftp` 的启动与 `ssh` 类似，例如 `sftp user@host`。
启动后我们可以通过 `help` 或者 `?` 列出可以使用的命令。大多 `Linux` 的命令都可以在 `sftp` 中使用。
除此之外，我们在通常的命令前面增加 `l` (或 `!`) 表示在本定执行，
例如 `pwd` 可以查看远程主机的当前目录，
而 `lpwd` 可以查看本地主机的当前目录；`cd` 可以切换远程主机的目录，
而 `lcd` 可以切换本地主机的目录。

如果你需要多次在本地执行某修操作，你可以使用 `!` 来切换到本地的 `shell`，
这样就可以不用在命令前面加 `l` 了。执行完本地操作后想要回到 `sftp` 中，只需要执行 `exit` 即可。

使用 `get` 命令可以下载文件，例如 `get file` 表示下载 `file` 文件到本地主机，
而 `put` 命令可以上传文件，例如 `put file` 表示上传 `file` 文件到远程主机。
两个命令的使用方式与 `cp` 类似，对于目录需要增加 `-r` 选项。

### `sshfs`

直接使用 `sftp` 可以完成一些简单的文件传输，但是如果我们并不只是希望直接传输远程的文件，
而且还需要对远程文件进行修改，例如我们希望直接使用我们本地配置丰富的编辑器来编辑远程文件，
那么此时我们就可以使用 `sshfs` 来实现。

`sshfs` 的原理就是利用 `sftp` 协议将远程主机的文件挂载到本地主机上，
相信使用过类似于 `NFS` 的用户对这种操作并不陌生。

使用 `sshfs` 进行挂载：

```shell
sshfs user@host:/path/to/dir /path/to/mount_point
```

**注意**：挂载点的拥有者必须是当前用户。

如果要进行卸载，可以使用 `fusermount -u /path/to/mount_point` 或者 `umount /path/to/mount_point`。

**注意**：不建议将挂载放入到 `/etc/fstab` 中，因为当网络不稳定时可能会进行很长时间的重试，
这样会导致系统启动时间非常漫长。

### `scp`

| 选项 | 说明 |
| ---  | --- |
| `-r` | 递归复制 |
| `-P` | 指定端口 |
| `-p` | 保留文件属性 (例如修改时间等) |
| `-q` | 静默模式 |
| `-v` | 显示详细信息。可以使用多个 `-v` 来显示更多的信息 |
| `-C` | 压缩传输 |
| `-i` | 指定密钥文件 |
| `-l` | 限制带宽, 单位是 `Kb/s` |
| `-3` | 通过本机在两个远端之间传输文件 |
| `-4` | 强制使用 `IPv4` |
| `-6` | 强制使用 `IPv6` |

使用 `scp` 的时候，如果在 `~/.ssh/config` 中配置了主机信息，可以直接使用主机名进行传输，
例如 `scp file host_name:/path/to/file`。

`scp` 可以一次拷贝多个文件，例如 `scp file1 file2 host_name:/path/to/`。

### `apropos`

| 选项 | 说明 |
| ---  | --- |
| `-a` | 逻辑与。可用于匹配多个关键字，例如 `apropos -a keyword1 keyword2` |
| `-e` | 精确匹配 |
| `-w` | 匹配带有 `Shell` 支持的通配符 |
| `-r` | 使用正则表达式匹配 |
| `-l` | 不依照终端宽度进行裁剪 |
| `-s` | 指定 `man` 手册的节。例如 `apropos -s 3 keyword` 表示查找第 `3` 节的手册 |

`man` 手册的节有以下几个：

* `1`：命令或程序。
* `2`：系统调用。
* `3`：库函数。
* `4`：特殊文件。
* `5`：文件格式和约定。
* `6`：游戏。
* `7`：杂项。
* `8`：系统管理命令。
* `9`：内核相关。

### `tee`

| 选项 | 说明 |
| ---  | --- |
| `-a` | 追加 |
| `-i` | 忽略中断信号 |

`tee` 的作用是将标准输入的内容输出到标准输出和文件，其通常在管道中使用：
管道在进行传递的时候可能会遇到需要 `root` 权限的时候，
这时候就需要使用 `sudo tee` 从管道中读取信息并写入到文件中。
例如 `echo "content" | sudo tee file`。

### `usermod`

| 选项 | 说明 |
| ---  | --- |
| `-l` | 修改用户名。例如 `usermod -l new_name old_name` 表示将 `old_name` 修改为 `new_name` |
| `-u` | 修改用户 `UID`。例如 `usermod -u 1000 user` 表示将 `user` 的 `UID` 修改为 `1000` |
| `-o` | 允许重复的 `UID` |
| `-g` | 修改基本用户组。例如 `usermod -g group user` 表示将 `user` 的基本用户组修改为 `group` |
| `-G` | 修改附加用户组。例如 `usermod -G group1,group2 user` 表示将 `user` 的附加用户组修改为 `group1` 和 `group2` |
| `-a` | 与 `-G` 同时使用，表示追加附加用户组 |
| `-c` | 修改用户描述 |
| `-d` | 修改用户主目录 |
| `-m` | 与 `-d` 同时使用，表示同时移动用户主目录 |
| `-s` | 修改用户登录 `shell` |
| `-e` | 修改用户过期时间。例如 `usermod -e 2025-12-31 user` 表示将 `user` 的过期时间修改为 `2025-12-31` |
| `-p` | 设置新的密码。注意新的密码应该是加密后的密码，可以使用 `openssl passwd` 来生成加密后的密码，更加推荐使用 `passwd` 命令进行密码修改 |
| `-L` | 锁定用户。 |
| `-U` | 解锁用户。 |

**注意**：创建出来的用户默认是不会过期的。如果设置了过期时间后，可以通过 `chmod -e ""` 来取消。
当用户过期后，用户将无法登录，但是用户依然存在，`root` 可以解锁用户。

### `su`

| 选项 | 说明 |
| ---  | --- |
| `-l` | 以登录状态切换用户 |
| `-c` | 执行命令。例如 `su -c command user` 表示以 `user` 用户的身份执行 `command` |
| `-s` | 指定 `shell`。例如 `su -s /bin/bash user` 表示以 `user` 用户的身份使用 `bash` `shell` |
| `-p` | 保留 `HOME`, `SHELL`, `USER`, `LOGNAME` 环境变量 |

**注意**：使用 `su` 切换用户时，如果不指定用户，会默认切换到 `root` 用户。

**注意**：如果不使用 `-l` 选项，`su` 不会切换用户的环境变量，而使用 `-l` 的时候，以下会被依次执行：

> 1. clears all the environment variables except `TERM` and variables specified by `--whitelist-environment`
> 1. initializes the environment variables `HOME`, `SHELL`, `USER`, `LOGNAME`, and `PATH`
> 1. changes to the target user’s home directory
> 1. sets `argv[0]` of the shell to `-` in order to make the shell a login shell

### `sudo`

| 选项 | 说明 |
| ---  | --- |
| `-l` | 列出用户可以执行的命令。可以使用 `sudo -l command` 来检查是否可以执行某个命令 |
| `-U` | 与 `-l` 一同使用，指定列出的用户而不是执行 `sudo` 的用户 |
| `-u` | 指定用户执行命令。例如 `sudo -u user command` 表示以 `user` 用户的身份执行 `command` |
| `-g` | 指定用户组。例如 `sudo -g group command` 表示以 `group` 用户组的身份执行 `command` |
| `-s` | 指定 `shell` |
| `-k` | 删除缓存的密码 |
| `-v` | 更新记住密码时间的时间戳为当前时刻 |

### `/etc/sudoers` 文件

`/etc/sudoers` 文件用于配置 `sudo` 的权限，只有拥有 `root` 权限的用户可以修改这个文件。

`/etc/sudoers` 文件的格式如下：

```bash
# 配置某个用户
user host=(runas[:runasgroup]) [NOPASSWD:] command
# 配置某个组下的所有用户
%group host=(runas[:runasgroup]) [NOPASSWD:] command
# 引入目录中的所有文件
@includedir dirname
```

对于上述的规则，方括号中代表可选项，这里给出几个实例：

* `user ALL=(ALL) ALL`：允许 `user` 用户在任何主机上以任何用户的身份执行任何命令。
* `%group ALL=(ALL) NOPASSWD: ALL`：允许 `group` 组下的所有用户在任何主机上以任何用户的身份执行任何命令，
且不需要输入密码。
* `@includedir /etc/sudoers.d` 表示引入 `/etc/sudoers.d` 目录下的所有文件，这是默认添加的配置，
这意味着我们对于其他用户的配置可以放在 `/etc/sudoers.d` 目录下，并以用户名命名文件方便管理。

**注意**：在 `@includedir` 目录下的文件不能以 `~` 结尾并且不能含有 `.`。
这是在 `/etc/sudoers.d/README` 中明确指出的：

> This will cause `sudo` to read and parse any files in the `/etc/sudoers.d` directory that do not
end in `~` or contain a `.` character.

**注意**：如果要配置多条命令应该使用 `,` 作为分隔符。

### `visudo`

推荐使用 `visudo` 对 `/etc/sudoers` 文件进行修改 (即通过命令 `sudo visudo`)，
`visudo` 会在修改后检查是否存在语法错误。

`visudo` 还有一些选项：

| 选项 | 说明 |
| ---  | --- |
| `-c` | 检查语法错误 |
| `-f` | 指定文件 |

### `mount`

| 选项     | 说明 |
| ---      | --- |
| `-a`     | 挂载所有在 `/etc/fstab` 中配置的文件系统 |
| `-t`     | 指定文件系统类型 |
| `-o`     | 指定挂载选项 |
| `-r`     | 以只读模式挂载 |
| `-w`     | 以读写模式挂载 |
| `--move` | 移动挂载点。例如 `mount --move /mnt1 /mnt2` 表示将 `/mnt1` 移动到 `/mnt2` 上 |
| `--fake` | 模拟挂载 |

常见的文件系统类型：

* `ext4`：`Linux` 文件系统。
* `ntfs`：`Windows` 文件系统, 使用 `mount -t ntfs-3g` 进行挂载。
* `FAT32`：`FAT32` 文件系统，使用 `mount -t vfat` 进行挂载。
* `exFAT`：需要安装 `exfat-fuse` 和 `exfat-utils` 包，使用 `mount -t exfat` 进行挂载。
* `ISO`：`ISO` 文件系统，使用 `mount -t iso9660` 进行挂载。
* `nfs`：网络文件系统，使用 `mount -t nfs -o vers=num` 可以指定版本。

**注意**：直接使用 `mount` 可以列出所有已经挂载的文件系统。
也可以使用 `mount -t type` 来列出指定类型的文件系统。

### 获取设备的文件系统类型及 `UUID`

可以使用 `blkid` 命令来获取设备的文件系统类型及 `UUID`，例如 `blkid /dev/sda1`。

### `/etc/fstab` 文件

直接通过 `mount` 命令挂载的文件系统在系统重启后会失效，为了让文件系统在系统重启后自动挂载，
我们可以将文件系统的信息写入 `/etc/fstab` 文件中。`/etc/fstab` 文件的格式如下：

```
# device <mount_point>   <type>  <options>     <dump>  <pass>
/dev/sda1       /mnt      ext4    defaults       0       2
```

其中各个字段的含义如下：

* `<device>`：设备文件。
* `<mount_point>`：挂载点。
* `<type>`：文件系统类型。
* `<options>`：挂载选项，多个选项通过 `,` 进行分隔。
* `<dump>`：备份标志。`0` 表示不备份，`1` 表示备份 (需要 `dump` 工具，通常设置为 `0`)。
* `<pass>`：文件系统检查顺序。`0` 表示不检查，`1` 表示第一个检查，
`2` 表示第二个检查 (根文件系统通常设置为 `1`，其他文件系统设置为 `2`)。

### `umount`

`umount` 用于卸载文件系统，使用方式为 `umount <mount_point>`，例如 `umount /mnt`。

**注意**：卸载文件系统的时候，如果文件系统正在被使用，会提示 `device is busy`，
这时候可以使用 `lsof <mount_point>` 来查看哪些进程在使用这个文件系统，
然后选择是否通过 `kill` 命令杀死这些进程。
也可以使用 `umount -l <mount_point>` 在空闲时自动卸载或者使用 `umount -f <mount_point>` 强制卸载。

### `lsof`

`lsof` 是 `list open files` 的缩写，用于列出系统中打开的文件。`lsof` 的输出包含如下几项：

* `COMMAND`：打开文件所使用的命令。
* `PID`：进程 `ID`。
* `USER`：进程的用户。
* `FD`：文件描述符。
* `TYPE`：文件类型，常见的有 `REG`、`DIR`、`CHR`、`FIFO`、`SOCK`、`LINK`，分别代表普通文件、目录、
字符设备、管道、套接字、符号链接。
* `DEVICE`：设备。
* `SIZE/OFF`：文件大小或者偏移量。
* `NODE`：`inode` 号。
* `NAME`：打开的文件名。

`lsof` 的可用选项如下：

| 选项 | 说明 |
| ---  | --- |
| `-u` | 指定用户 |
| `-c` | 指定命令开头 |
| `-b` | 避免获取结果时调用可能会阻塞的内核函数 |
| `+D` | 指定目录 |
| `-i` | 查看网络连接 |
| `-n` | 禁止显示域名，域名全部显示为 `IP` 地址 |
| `-P` | 禁止将端口号转换为服务名 |
| `-p` | 指定进程 `ID` |
| `-U` | 列出 `UNIX` 域套接字。`TYPE` 为 `unix` 的文件 |
| `-R` | 同时列出父进程 `ID` |
| `-l` | 显示用户的 `ID` |
| `-t` | 只输出打开文件的进程 `ID` |
| `-a` | 逻辑与。例如 `lsof -u user -a -c command` 表示查找用户为 `user` 且命令开头为 `command` 的进程 |
| `-d` | 指定文件描述符。例如 `lsof -d 1` 表示查找文件描述符为 `1` 的文件 |

**注意**：使用某些选项时可以使用 `^` 来表示排除某个用户，例如 `lsof -u ^root` 表示排除 `root` 用户。

**注意**：`-i` 的完整格式为：

```shell
-i[46][protocol][@hostname|@hostaddr][:service|:port]，
```

其中 `protocol` 可以是 `TCP`、`UDP`、`TCP:UDP`，`hostname` 可以是主机名、`IPv4` 地址、`IPv6` 地址，
`service` 可以是服务名、端口号。

## Makefile

### Variables

You can define variables in `Makefile` and use the variables through `$(variable)`:

```Makefile
objects = obj1.o obj2.o obj3.o
all: $(object)
	$(CC) $(object) -o main
```

If you want to express the literal `$`, you can use `$$`.

If you want to define a variable whose value is a single space, you can use:

```Makefile
nullstring :=
space := $(nullstring) # end of line
```

The `$nullstring` is a empty string, and `space` is a single space.

**NOTE**: Note that the `#` and a single space before `#` are necessary.
This is because that the trailing spaces will be added to variables.

You can nest `$` to get value:

```Makefile
x = y
y = z
z = u
# $(x) is y
# $($(x)) is $(y), and $(y) is z
# $($($(x))) is $(z), and $(z) is u
a := $($($(x)))
```

### Target Variables

You can set variables only valid for the specified target:

```Makefile
prog: CFLAGS = -g
prog: prog.o foo.o bar.o
	$(CC) $(CFLAGS) prog.o foo.o bar.o
prog.o: prog.c
	$(CC) $(CFLAGS) prog.c
foo.o: foo.c
	$(CC) $(CFLAGS) foo.c
bar.o: bar.c
	$(CC) $(CFLAGS) bar.c
```

In the example above, only the `prog`'s `CFLAGS` is `-g`.

### `=` `:=` `+=` and `?=`

There are many equal signs in `make`, but they are very different with each other:

* `=`: This is like references in `C/C++`.
For example, if you use `a = $(b)`, once `b`'s value changed after, the `a` will change too.
* `:=`: This is assignment equal sign, which is similar with `=` in `C/C++`.
* `+=`: This is to append a variable with new values.
* `?=`: This will check if the variable is assigned before;
if it is, this will not work, otherwise it is similar with `=`.

For the `+=`:

* If the variable has not been defined, it will be `=`.
* If the variable has been define, it will follow the last equal sign.
If the last equal is `=`, `+=` will use `=`; if the last equal sign is `:=`, `+=` will use `:=`.

### override

When the variable is defined by the `make` command,
for example `make a=12` will define a variable called `a` whose value is `12`,
the variable defined and assigned in your `Makefile` will be replaced by the command line's.
If you don't want the variable be replaced, you can use `override`:

```Makefile
# you can use other equal signs
override a := 0
```

### Multi Line Variables

You can define multi line variables by `define`,
the signature after `define` is the name of the variable.
Note that commands in macro must start with tab, so if the lines are not started with tab,
they will be treated as a multi line variable's value. There is an example below:

```Makefile
define two-lines
echo foo
echo $(bar)
endef
```

**NOTE**: `$(bar)` will be replaced by the value of `bar`.

### `$@` `$<` `$*` `$%` `$?` `$^` and `$+`

* `$@`: A variable in `make`, whose value is the target.
For example, if `$@` appears in commands following `main.o: main.cpp`,
`$@` will be `main.o` exactly.
* `$<`: A variable in `make`, whose value is the first dependency.
For example, if `$<` appears in commands following `main.o: main.cpp`,
`$<` will be `main.cpp` exactly.
* `$*`: A variable in `make`, whose value is the stem of the target.
For example, using `$*` following `pre_%.o: pre_%.c`, and the target is `pre_foo.o`,
`$*` will be `foo`.
* `$%`: When the target is in an archive (like `foo.a`),
this variable is the names of members.
For example, if a target is `foo.a(bar.o)` the `$%` will be `bar.o` and the `$@` will be `foo.a`.
* `$?`: A variable in `make`, whose value is all the dependencies that are newer than the target.
* `$^`: A variable in `make`, whose value is all the dependencies of the target.
This will have only one copy if the target depends on the same file more than once.
* `$+`: A variable similar with `$^` but will store the repetitive files.

### Auto Deduction

You can use `Makefile` with auto deduction. Auto deduction looks like this:

```Makefile
%.o: %.c
```

The example above will add `main.c` or `main.cpp` for target `main.o`,
and the command `$(CC) $(CFLAGS) -c $< -o $@`
will be added automatically too.

### Implicit Rules

Implicit rules are similar with auto deduction.
Or we can say auto deduction depends on implicit rules.

There are different implicit rules for different files.
I'll give the implicit rules of `C` and `C++` files:

* `C`: `*.o` will be deducted depending on `*.c`
and the build command will be `$(CC) -c $(CPPFLAGS) $(CFLAGS)`.
* `C++`: `*.o` will be deducted depending on`*.C, *.cc or *.cpp`
and the build command will be `$(CC) -c $(CPPFLAGS) $(CFLAGS)`.

### PHONY

`.PHONY` is to specify the target is a pseudo target.
This is usually used for `clean` and `all`.
`.PHONY` will let `make` not treat the target as a file:

```Makefile
.PHONY: clean
```

### include

`include` is very similar with the `include` in `C/C++`.
The command will read the files' contents and put them where the `include` command is:

```Makefile
# this will read the contents of config.make and put it there.
include config.make
```

`make` can also use the `-I` to specify where to find the files used by `include` command.
For example `make -I./include` will let `include` command find the `./include` directory.

### `VPATH` and `vpath`

`VPATH` is a variable in `make`,
which is used to specify the directories where `make` will look for files when they are not found in the current directory.
The value of `VPATH` is a colon-separated list of directories.

```Makefile
# this will specify two directories to be used to find files
VPATH = src:../include
```

`vpath` is a keyword of `make`, and this keyword also can be used to find files:

```Makefile
# % is similar with .* of regexpr
# this is to specify to find header files in ../include
vpath %.h ../include

# this is to specify to find C files in ./src
vpath %.c ./src
```

### Multi Targets

You can write more than one target in one line:

```Makefile
bigoutput littleoutput : text.g
	-generate text.g $(subst output,,$@) > $@
```

`$(subst output,,$@)` means substitute `output` of `$@` with empty string.
The hyphen before means to ignore errors.
`make` will check the return value after each command.
When return value is non-zero, `make` will stop, the hyphen means don't check the return value.

More straightforward, the commands above are same as:

```Makefile
bigoutput : text.g
	-generate text.g big > bigoutput
littleoutput : text.g
	-generate text.g little > littleoutput
```

### Static Pattern Rules

In `make`, you can use static pattern rules to simplify the `Makefile`:

```Makefile
objects = foo.o bar.o
all: $(objects)
$(objects): %.o: %.c
	$(CC) -c $(CFLAGS) $< -o $@
```

`$(objects): %.o: %.c` will find `foo.o` and `bar.o` **from `$(objects)`** and
place them at `%.o`, and then find `foo.c` and `bar.c` **from `foo.o bar.o`**.

There is another example to use `filter` and static pattern rules:

```Makefile
files = foo.elc bar.o lose.o
# $(filter %.o,%(files)) will get all the .o files from $(files)
$(filter %.o,$(files)): %.o: %.c
	$(CC) -c $(CFLAGS) $< -o $@
$(filter %.elc,$(files)): %.elc: %.el
	emacs -f batch-byte-compile $<
```

### Generate Dependencies Automatically

`gcc -MM *.c` will print the header files of `C` files.
For example `gcc -MM test.c`'s output will look like this:

```
test.o: test.c header1 header2 header3
```

We can use the command to automatically add dependencies for a target:

```Makefile
# Add @ before one command will not print the command
# set -e will let the command stop when error occurs
%.d: %.c
	@set -e; \
	$(CC) -MM $< > $@.; \
	sed -e 's/\($*\)\.o[ : ]*/\1.o $@ :/g' < $@. > $@; \
	rm -f $@.
```

We use `$(CC) -MM $< > $@.` to generate a file `%.d.` to store the output.
Then we use `sed` to substitute the `%.o` with `%.o %.d` and output to `%.d`.
Finally we remove the `%.d.`.

After this, we can include the `%.d` files:

```Makefile
sources = a.c b.c
# substitude .c with .d in sources
include $(sources:.c=.d)
```

After `include`,
we'll have something like `a.o a.d: a.c header` in our `Makefile`,
then `make` will auto deduction the commands.

### Nested Makefiles

If you use 3rd parties, you may want to build the 3rd parties by `make`:

```Makefile
subsystem:
	$(MAKE) -C subdir
```

You can pass the variables of current `Makefile` to `sub-Makefile` through `export`:

```Makefile
export variable = value
# pass all variables
export
```

### define

In `make`, you can use `define` to define macros:

```Makefile
# note that there is no semicolon after each command
define name
	command1
	command2
	...
endef
```

If you want to use the macro, you just need use `$(macro_name)` to call the macro.

### Conditional Structures

This part is simple, so I just post some examples.

The example using `ifeq`:

```Makefile
libs_for_gcc = -lgnu
normal_libs =
foo: $(objects)
ifeq ($(CC),gcc)
	$(CC) -o foo $(objects) $(libs_for_gcc)
else
	$(CC) -o foo $(objects) $(normal_libs)
endif
```

You can use `ifneq`, `ifdef` and `ifndef`, too.

### Functions

You can use functions in `make` through `$(func_name args)`.

There are some functions related to strings:

* `$(subst from,to,text)`: substitute `from` with `to` in `text`.
* `$(patsubst pattern,replacement,text)`: substitute `pattern` with `replacement` in `text`.
* `$(strip text)`: remove all the leading and trailing spaces in `text`.
* `$(findstring target,text)`: find `target` in text, if found, return `target` otherwise, return empty string.
* `$(filter pattern...,text)`: filter the contents matching the `pattern...` from `text`.
* `$(filter-out pattern...,text)`: filter out the contents matching the `pattern...` from `text`.
* `$(sort list)`: sort the contents in `list` lexicographically. Note that `sort` will unique the contents, too.
* `$(word i,text)`: get the `i`-th word from `text` (index started from `1`).
* `$(wordlist l,r,text)`: get the words whose index is in `[l,r]` in sequence.
* `$(firstword text)`: get the first word of `text`.

There are some functions related to files:

* `$(dir name...)`: get the directory part from `name`.
* `$(notdir name...)`: get the non-directory part from `name`.
* `$(suffix name...)`: get the suffixes of files from `name`.
* `$(basename name...)`: get the base name (files' name without extension) of files from `name`.
* `$(addsuffix suffix,name)`: add `suffix` for `name`.
* `$(addprefix prefix,name)`: add `prefix` for `name`.
* `$(join list1,list2)`: join two lists. This will append words in `list2` to `list1`. For example `$(join aaa bbb,111 222 333)` will get `aaa111 bbb222 333`.

---

If I want add a suffix for every item in a variable, I can do it with this:

```Makefile
names := a b c d
files := $(foreach item,$(names),$(item).o)
```

---

* `$(if condition,then-part,else-part)`: if the `condition` is a non-empty string,
it will return the `then-part`, otherwise it will return the `else-part`.

---

Now if you hope to have a function which can reverse two parameters, you can do it through this:

```Makefile
reverse = $(2) $(1)
reversedItemList = $(call reverse,item1,item2)
```

After that, `reversedItemList` will be `item2 item1`.
Of course, you can add different suffixes for items:

```Makefile
addTargetAndDependency = $(1).o : $(1).c
result = $(call addTargetAndDependency,main)
```

`result` will be `main.o : main.c`.

---

`$(origin variablename)` will tell you where the `variablename` comes from.
The return values are explained below:

* `undefined`: never defined before.
* `environment`: the variable comes from environment variables.
* `default`: the default variable, such `CC`.
* `file`: the variable is defined in a `Makefile`.
* `command line`: the variable is defined by command lines
(when you type `make a=1`, `a` is defined by command lines).
* `override`: the variable if defined by `override`.
* `automatic`: the variable is defined by `make` automatically.

---

This is to execute a command in shell:

```Makefile
contents := $(shell cat foo)
files := $(shell echo *.c)
```

The commands above will get the output of a shell command.

---

* `$(erro text)`: output `text` and stop `make`.
* `$(warning text)`: output `text` but don't stop `make`.

### Return Value of Make

* `0`: success.
* `1`: some errors.
* `2`: when you use `-q` (`-q` will not run `make`,
but give you a return value `0` if the targets are up to date)
and `make` cannot make sure whether or not the files is up to date.

### Specify Target

If you run `make`, `make` will build the first target.
But you can specify target by `make targetname`.

There are some rules you should obey for naming a target when you write `Makefile`:

* `all`: build all targets.
* `clean`: remove all the files created by `make`.
* `install`: install the targets, when in `C/C++`,
this will move the binary files to `/usr/bin` and move the header files to `/usr/include`.
* `print`: print the files having been updated.
* `tar`: pack the source files into a tar file.
* `dist`: create a compressed file including all source files.

### Check Make Syntax

When you want to check your syntax in `Makefile` rather than run the `make`,
you can use those options: `-n, --just-print, --dry-run, --recon`. The four options are synonyms.

### Other Options in make

| Option | Description |
| --- | --- |
| `-j` | Use the specified number of jobs (cores) to build |
| `-q` | Check if the target exists |
| `-W` | Build the targets that depend on the specified file |
| `-o` | Ignore the specified file while building |
| `-B` | Always re-build all targets, even if they are up to date |
| `-C` | Change working directory to the specified directory |
| `-t` | Touch all the targets |
| `--debug` | Print debug info |
| `-e` | Environment overrides |
| `-f` | Use the specified file as `Makefile` |
| `-i` | Ignore errors |
| `-I` | Include directory |
| `-k` | Keep going even if there are errors |
| `-S` | Stop when an error occurs |
| `-p` | Print the data base |
| `-r` | Do not use built-in rules |
| `-R` | Do not use built-in variables |
| `-s` | Silent mode, do not print commands |
| `-w` | Print the working directory before and after processing |
| `--no-print-directory` | Do not print the working directory |
| `--warn-undefined-variables` | Warn when a variable is undefined |

`--debug=options` will print the debug info of `make`, the available options are:

* `a`: print all info.
* `b`: print basic info.
* `v`: verbose.
* `i`: print implicit rules.
* `j`: print jobs' info including `PID`, return value and so on.
* `m`: this is for debugging when remaking makefiles.
* If you just use `-d`, this is same with `--debug=a`.

You have learned a lot about `make`, why not to read the
[Makefile](https://github.com/torvalds/linux/blob/master/Makefile)
of
[Linux Kernel](https://github.com/torvalds/linux).

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
* [Man Page of Bash](https://www.gnu.org/software/bash/manual/bash.html)
* [How to Use the awk Command on Linux](https://www.howtogeek.com/562941/how-to-use-the-awk-command-on-linux/)
* [30+ awk examples for beginners / awk command tutorial in Linux/Unix](https://www.golinuxcloud.com/awk-examples-with-command-tutorial-unix-linux/)
* [8 Powerful Awk Built-in Variables – FS, OFS, RS, ORS, NR, NF, FILENAME, FNR](https://www.thegeekstuff.com/2010/01/8-powerful-awk-built-in-variables-fs-ofs-rs-ors-nr-nf-filename-fnr/)
* [Getting Started With AWK Command](https://linuxhandbook.com/awk-command-tutorial/)
* [cat command examples for beginners](https://www.golinuxcloud.com/cat-command-examples/)
* [25+ most used find commands in Linux](https://www.golinuxcloud.com/find-command-in-linux/)
* [find Linux Command Cheatsheet](https://onecompiler.com/cheatsheets/find)
* [Linux Find Cheatsheet](https://linuxtutorials.org/linux-find-cheatsheet/)
* [Find files and directories on Linux with the find command](https://opensource.com/article/21/9/linux-find-command)
* [10 ways to use the Linux find command](https://www.redhat.com/sysadmin/linux-find-command)
* [Find cheatsheet](https://quickref.me/find)
* [Linux Tutorial - Cheat Sheet - grep](https://ryanstutorials.net/linuxtutorial/cheatsheetgrep.php)
* [20 grep command examples in Linux](https://www.golinuxcloud.com/grep-command-in-linux/)
* [Sed Command Cheat Sheet: 30 Essential One-Liners for Text Processing](https://karandeepsingh.ca/posts/sed-command-cheat-sheet-30-essential-one-liners/)
* [sed cheatsheet](https://devhints.io/sed)
* [Linux Handbook: sort Command Examples](https://linuxhandbook.com/sort-command/)
* [15+ Tips to PROPERLY sort files in Linux](https://www.golinuxcloud.com/linux-sort-files/#1_Sort_by_name)
* [Linux and Unix sort command tutorial with examples](https://shapeshed.com/unix-sort/)
* [Linux sort Command](https://www.baeldung.com/linux/sort-command)
* [Linux Audit: tar cheat sheet](https://linux-audit.com/cheat-sheets/tar/)
* [15+ tar command examples in Linux](https://www.golinuxcloud.com/tar-command-in-linux/)
* [How to write makefiles](https://github.com/seisman/how-to-write-makefile).
* [Ignoring files](https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files)
* [10+ practical examples to create symbolic link in Linux](https://www.golinuxcloud.com/create-symbolic-link-linux/)
* [15+ scp command examples in Linux](https://www.golinuxcloud.com/scp-command-in-linux/)
* [apropos Linux Command Explained](https://phoenixnap.com/kb/apropos-linux)
* [10 tee command examples in Linux](https://www.golinuxcloud.com/tee-command-in-linux/)
* [How To Use ‘sudo’: The Complete Linux Command Guide](https://raspberrytips.com/sudo-linux-command/)
* [Linux visudo command](https://www.computerhope.com/unix/visudo.htm)
* [15 usermod command examples in Linux](https://www.golinuxcloud.com/usermod-command-in-linux/)
* [Linux mount command to access filesystems, iso image, usb, network drives](https://www.golinuxcloud.com/linux-mount-command-iso-usb-network-drive/)
* [The “mount” Command in Linux](https://linuxsimply.com/mount-command-in-linux/)
* [9 su command examples in Linux](https://www.golinuxcloud.com/su-command-in-linux/)
* [15+ SSH command examples in Linux](https://www.golinuxcloud.com/ssh-command-in-linux/)
* [15+ lsof command examples in Linux](https://www.golinuxcloud.com/lsof-command-in-linux/)
* [OpenSSH 9.5: A User-Friendly Guide to the Latest Update](https://stilia-johny.medium.com/openssh-9-5-a-user-friendly-guide-to-the-latest-update-840a09886a5a)
* [How to generate and manage ssh keys on Linux](https://linuxconfig.org/how-to-generate-and-manage-ssh-keys-on-linux)
* [10 examples to generate SSH key in Linux (ssh-keygen)](https://www.golinuxcloud.com/generate-ssh-key-linux/)
* [The Complete Guide to SSH Config Files](https://thelinuxcode.com/ssh-config-file/)
* [Beginners guide to use ssh config file with examples](https://www.golinuxcloud.com/ssh-config/)
* [How to use the command `ssh-add` (with examples)](https://commandmasters.com/commands/ssh-add-common/)
* [5 Unix / Linux ssh-add Command Examples to Add SSH Key to Agent](https://linux.101hacks.com/unix/ssh-add/)
* [How To Use SFTP to Securely Transfer Files with a Remote Server](https://www.digitalocean.com/community/tutorials/how-to-use-sftp-to-securely-transfer-files-with-a-remote-server)
* [libfuse/sshfs](https://github.com/libfuse/sshfs)
