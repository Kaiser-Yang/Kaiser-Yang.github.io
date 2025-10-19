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

- `Ctrl + A`: Move to the beginning of the line.
- `Ctrl + E`: Move to the end of the line.
- `Ctrl + U`: Delete from the cursor to the beginning of the line.
- `Ctrl + K`: Delete from the cursor to the end of the line.
- `Ctrl + W`: Delete the word before the cursor.
- `Ctrl + L`: Clear the screen.
- `Ctrl + P`: Previous command in history (same as the up arrow key).
- `Ctrl + N`: Next command in history (same as the down arrow key).
- `Ctrl + Z`: Send SIGTSTP to the foreground process (suspend).
- `Ctrl + C`: Send SIGINT to the foreground process (terminate).
- `Ctrl + \`: Send SIGQUIT to the foreground process (terminate and create core dump).
- `Ctrl + D`: Send EOF to the foreground process (end of file).

### Wildcards

Wildcards are symbols that can be used to match multiple files or directories.
Those symbols are supported by the shell, and they can be used in any command
as long as the shell supports them.

Different shells may have different wildcard symbols,
there are some wildcards in the `bash` shell:

- `*`: matches any number (including `0`) of characters.
- `**`: matches all files and directories recursively.
- `**/`: matches all directories recursively.
- `?`: matches a single character.
- `[]`：matches any character in the brackets. For example, `[abc]` matches `a` or `b` or `c`.
- `[!]` or `[^]`: matches any character not in the brackets. For example, `[!abc]` matches any
  character except `a`, `b` and `c`.

**NOTE**: `**` and `**/` are only available when `globstar` shell option is enabled
(use `shopt | grep globstar` to check). You can use `shopt -s globstar` to enable it.

**NOTE**: For `*`, hidden files/directories (e.g., .bashrc, .config) are **not matched** by
unless the `dotglob` option is enabled (via `shopt -s dotglob`).

In the square brackets, you can use `-` to represent a range, for example, `[0-9]`
matches any digit, and `[a-z]` matches any lowercase letter.

There is a symbol `{}` which is not a wildcard but is often used in the same way.
It is used to represent multiple strings, for example, `a{b,c}d` will be parsed as `abd` or `acd`.
For example, `mv file_{old,new}` will be parsed as `mv file_old file_new`. You can use
nested curly brackets, for example, `echo a{b{c,d},e}f` will be parsed as `echo abcf abdf aef`,
which works like distributive law in mathematics. `..` is used to represent a range,
for example, `echo {a..c}` will be parsed as `echo a b c`.

**Note**: These wildcards are not all same with regex. In regex, `*` means 0 or more times,
`?` means 0 or 1 time, and `^` also means the beginning of a line.

When `extglob` is on (use `shopt | grep extglob` to check), those below are supported:

- `?(pattern-list)`: Matches zero or one occurrence of the given patterns.
- `*(pattern-list)`: Matches zero or more occurrences of the given patterns.
- `+(pattern-list)`: Matches one or more occurrences of the given patterns.
- `@(pattern-list)`: Matches one of the given patterns.
- `!(pattern-list)`: Matches anything except one of the given patterns.

#### `POSIX` Character Classes

- `[:alnum:]`: Letter or digit.
- `[:alpha:]`: Letter.
- `[:blank:]`: Space or tab.
- `[:cntrl:]`: Control character.
- `[:digit:]`: Digit.
- `[:xdigit:]`: Hexadecimal digit (0-9, a-f, A-F).
- `[:graph:]`: Printable character except space.
- `[:lower:]`: Lowercase letter.
- `[:upper:]`: Uppercase letter.
- `[:print:]`: Printable character including space.
- `[:punct:]`: Punctuation character.
- `[:space:]`: Space, tab, newline, carriage return, form feed, or vertical tab.

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

**NOTE**: When the shell is in POSIX mode, the `!` has no special meaning within double quotes,
even when history expansion is enabled.

`$` and `` ` `` retain their special meaning within double quotes. For example:

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

| Option | Meaning                            |
| ------ | ---------------------------------- |
| `-a`   | indexed array                      |
| `-A`   | associative array                  |
| `-i`   | integer                            |
| `-l`   | lowercase value automatically      |
| `-u`   | uppercase value automatically      |
| `-r`   | readonly                           |
| `-x`   | export                             |
| `-n`   | name reference to another variable |
| `-t`   | trace, rarely used                 |

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
args[1]="mongo" # Change the second element (2 --> mongo)
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

**NOTE**: `0` is not a positional parameter, and it is a special parameter,
which will be expanded to the name of the shell or shell script.

**NOTE**: In `bash`, the array is 0-indexed, but in `zsh`, the array is 1-indexed.

### Special Parameters

- `$*`: All positional parameters, each of which expands to a separate word.
- `"$*"`: A single string with all positional parameters
  separated by the first character of the `IFS` variable.
  If `IFS` is unset, the parameters are separated by spaces.
  If `IFS` is null, the parameters are joined without intervening separators.
- `$@`: All positional parameters, each of which expands to a separate word.
- `"$@"`: Equivalent to `"$1" "$2" ... "$N"` (where N is the number of positional parameters).
  `prefix"$@"suffix` will be parsed as `prefix"$1" "$2" ... "$N"suffix`.
- `$#`: The number of positional parameters in decimal.
- `$?`: The exit status of the most recently executed foreground pipeline.
- `$$`: The process ID of the shell. In a sub-shell,
  it expands to the process ID of the current shell, not the sub-shell.
- `$!`: The process ID of the job most recently placed into the background.
- `$0`: The name of the shell or shell script.
  If bash is invoked with a file of commands, `$0` is set to the name of that file.
  If bash is started with the `-c` option,
  then `$0` is set to the first argument after the string to be executed, if one is present.
  Otherwise, it is set to the filename used to invoke bash, as given by argument zero.
- `$_`: The last argument of the previous command or script path.
- `$-`: The current option flags as specified upon invocation,
  by the set builtin command, or those set by the shell itself (such as the -i option).

The characters of `$-` and their meanings:

| Flag | Meaning                                                            |
| ---- | ------------------------------------------------------------------ |
| `h`  | hashall (remembers command locations in `$PATH`)                   |
| `i`  | interactive shell                                                  |
| `m`  | monitor mode (job control enabled)                                 |
| `H`  | history expansion enabled (e.g., `!!` expands to the last command) |
| `B`  | brace expansion enabled (e.g., `{a, b}` expands to `a b`)          |
| `s`  | compounds read from stdin (e.g., `bash -s`                         |

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

- brace expansion;
- tilde expansion, parameter and variable expansion,
  arithmetic expansion, command substitution (done in a left-to-right fashion)
  and process substitution (if the system supports it);
- word splitting;
- pathname expansion.

After these expansions are performed,
quote characters present in the original word are removed
unless they have been quoted themselves (quote removal).

#### Brace Expansion

This is the expansion of `{}` see [Wildcards](#wildcards).

#### Tilde Expansion

- `~`: Current user's home (`$HOME`).
- `~username`: Home directory of `username`.
- `~+`: Current working directory (`$PWD`).
- `~-`: Previous working directory (`$OLDPWD`).
- `"~"` or `'~'`: Literal `~` (no expansion).
- `~+number`: The `number`-th entry of the output `dirs` (0-indexed).
- `~-number`: The `number`-last entry of the output `dirs` (1-indexed).
- `~number`: Same as `~+number`.

**NOTE:** You can use `pushd` to add directory to `dirs` and `popd` to remove directory from `dirs`.

#### Parameter and Variable Expansion

- `${parameter}`: The value of parameter is substituted. Sometimes, the braces can be omitted.
- `${!parameter}`: Expands to the value of the variable named by `parameter`.
  For example:

```bash
foo='Hello'
bar='foo'
echo "${!bar}" # Hello
```

- `${!nameref}`: Expands to the referenced name, for example:

```bash
declare -n nameref_var="target_var"  # nameref_var is a reference to target_var
target_var="Hello"

echo "$nameref_var" # Hello (dereferences automatically)
echo "${!nameref_var}" # target_var (returns the referenced name)
```

- `${!prefix*}` or `${prefix@}`: Expands to the values of variables whose names begin with `prefix`.
  For example:

```bash
a=1
aa=11
aaa=111
echo "${!a*}" # 1 11 111 (all variables starting with a)
echo "${!a@}" # 1 11 111 (all variables starting with a)
```

- `${parameter:offset}`: Expands to the substring of the value of `parameter`
  starting at the character specified by `offset`.
- `${parameter:offset:length}`: Expands to the substring of the value of `parameter`
  starting at the character specified by `offset` and extending for `length` characters.
  For examples:

```bash
# Basic usage
str="Hello, World\!"
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

**NOTE**: In the splitting operation, the offset for both `zsh` and `bash` is 0-indexed.

- `${!array[@]}` or `${!array[*]}`: Expands to the indices of the array `array`.
- `${#parameter}`: The length of the value of `parameter` is substituted.
- `${#*}` or `${#@}`: Same with `$#`: the number of positional parameters.

For those below, you can remove `:` to make it only work for unset variables:

- `${parameter:-word}`: Expands to `word` if `parameter` is unset or null;
  otherwise, it expands to the value of `parameter`.
- `${parameter:=word}`: Assigns `word` to `parameter` and expands to `word`
  if `parameter` is unset or null;
  otherwise, it expands to the value of `parameter`.
  You can not use this to positional parameters.
- `${parameter:?word}`: `word` is written to standard error if `parameter` is unset or null,
  if it is not interactive, exits;
  otherwise, it expands to the value of `parameter`.
- `${parameter:+word}`: Nothing is substituted if `parameter` is unset or null;
  otherwise, the expansion of `word` is substituted.

For those below,
if `parameter` is `@` or `*` or an array subscripted with `@` or `*`,
the pattern removal operation is applied to each element in turn,
and the expansion is the resultant list:

- `${parameter#word}`: Removes the shortest match of `word` from the beginning of `parameter`.
  Wildcards are allowed in `word`. See [Wildcards](#wildcards).
- `${parameter##word}`: Removes the longest match of `word` from the beginning of `parameter`.
  Wildcards are allowed in `word`. See [Wildcards](#wildcards).
- `${parameter%word}`: Similar to `${parameter#word}`
  but removes the suffix instead of the prefix.
- `${parameter%%word}`: Similar to `${parameter##word}`
  but removes the suffix instead of the prefix.
- `${parameter@U}`: Converts the value of `parameter` to uppercase.
- `${parameter@u}`: Converts the first character of the value of `parameter` to uppercase.
- `${parameter@L}`: Converts the value of `parameter` to lowercase.
- `${parameter@a}`: Expands to the attributes of `parameter`.
- `${parameter@E}`: Expands to a string with all the escaped characters expanded
  (such as `\n` -> newline).
- `${parameter@A}`: Expands to a string whose value,
  if evaluated,
  will recreate `parameter` with its attributes and value.
  If used for array variables, you should use `${a[@]@A}` to get the string.
- `${parameter@Q}`: Expands to a single-Quoted string with any special characters
  (such as `\n`, `\t`, etc.) escaped.
  For examples:

```bash
a='Hello World'
b=('Hello' 'World')
declare -A c=([first]='Hello' [second]='World')
echo "${a@Q}" # 'Hello World'
echo "${b[@]@Q}" # 'Hello' 'World'
echo "${c[@]@Q}" # 'World' 'Hello' (unordered)
```

- `${parameter@K}`: Similar to `${parameter@Q}`, but this will
  print the values of indexed and associative arrays as a sequence of quoted key-value pairs.
  For examples:

```bash
a='Hello World'
b=('Hello' 'World')
declare -A c=([first]='Hello' [second]='World')
echo "${a@K}" # 'Hello World'
echo "${b[@]@K}" # 0 "Hello" 1 "World"
echo "${c[@]@K}" # first "Hello" second "World"
```

- `${parameter@P}`: Expands as if it were a prompt string.
  For examples:

```bash
PS1='\u@\h:\w\$ '
echo "${PS1@P}" # user@host:/path$  (expands prompt codes)
```

- `${parameter/pattern/string}`: Replace the longest match of `pattern` with `string`.
  If `pattern` begins with `/`, all matches of `pattern` are replaced with `string`.
  If `pattern` begins with `#`, it must match at the beginning of the expanded value of `parameter`.
  If `pattern` begins with `%`, it must match at the end of the expanded value of `parameter`.
  If `string` is null,
  matches of `pattern` are deleted and the `/` following `pattern` may be omitted.
  If the `nocasematch` shell option is enabled,
  the match is performed without regard to the case of alphabetic characters.
- `${paramter^pattern}`: Convert the first match of `pattern` to uppercase.
  The `pattern` can only match one character.
  If `pattern` is omitted, it is treated like a `?`, which matches every character.
  Wildcards are allowed in `pattern`.
  See [Wildcards](#wildcards).
- `${paramter^^pattern}`: Convert all matches of `pattern` to uppercase.
  The `pattern` can only match one character.
  If `pattern` is omitted, it is treated like a `?`, which matches every character.
  Wildcards are allowed in `pattern`.
  See [Wildcards](#wildcards).
- `${paramter,pattern}`: Convert the first match of `pattern` to lowercase.
  The `pattern` can only match one character.
  If `pattern` is omitted, it is treated like a `?`, which matches every character.
  Wildcards are allowed in `pattern`.
  See [Wildcards](#wildcards).
- `${paramter,,pattern}`: Convert all matches of `pattern` to lowercase.
  The `pattern` can only match one character.
  If `pattern` is omitted, it is treated like a `?`, which matches every character.
  Wildcards are allowed in `pattern`.
  See [Wildcards](#wildcards).

#### Arithmetic Expansion

- `$$(expression)`: The value of `expression` is substituted.

#### Command Substitution

- `$(command)` or `` `command` ``: The standard output of `command` is substituted
  with trailing newlines deleted.

#### Process Substitution

- `<(command)`: Provides the output of `command` as a file that can be read from.
- `>(command)`: Provides a file that, when written to, becomes the input for `command`

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

### `[[ expression ]]` and `[ expression ]`

For the code below:

```bash
if [ $(grep nothing /dev/null) = '' ]; then
  echo 'This will not work'
fi

if [[ $(grep nothing /dev/null) == '' ]]; then
  echo 'Empty'
fi
```

The first `if` statement will cause an error because
`$(grep nothing /dev/null)` returns nothing, and the command becomes
`[ = '' ]`, which is invalid. The second `if` statement works because
`[[ ... ]]` does not perform word splitting or pathname expansion.
Therefore, it is recommended to use `[[ ... ]]` instead of `[ ... ]`
when possible.

## Environment Variables

### Locality

| Variable            | Description                                  |
| ------------------- | -------------------------------------------- |
| `LC_ALL`            | Overrides all locale settings                |
| `LC_CTYPE`          | Character classification and case conversion |
| `LC_COLLATE`        | String collation order                       |
| `LC_MESSAGES`       | Language for system messages                 |
| `LC_TIME`           | Date and time formatting                     |
| `LC_NUMERIC`        | Number formatting                            |
| `LC_MONETARY`       | Currency formatting                          |
| `LC_PAPER`          | Paper size and format                        |
| `LC_NAME`           | Name formatting                              |
| `LC_ADDRESS`        | Address formatting                           |
| `LC_TELEPHONE`      | Telephone number formatting                  |
| `LC_MEASUREMENT`    | Measurement units                            |
| `LC_IDENTIFICATION` | Locale identification                        |
| `LANG`              | Default locale setting                       |

The priority of locale settings:

```bash
LC_ALL > LC_* (specific category) > LANG
```

**NOTE**: There is another variable called `LANGUAGE`
which is used to specify the language priority list for messages.

## Commands

### `apropos`

| Option | Description                                 |
| ------ | ------------------------------------------- |
| `-a`   | Logical AND of search terms (default is OR) |
| `-e`   | Exact match only                            |
| `-w`   | Wildcard matching                           |
| `-r`   | Regular expression matching                 |
| `-l`   | Do not trim output to terminal width        |
| `-s`   | Specify the section(s) to search            |

For `-s` you can use those below to specify the section:

- `1`: commands.
- `2`: system calls.
- `3`: library functions.
- `4`: special files (usually devices).
- `5`: file formats and conventions.
- `6`: games.
- `7`: potpourri.
- `8`: system administration commands and daemons.
- `9`: kernel routines.

### `awk`

| Option | Description                                  |
| ------ | -------------------------------------------- |
| `-F`   | Specify the input field separator            |
| `-f`   | Specify the file containing the `awk` script |

**Digressing**: `awk` comes from the initials of its three creators:
`Alfred Aho`, `Peter Weinberger`, and `Brian Kernighan`.

#### Column Variables

- `$0`: the whole line.
- `$1`: the first column.
- `...`
- `$NF`: the last column (where `NF` is the number of fields in the current record).

For example, `awk '{print $1, $2, $NF}' file` will print the first column, second column,
and last column of each line in `file`.

#### Separators

You can specify the output field separator (OFS) using the `OFS` variable.
For example, `awk '{print $1,$2,$NF}' OFS=","` will print the first column,
second column, and last column of each line in `file`, separated by a comma.

Is is also possible to specify `OFS` in the `BEGIN` mode string, for example:
`awk 'BEGIN {OFS=","} {print $1,$2,$NF}' file` does the same thing.

Besides, `FS` is used to specify the input field separator.

If you want to change the separators, you may write the wrong command like
`awk '{print}' FS=':' OFS=' '` trying to change `:` to a space, but this will not work as expected.
In `awk`, it will only use the new `OFS` when printing multiple fields,
or when the fields are modified. Therefore, we can use
`awk '($1=$1) || 1' FS=':' OFS=' '` to update the `OFS`,
which is trying to change the first field to itself.
And we use `|| 1` here to make sure the empty lines are also printed.

**NOTE**: The parentheses around `$1=$1` are necessary, because without them,
`awk` would interpret it as `$1=($1 || 1)`,
which would assign `1` to `$1` instead of keeping its original value.

#### Predefined Variables

`OFS`, `FS` are predefined variables in `awk`,
which are used to specify the output and input field separators respectively.

And there are some other predefined variables in `awk`:

- `NR`: current record number starting from `1`.
- `NF`: number of fields in the current record .
- `RS`: input record separator, default is newline.
- `ORS`: output record separator, default is newline.
- `FILENAME`: the name of the current input file.
- `FNR`: current record number in the current file.

We can use `awk 'NR > 1'` to print all lines except the first line,
and `awk 'NF > 0'` to print non-empty lines (i.e., remove empty lines).

We can use `awk 'NR == 1, NR == 4'` or `awk 'NR >= 1 && NR <= 4'` to print the first four lines.

#### Patterns

`BEGIN` and `END`:

- `BEGIN`: executed before processing any input.
- `END`: executed after processing all input.

For example, you can use
`awk 'BEGIN {print "Start"} {print} END {print "End"}'`
to print the contents of a file with `Start` and `End` messages.

In `awk`, you can use patterns to filter lines,
for example, `awk '$1 > 10'` means to print only the lines
where the first column is greater than `10`
(when no action is specified, the default action is to print the whole line).
`awk '/pattern/'` means to print only the lines that contain `pattern`,
and the search pattern supports regular expressions.
`awk '/pattern1/,/pattern2/'` means to print all lines from the first line that matches `pattern1`
to the first line that matches `pattern2`.

You can use `~` and `!~` to search for patterns in a column.
For example, `awk '$1 ~ /pattern/'` means to print only the lines
that match the regular expression `pattern` in the first column.
`awk '$1 !~ /pattern/'` means to print only the lines
that do not match the regular expression `pattern` in the first column.

#### `awk` Scripts

For some complex tasks, you can write `awk` scripts.
Here is an example of a simple `awk` script that statistics the number of words in a file:

```awk
#! /usr/bin/awk -f

BEGIN {
    # Update the input and output field separators
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

**NOTE**: `awk file` is not the right way to run an `awk` script,
and we must use the `-f` option to specify the script file.

### `cat`

| Option | Description                                                                      |
| ------ | -------------------------------------------------------------------------------- |
| `-n`   | Show line numbers                                                                |
| `-b`   | Only show non-empty lines numbers                                                |
| `-s`   | Suppress repeated empty lines                                                    |
| `-v`   | Use `^` and `M-` to display non-printable characters, except for tab and newline |
| `-E`   | Display `$` at the end of each line                                              |
| `-T`   | Display `^I` for tab characters                                                  |
| `-A`   | Equivalent to `-vET`                                                             |
| `-e`   | Equivalent to `-vE`                                                              |
| `-t`   | Equivalent to `-vT`                                                              |

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

### `find`

| Option        | Description                                                  |
| ------------- | ------------------------------------------------------------ |
| `-type`       | Specify the type of file to search for                       |
| `-readable`   | Files or directories that are readable                       |
| `-writable`   | Files or directories that are writable                       |
| `-executable` | Files or directories that are executable                     |
| `-name`       | File name to search for                                      |
| `-path`       | Path to search for                                           |
| `-iname`      | Case-insensitive file name search                            |
| `-ipath`      | Case-insensitive path search                                 |
| `-empty`      | Empty files or directories                                   |
| `-size`       | Size of the files or directories to search for               |
| `-exec`       | Execute a command on the found files or directories          |
| `-perm`       | Permissions to search for                                    |
| `-user`       | Owner of the file or directory                               |
| `-group`      | Group of the file or directory                               |
| `-maxdepth`   | Maximum depth to search                                      |
| `-mindepth`   | Minimum depth to search                                      |
| `-depth`      | Same with `-maxdepth`                                        |
| `-delete`     | Delete the found files or directories                        |
| `-and`        | Logic and                                                    |
| `-or`         | Logic or                                                     |
| `-not`        | Logic not                                                    |
| `-regex`      | Regular expression                                           |
| `-iregex`     | Case-insensitive regular expression                          |
| `-print0`     | Print the found files or directories separated by null       |
| `-samefile`   | Files that are hard links to the specified file              |
| `-links`      | Files with a specific number of hard links                   |
| `-P`          | Never follow symbolic links (default)                        |
| `-L`          | Follow symbolic links                                        |
| `-H`          | Follow symlinks only for ones explicitly passed as arguments |
| `-mtime`      | Modification time of the file or directory in days           |
| `-atime`      | Access time of the file or directory in days                 |
| `-ctime`      | Creation time of the file or directory in days               |
| `-mmin`       | Modification time of the file or directory in minutes        |
| `-amin`       | Access time of the file or directory in minutes              |
| `-cmin`       | Creation time of the file or directory in minutes            |

The options for `-type`:

- `b`: block device file
- `c`: character device file
- `p`: pipe file
- `s`: socket file
- `f`: regular file
- `d`: directory
- `l`: soft link file

**NOTE**: When you use `-regex`, the pattern is matched against the entire file name,
which is a little bit different from `grep`. If you want to match a specific part of the file name,
you need to use `.*` to match any characters before and after the pattern. For example,
`find . -regex ".*pattern.*"` will find files that contain `pattern` in their names.

The units for `-size`:

- `b`: block size, which is decided by the file system (usually 512 bytes)
- `c`: bytes
- `k`: kilobytes (1024 bytes)
- `M`: megabytes (1024 kilobytes)
- `G`: gigabytes (1024 megabytes)

You can use `find . -size 1M` to find files that are exactly `1` megabyte in size;
use `find . -size +1M` to find files larger than `1` megabyte;
use `find . -size -1M` to find files smaller than `1` megabyte;
use `find . -size +1M -size -2M` to find files
larger than `1` megabyte but smaller than `2` megabytes.

You can use `-exec` to execute a command on the found files or directories.

For example, `find . -name "*.txt" -exec ls -l {} \;`
will find all `txt` files in the current directory and execute `ls -l` on each of them.
`{}` will be replaced by the found file name, `\;` means the end of the command,
and the semicolon needs to be escaped to prevent it from being interpreted by the shell.

### `grep`

| Option          | Description                                             |
| --------------- | ------------------------------------------------------- |
| `-i`            | Ignore case                                             |
| `-v`            | Invert match                                            |
| `-o`            | Only show the matching part                             |
| `-e`            | Specify a pattern explicitly                            |
| `--include`     | Search only in files matching the pattern               |
| `--exclude`     | Skip files matching the pattern                         |
| `--exclude-dir` | Skip directories matching the pattern                   |
| `-A 2`          | Show matching lines and the next two lines              |
| `-B 2`          | Show matching lines and the previous two lines          |
| `-C 2`          | Show matching lines and two lines before and after      |
| `-r`            | Recursively search directories                          |
| `-n`            | Show line numbers                                       |
| `-F`            | Interpret the pattern as a fixed string                 |
| `-w`            | Only match whole words                                  |
| `-x`            | Only match whole lines                                  |
| `-m`            | Stop after N matches for each file                      |
| `-c`            | Only show the count of match in each file               |
| `-l`            | Only show the names of files with matches               |
| `-L`            | Only show the names of files without matches            |
| `-H`            | Print file names in output                              |
| `-h`            | Do not print file names in output                       |
| `--color`       | Highlight rules, auto, always, or never                 |
| `-f`            | Read patterns from a file, one pattern per line         |
| `-E`            | Interpret the pattern as an extended regular expression |

The difference between extended regular expressions (ERE) and basic regular expressions (BRE)
is that in ERE, `?`, `+`, `{`, `|`, `(`, and `)` are special characters,
while in BRE, they are not special characters unless escaped with a backslash (`\`).

**NOTE**: `-e` is useful when you
want to specify multiple patterns or the pattern starts with a hyphen (`-`).
When you use multiple `-e` options,
`grep` will search for lines that match any of the specified patterns (logical OR).

**NOTE**: We usually use `egrep` as an alias for `grep -E`, and `fgrep` as an alias for `grep -F`.

**NOTE**: Word-constituent characters are letters, digits, and underscores.

**NOTE**: In shell command, you usually need to quote the pattern to prevent pathname expansion.

**Regression**: `grep` comes from the command `g/re/p`,
where `g` stands for `global`, `re` stands for `regular expression`, and `p` stands for `print`.

### `ln`

| Option | Description                                                          |
| ------ | -------------------------------------------------------------------- |
| `-s`   | Soft link                                                            |
| `-f`   | Force the creation of the link, removing existing files if necessary |
| `-t`   | Specify the target directory for the link                            |

**NOTE**: When using `ln target link_name`, and `link_name` is an existing directory,
it will create a file named `target` in that directory.

**NOTE**: `rm` and `unlink` commands can both delete symbolic links or hard links.
But `unlink` can only delete one file at a time,

#### Hard Links and Soft Links

You can use `ln` to create hard links and soft links (symbolic links) in `Linux`.
Hard links are files that point to the same `inode` as the original file,
while soft links are files that point to the original file by its path.
Besides, hard links and soft links have the following differences:

- Soft links can point to files in different file systems,
  while hard links can only point to files in the same file system.
- Soft links can point to directories, while hard links cannot point to directories.
- When the source file is deleted, hard links will still be valid,
  while soft links will become invalid.

In Linux, you can use the `ls -l` command to view the number of hard links to a file.

### `sed`

| Option       | Description                                          |
| ------------ | ---------------------------------------------------- |
| `-i[SUFFIX]` | Edit files in place, optionally with a backup suffix |
| `-e`         | Add a script to the commands to be executed          |
| `-f`         | Add a script file to the commands to be executed     |
| `-E`         | Use extended regular expressions                     |
| `-n`         | Suppress automatic printing of pattern space         |

**NOTE**: With `-n`, `sed` will only print lines explicitly specified with the `p`, `P`, or `w`.

#### Pattern Space and Hold Space

`sed` uses two spaces: the pattern space and the hold space.

The pattern space is where the current line is processed.
`sed` will read one line at a time from the input into the pattern space.

The hold space is a temporary storage area that can be used to store data between commands.
It is empty until you explicitly use some commands to store data in it.
And content in the hold space persists across multiple lines.

#### Commands

| Command | Description                                                                          |
| ------- | ------------------------------------------------------------------------------------ |
| `s`     | Substitute a pattern in the pattern space                                            |
| `p`     | Print the pattern space                                                              |
| `P`     | Print the first part of the pattern space (up to the first newline)                  |
| `d`     | Delete the pattern space                                                             |
| `D`     | Delete the first part of the pattern space (up to the first newline)                 |
| `q`     | Quit immediately                                                                     |
| `h`     | Copy the pattern space to the hold space                                             |
| `H`     | Append the pattern space to the hold space                                           |
| `g`     | Copy the hold space to the pattern space                                             |
| `G`     | Append the hold space to the pattern space                                           |
| `n`     | Read the next line into the pattern space                                            |
| `N`     | Append the next line to the pattern space                                            |
| `x`     | Exchange the pattern space and the hold space                                        |
| `=`     | Print current line number                                                            |
| `w`     | Write the pattern space to a file                                                    |
| `W`     | Write the first part of the pattern space (up to the first newline) to a file        |
| `a\`    | Append text after the current line (use `\` to continue on the next line)            |
| `c\`    | Change the current line to the specified text (use `\` to continue on the next line) |
| `i\`    | Insert text before the current line (use `\` to continue on the next line)           |

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

### `sort`

| Option                   | Description                                                |
| ------------------------ | ---------------------------------------------------------- |
| `-r`                     | Sort in reverse order                                      |
| `-n`                     | Sort numerically                                           |
| `-k`                     | Sort by a specific column                                  |
| `-u`                     | Unique sort                                                |
| `-t`                     | Specify the field separator                                |
| `-c`                     | Check if the input is already sorted                       |
| `-f`                     | Ignore case when sorting                                   |
| `-h`                     | Sort by human-readable numbers (e.g., 1K, 2M)              |
| `-M`                     | Sort by month names (e.g., Jan, Feb)                       |
| `--files0-from=-`        | Read file names from standard input with null as separator |
| `--files0-from=filename` | Read file names from a file with null as separator         |

**NOTE**: You can use `-k` to specify multiple keys for sorting.
For example, `-k2,2 -k1,1` means sort by the second column first, then by the first column.
And you can specify parts of a column for sorting,
such as `-k2.2,2.3` to sort by the second column from the second character to the third character.
Besides, you can specify the type of the column for sorting,
such as `-k2n,2` to sort the second column numerically
and `-k2r,2` to sort the second column in reverse order.

### `ssh`

`ssh` can be divided into two parts: `client` and `server`.
`client` is used to connect to remote hosts,
while `server` is used to accept connections from remote hosts.
`openssh-client` and `openssh-server` are the two main components of `ssh`.

| Option | Description                                |
| ------ | ------------------------------------------ |
| `-p`   | Specify the port                           |
| `-i`   | Specify the private key                    |
| `-l`   | Specify the username                       |
| `-q`   | Quiet mode                                 |
| `-t`   | Force pseudo-terminal allocation           |
| `-v`   | Verbose                                    |
| `-C`   | Enable compression                         |
| `-X`   | Limited `X11` forwarding                   |
| `-Y`   | Trusted `X11` forwarding                   |
| `-f`   | Run in the background after authentication |
| `-N`   | Do not execute remote commands             |
| `-L`   | Local port forwarding                      |
| `-R`   | Remote port forwarding                     |
| `-D`   | Dynamic port forwarding                    |
| `-o`   | Specify options                            |

**NOTE**: We can use `user@host` to substitute `-l user host` when connecting to a remote host.

**NOTE**: As to port forwarding, check [SSH Port Forwarding](/blog/2024/ssh-port-forwarding) for
details.

**NOTE**: `-o` is often used to override the configurations in `~/.ssh/config`.
For example, `ssh -o "Port=2222" host_name` will connect to `host_name` using port `2222`,
while keeping other configurations unchanged.

#### `scp`

| Option | Description                                      |
| ------ | ------------------------------------------------ |
| `-r`   | Recursively                                      |
| `-P`   | Specify the port                                 |
| `-p`   | Reserve file attributes                          |
| `-q`   | Quiet mode                                       |
| `-v`   | Verbose                                          |
| `-C`   | Compression                                      |
| `-i`   | Specify the private key                          |
| `-l`   | Limit the bandwidth (in Kbit/s)                  |
| `-3`   | Copy between two remote hosts via the local host |
| `-4`   | Force use of `IPv4`                              |
| `-6`   | Force use of `IPv6`                              |

You can use `scp` to copy multiple files at once, for example,
`scp file1 file2 host_name:/path/to/`。

#### `ssh-add`

| Option | Description                      |
| ------ | -------------------------------- |
| `-l`   | List fingerprints of loaded keys |
| `-L`   | List public keys of loaded keys  |
| `-d`   | Delete a specific key            |
| `-D`   | Delete all keys                  |
| `-x`   | Lock `ssh-agent`                 |
| `-X`   | Unlock `ssh-agent`               |

**NOTE**: If there is `Could not open a connection to your authentication agent`
while executing the command, it may be because `ssh-agent` is not started.
You can start `ssh-agent` by using `eval $(ssh-agent)`.

#### `ssh-agent`

`ssh-agent` is used to manage user's keys.
Once a key is added to `ssh-agent`,
the user can log in without entering the key's password in subsequent login processes.

You can use `eval $(ssh-agent)` to start `ssh-agent` in the current shell.

#### `ssh-copy-id`

| Option | Description                         |
| ------ | ----------------------------------- |
| `-i`   | Specify the public key to be copied |
| `-p`   | Specify the port                    |

#### `ssh-keygen`

| Option | Description                                |
| ------ | ------------------------------------------ |
| `-a`   | Specify the number of KDF rounds           |
| `-t`   | Specify the type of key to create          |
| `-b`   | Specify the number of bits in the key      |
| `-C`   | Add comment to the key                     |
| `-c`   | Change the comment of the key              |
| `-f`   | Specify the file name of the key           |
| `-P`   | Specify the password of the key            |
| `-p`   | Change the password of the key             |
| `-l`   | Show the fingerprint of the key            |
| `-H`   | Hash the key in the `known_hosts` file     |
| `-R`   | Remove the key from the `known_hosts` file |

**NOTE**: Starting with `OpenSSH 9.5`,
the default generated key is `ed25519` rather than `rsa`.
`ed25519` is more secure than rsa and also offers better performance.

#### `~/.ssh/known_hosts`

This file is used to store the public keys of remote hosts.
When you connect to a remote host for the first time,
`ssh` will prompt you to confirm the authenticity of the host, and if you confirm,
`ssh` will store the public key of the remote host in the `known_hosts` file.
The next time you connect to the remote host,
`ssh` will check if the public key of the remote host matches the one in the
`known_hosts` file.
If it matches, the connection is successful; otherwise, it will prompt that the public key does
not match.

The main purpose of this file is to prevent `Man-in-the-middle` attacks.
When an attacker intercepts the connection between the client and the server,
they can pretend to be the server and send their own public key to the client.
If the client does not have the server's public key in the `known_hosts` file,
they may accept the attacker's public key, allowing the attacker to intercept and modify
the communication between the client and the server.

By default, the content of `known_hosts` is stored in plain text,
and each line contains the remote host's address and its public key.
You can use `ssh-keygen -H` to hash the remote host's address in the `known_hosts` file.
This can enhance security by preventing attackers from easily obtaining the remote host's address
and its corresponding public key if the `known_hosts` file is leaked.

#### `~/.ssh/config`

This file is used to configure `ssh` options for different remote hosts. For example:

```shell
Host host_name
    HostName host_ip
    Port port
    User user_name
    IdentityFile ~/.ssh/id_rsa
```

When you have the above configuration, you can connect to the remote host using
`ssh host_name` without specifying the remote host's `ip`, `port`, `user`, and `identity file`.

You can use `*` as a wildcard to match multiple hosts, and this can be used to set
default configurations for all hosts. For example:

```shell
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

Other glob patterns:

- `*`: matches any number of characters
- `?`: matches any single character
- `!`: excludes the following pattern

### `su`

| Option | Description                                                        |
| ------ | ------------------------------------------------------------------ |
| `-l`   | Switch user as login shell                                         |
| `-c`   | Execute command as the specified user                              |
| `-s`   | Specify shell                                                      |
| `-p`   | Retain `HOME`, `SHELL`, `USER` and `LOGNAME` environment variables |

**NOTE**: When use `su` without specifying a user, it will switch to the `root` user by default.

**NOTE**: If you do not use `-l`, `su` will not switch the user's environment variables.
When using `-l`, the following will be executed in order:

> 1. clears all the environment variables except `TERM` and variables specified by `--whitelist-environment`
> 2. initializes the environment variables `HOME`, `SHELL`, `USER`, `LOGNAME`, and `PATH`
> 3. changes to the target user’s home directory
> 4. sets `argv[0]` of the shell to `-` in order to make the shell a login shell

### `tar`

| Option        | Description                                   |
| ------------- | --------------------------------------------- |
| `-c`          | Create                                        |
| `-f`          | Specify the name                              |
| `-v`          | Verbose                                       |
| `-z`          | Use `gzip`                                    |
| `-j`          | Use `bzip2`                                   |
| `-J`          | Use `xz`                                      |
| `-x`          | Extract                                       |
| `-C`          | Working directory                             |
| `-t`          | List contents                                 |
| `--wildcards` | Enable wildcard matching for file names       |
| `--delete`    | Delete files from a tar archive               |
| `--exclude=`  | Exclude files or directories                  |
| `-r`          | Append files to a tar archive                 |
| `-A`          | Append another tar archive to the current one |
| `-u`          | Update files in a tar archive                 |

**NOTE**: When using `--exclue`,
`--exclude` must appear before the files or directories to be packed.
For example, `tar -cf a.tar --exclude=*.txt .` is correct,
but `tar -cf a.tar . --exclude=*.txt` is incorrect.

**NOTE**: When using `-u`, it will not overwrite old files, but will append new files directly,
after which the tar archive may contain multiple files with the same name.
But I have not found a way to extract the old files.

### `ssh`

`ssh` can be divided into two parts: `client` and `server`.
`client` is used to connect to remote hosts,
while `server` is used to accept connections from remote hosts.
`openssh-client` and `openssh-server` are the two main components of `ssh`.

| Option | Description                                |
| ------ | ------------------------------------------ |
| `-p`   | Specify the port                           |
| `-i`   | Specify the private key                    |
| `-l`   | Specify the username                       |
| `-q`   | Quiet mode                                 |
| `-t`   | Force pseudo-terminal allocation           |
| `-v`   | Verbose                                    |
| `-C`   | Enable compression                         |
| `-X`   | Limited `X11` forwarding                   |
| `-Y`   | Trusted `X11` forwarding                   |
| `-f`   | Run in the background after authentication |
| `-N`   | Do not execute remote commands             |
| `-L`   | Local port forwarding                      |
| `-R`   | Remote port forwarding                     |
| `-D`   | Dynamic port forwarding                    |
| `-o`   | Specify options                            |

**NOTE**: We can use `user@host` to substitute `-l user host` when connecting to a remote host.

**NOTE**: As to port forwarding, check [SSH Port Forwarding](/blog/2024/ssh-port-forwarding) for
details.

### `tee`

| Option | Description              |
| ------ | ------------------------ |
| `-a`   | Append                   |
| `-i`   | Ignore interrupt signals |

We usually use `tee` in pipelines. For example,
when we want to write some content of a command to a file,
but the file requires `root` permission, we can use `sudo tee` to achieve this.

### `usermod`

| Option | Description                                                                |
| ------ | -------------------------------------------------------------------------- |
| `-l`   | Modify the username                                                        |
| `-u`   | Modify the `UID` of the user                                               |
| `-g`   | Modify the primary group of the user                                       |
| `-G`   | Modify the supplementary groups of the user                                |
| `-a`   | Used with `-G` to append the user to supplementary groups                  |
| `-d`   | Modify the home directory of the user                                      |
| `-m`   | Used with `-d` to move the contents of the home directory at the same time |
| `-s`   | Modify the login shell of the user                                         |
| `-o`   | Allow non-unique `UID`                                                     |
| `-c`   | Update the user's comment field                                            |
| `-e`   | Specify the expiration date                                                |
| `-p`   | Modify the password                                                        |
| `-L`   | Lock the user                                                              |
| `-U`   | Unlock the user                                                            |

**NOTE**: When you use `-p` to update the password, the password should be encrypted.
You can use `openssl passwd` to generate an encrypted password.
But it is more recommended to use the `passwd` command to change the password.

**NOTE**: The newly created user will not expire by default.
If you set an expiration date, you can use `chmod -e ""` to cancel it.
When the user expires, the user will not be able to log in, but the user still exists,

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

- `=`: This is like references in `C/C++`.
  For example, if you use `a = $(b)`, once `b`'s value changed after, the `a` will change too.
- `:=`: This is assignment equal sign, which is similar with `=` in `C/C++`.
- `+=`: This is to append a variable with new values.
- `?=`: This will check if the variable is assigned before;
  if it is, this will not work, otherwise it is similar with `=`.

For the `+=`:

- If the variable has not been defined, it will be `=`.
- If the variable has been define, it will follow the last equal sign.
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

- `$@`: A variable in `make`, whose value is the target.
  For example, if `$@` appears in commands following `main.o: main.cpp`,
  `$@` will be `main.o` exactly.
- `$<`: A variable in `make`, whose value is the first dependency.
  For example, if `$<` appears in commands following `main.o: main.cpp`,
  `$<` will be `main.cpp` exactly.
- `$*`: A variable in `make`, whose value is the stem of the target.
  For example, using `$*` following `pre_%.o: pre_%.c`, and the target is `pre_foo.o`,
  `$*` will be `foo`.
- `$%`: When the target is in an archive (like `foo.a`),
  this variable is the names of members.
  For example, if a target is `foo.a(bar.o)` the `$%` will be `bar.o` and the `$@` will be `foo.a`.
- `$?`: A variable in `make`, whose value is all the dependencies that are newer than the target.
- `$^`: A variable in `make`, whose value is all the dependencies of the target.
  This will have only one copy if the target depends on the same file more than once.
- `$+`: A variable similar with `$^` but will store the repetitive files.

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

- `C`: `*.o` will be deducted depending on `*.c`
  and the build command will be `$(CC) -c $(CPPFLAGS) $(CFLAGS)`.
- `C++`: `*.o` will be deducted depending on`*.C, *.cc or *.cpp`
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
which is used to specify the directories
where `make` will look for files when they are not found in the current directory.
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

- `$(subst from,to,text)`: substitute `from` with `to` in `text`.
- `$(patsubst pattern,replacement,text)`: substitute `pattern` with `replacement` in `text`.
- `$(strip text)`: remove all the leading and trailing spaces in `text`.
- `$(findstring target,text)`: find `target` in text, if found, return `target` otherwise,
  return empty string.
- `$(filter pattern...,text)`: filter the contents matching the `pattern...` from `text`.
- `$(filter-out pattern...,text)`: filter out the contents matching the `pattern...` from `text`.
- `$(sort list)`: sort the contents in `list` lexicographically.
  Note that `sort` will unique the contents, too.
- `$(word i,text)`: get the `i`-th word from `text` (index started from `1`).
- `$(wordlist l,r,text)`: get the words whose index is in `[l,r]` in sequence.
- `$(firstword text)`: get the first word of `text`.

There are some functions related to files:

- `$(dir name...)`: get the directory part from `name`.
- `$(notdir name...)`: get the non-directory part from `name`.
- `$(suffix name...)`: get the suffixes of files from `name`.
- `$(basename name...)`: get the base name (files' name without extension) of files from `name`.
- `$(addsuffix suffix,name)`: add `suffix` for `name`.
- `$(addprefix prefix,name)`: add `prefix` for `name`.
- `$(join list1,list2)`: join two lists. This will append words in `list2` to `list1`.
  For example `$(join aaa bbb,111 222 333)` will get `aaa111 bbb222 333`.

---

If I want add a suffix for every item in a variable, I can do it with this:

```Makefile
names := a b c d
files := $(foreach item,$(names),$(item).o)
```

---

- `$(if condition,then-part,else-part)`: if the `condition` is a non-empty string,
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

- `undefined`: never defined before.
- `environment`: the variable comes from environment variables.
- `default`: the default variable, such `CC`.
- `file`: the variable is defined in a `Makefile`.
- `command line`: the variable is defined by command lines
  (when you type `make a=1`, `a` is defined by command lines).
- `override`: the variable if defined by `override`.
- `automatic`: the variable is defined by `make` automatically.

---

This is to execute a command in shell:

```Makefile
contents := $(shell cat foo)
files := $(shell echo *.c)
```

The commands above will get the output of a shell command.

---

- `$(erro text)`: output `text` and stop `make`.
- `$(warning text)`: output `text` but don't stop `make`.

### Return Value of Make

- `0`: success.
- `1`: some errors.
- `2`: when you use `-q` (`-q` will not run `make`,
  but give you a return value `0` if the targets are up to date)
  and `make` cannot make sure whether or not the files is up to date.

### Specify Target

If you run `make`, `make` will build the first target.
But you can specify target by `make targetname`.

There are some rules you should obey for naming a target when you write `Makefile`:

- `all`: build all targets.
- `clean`: remove all the files created by `make`.
- `install`: install the targets, when in `C/C++`,
  this will move the binary files to `/usr/bin` and move the header files to `/usr/include`.
- `print`: print the files having been updated.
- `tar`: pack the source files into a tar file.
- `dist`: create a compressed file including all source files.

### Check Make Syntax

When you want to check your syntax in `Makefile` rather than run the `make`,
you can use those options: `-n, --just-print, --dry-run, --recon`. The four options are synonyms.

### Other Options in make

| Option                       | Description                                              |
| ---------------------------- | -------------------------------------------------------- |
| `-j`                         | Use the specified number of jobs (cores) to build        |
| `-q`                         | Check if the target exists                               |
| `-W`                         | Build the targets that depend on the specified file      |
| `-o`                         | Ignore the specified file while building                 |
| `-B`                         | Always re-build all targets, even if they are up to date |
| `-C`                         | Change working directory to the specified directory      |
| `-t`                         | Touch all the targets                                    |
| `--debug`                    | Print debug info                                         |
| `-e`                         | Environment overrides                                    |
| `-f`                         | Use the specified file as `Makefile`                     |
| `-i`                         | Ignore errors                                            |
| `-I`                         | Include directory                                        |
| `-k`                         | Keep going even if there are errors                      |
| `-S`                         | Stop when an error occurs                                |
| `-p`                         | Print the data base                                      |
| `-r`                         | Do not use built-in rules                                |
| `-R`                         | Do not use built-in variables                            |
| `-s`                         | Silent mode, do not print commands                       |
| `-w`                         | Print the working directory before and after processing  |
| `--no-print-directory`       | Do not print the working directory                       |
| `--warn-undefined-variables` | Warn when a variable is undefined                        |

`--debug=options` will print the debug info of `make`, the available options are:

- `a`: print all info.
- `b`: print basic info.
- `v`: verbose.
- `i`: print implicit rules.
- `j`: print jobs' info including `PID`, return value and so on.
- `m`: this is for debugging when remaking makefiles.
- If you just use `-d`, this is same with `--debug=a`.

You have learned a lot about `make`, why not to read the
[Makefile](https://github.com/torvalds/linux/blob/master/Makefile)
of
[Linux Kernel](https://github.com/torvalds/linux).

## References

- [10 Practical Examples Using Wildcards to Match Filenames in Linux](https://www.tecmint.com/use-wildcards-to-match-filenames-in-linux/)
- [Man Page of Bash](https://www.gnu.org/software/bash/manual/bash.html)
- [How to Use the awk Command on Linux](https://www.howtogeek.com/562941/how-to-use-the-awk-command-on-linux/)
- [30+ awk examples for beginners / awk command tutorial in Linux/Unix](https://www.golinuxcloud.com/awk-examples-with-command-tutorial-unix-linux/)
- [8 Powerful Awk Built-in Variables – FS, OFS, RS, ORS, NR, NF, FILENAME, FNR](https://www.thegeekstuff.com/2010/01/8-powerful-awk-built-in-variables-fs-ofs-rs-ors-nr-nf-filename-fnr/)
- [Getting Started With AWK Command](https://linuxhandbook.com/awk-command-tutorial/)
- [cat command examples for beginners](https://www.golinuxcloud.com/cat-command-examples/)
- [25+ most used find commands in Linux](https://www.golinuxcloud.com/find-command-in-linux/)
- [find Linux Command Cheatsheet](https://onecompiler.com/cheatsheets/find)
- [Linux Find Cheatsheet](https://linuxtutorials.org/linux-find-cheatsheet/)
- [Find files and directories on Linux with the find command](https://opensource.com/article/21/9/linux-find-command)
- [10 ways to use the Linux find command](https://www.redhat.com/sysadmin/linux-find-command)
- [Find cheatsheet](https://quickref.me/find)
- [Linux Tutorial - Cheat Sheet - grep](https://ryanstutorials.net/linuxtutorial/cheatsheetgrep.php)
- [20 grep command examples in Linux](https://www.golinuxcloud.com/grep-command-in-linux/)
- [Sed Command Cheat Sheet: 30 Essential One-Liners for Text Processing](https://karandeepsingh.ca/posts/sed-command-cheat-sheet-30-essential-one-liners/)
- [sed cheatsheet](https://devhints.io/sed)
- [Linux Handbook: sort Command Examples](https://linuxhandbook.com/sort-command/)
- [15+ Tips to PROPERLY sort files in Linux](https://www.golinuxcloud.com/linux-sort-files/#1_Sort_by_name)
- [Linux and Unix sort command tutorial with examples](https://shapeshed.com/unix-sort/)
- [Linux sort Command](https://www.baeldung.com/linux/sort-command)
- [Linux Audit: tar cheat sheet](https://linux-audit.com/cheat-sheets/tar/)
- [15+ tar command examples in Linux](https://www.golinuxcloud.com/tar-command-in-linux/)
- [How to write makefiles](https://github.com/seisman/how-to-write-makefile).
- [Ignoring files](https://docs.github.com/en/get-started/getting-started-with-git/ignoring-files)
- [10+ practical examples to create symbolic link in Linux](https://www.golinuxcloud.com/create-symbolic-link-linux/)
- [15+ scp command examples in Linux](https://www.golinuxcloud.com/scp-command-in-linux/)
- [apropos Linux Command Explained](https://phoenixnap.com/kb/apropos-linux)
- [10 tee command examples in Linux](https://www.golinuxcloud.com/tee-command-in-linux/)
- [How To Use ‘sudo’: The Complete Linux Command Guide](https://raspberrytips.com/sudo-linux-command/)
- [Linux visudo command](https://www.computerhope.com/unix/visudo.htm)
- [15 usermod command examples in Linux](https://www.golinuxcloud.com/usermod-command-in-linux/)
- [Linux mount command to access filesystems, iso image, usb, network drives](https://www.golinuxcloud.com/linux-mount-command-iso-usb-network-drive/)
- [The “mount” Command in Linux](https://linuxsimply.com/mount-command-in-linux/)
- [9 su command examples in Linux](https://www.golinuxcloud.com/su-command-in-linux/)
- [15+ SSH command examples in Linux](https://www.golinuxcloud.com/ssh-command-in-linux/)
- [15+ lsof command examples in Linux](https://www.golinuxcloud.com/lsof-command-in-linux/)
- [OpenSSH 9.5: A User-Friendly Guide to the Latest Update](https://stilia-johny.medium.com/openssh-9-5-a-user-friendly-guide-to-the-latest-update-840a09886a5a)
- [How to generate and manage ssh keys on Linux](https://linuxconfig.org/how-to-generate-and-manage-ssh-keys-on-linux)
- [10 examples to generate SSH key in Linux (ssh-keygen)](https://www.golinuxcloud.com/generate-ssh-key-linux/)
- [The Complete Guide to SSH Config Files](https://thelinuxcode.com/ssh-config-file/)
- [Beginners guide to use ssh config file with examples](https://www.golinuxcloud.com/ssh-config/)
- [How to use the command `ssh-add` (with examples)](https://commandmasters.com/commands/ssh-add-common/)
- [5 Unix / Linux ssh-add Command Examples to Add SSH Key to Agent](https://linux.101hacks.com/unix/ssh-add/)
- [How To Use SFTP to Securely Transfer Files with a Remote Server](https://www.digitalocean.com/community/tutorials/how-to-use-sftp-to-securely-transfer-files-with-a-remote-server)
- [libfuse/sshfs](https://github.com/libfuse/sshfs)
