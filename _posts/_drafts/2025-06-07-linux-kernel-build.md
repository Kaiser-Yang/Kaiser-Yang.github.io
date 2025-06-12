---
layout: post
title: 'Linux Kernel Learning: Build'
date: 2025-06-07 18:57:47+0800
last_updated: 2025-06-07 18:57:47+0800
description:
tags:
  - Linux
categories: Potpourri
featured: true
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

## Beautify Output

In the Linux kernel build system, it uses three types of commands to beautify the output:

* `quiet_cmd_*`: There are actually not runnable commands,
but rather a way to define a short description of the command.
* `cmd_*`: These are the actual commands that will be executed.
* `silent_cmd_*`: These are all empty string.

**NOTE**: Actually, there are no commands starting with `silent_cmd_`.
I list them here for better understanding.

With these three types of commands,
the user can use the variable `V` or `-s` option to control the verbosity of the output.

There are two variables defined in the root Makefile:

* `Q`: empty or `@`.
This is used to control whether the command will be printed.
* `quiet`: empty, `quiet_`, or `silent_`.
This is used to control the type of command to be printed.

When building the kernel with `-s`, the `Q` variable is set to `@`,
and the `quiet` variable is set to `silent_`, which means no commands will be printed.

When building the kernel with `V=`:

* `V=1`: The whole command is printed.
* `V=2`: The reason for rebuilding is printed.
* `V=12`: The whole command is printed, and the reason for rebuilding is printed.

Here is an example of how these variables are used in the Makefile,
we use the `CHECK` command as an example:

```Makefile
# From scripts/Makefile.build
quiet_cmd_checksrc = CHECK   $<
      cmd_checksrc = $(CHECK) $(CHECKFLAGS) $(c_flags) $<

# From scripts/Kbuild.include
ifneq ($(findstring 2, $(KBUILD_VERBOSE)),)
_why =                                                                       \
    $(if $(filter $@, $(PHONY)),- due to target is PHONY,                    \
        $(if $(wildcard $@),                                                 \
            $(if $(newer-prereqs),- due to: $(newer-prereqs),                \
                $(if $(cmd-check),                                           \
                    $(if $(savedcmd_$@),- due to command line change,        \
                        $(if $(filter $@, $(targets)),                       \
                            - due to missing .cmd file,                      \
                            - due to $(notdir $@) not in $$(targets)         \
                         )                                                   \
                     )                                                       \
                 )                                                           \
             ),                                                              \
             - due to target missing                                         \
         )                                                                   \
     )

why = $(space)$(strip $(_why))
endif
silent_log_print = exec >/dev/null;
 quiet_log_print = $(if $(quiet_cmd_$1), echo '  $(call escsq,$(quiet_cmd_$1)$(why))';)
       log_print = echo '$(pound) $(call escsq,$(or $(quiet_cmd_$1),cmd_$1 $@)$(why))'; \
                   echo '  $(call escsq,$(cmd_$1))';
delete-on-interrupt = \
    $(if $(filter-out $(PHONY), $@), \
        $(foreach sig, HUP INT QUIT TERM PIPE, \
            trap 'rm -f $@; trap - $(sig); kill -s $(sig) $$$$' $(sig);))
cmd = @$(if $(cmd_$(1)),set -e; $($(quiet)log_print) $(delete-on-interrupt) $(cmd_$(1)),:)
```

In the example above, the `quiet_cmd_checksrc` is defined as `CHECK   $<`, which
is a short description of the command. `cmd_checksrc` is the actual command that will
be executed. Use `$(call cmd,checksrc)` to execute the command,
which will print the command depending on the value of `V` or `-s` option.
`log_print` will try to print the short message and the executed command;
`quiet_log_print` will just try to print the short message;
`silent_log_print` will not print anything.

The `delete-on-interrupt` is used to delete the target file if the build is interrupted.
It will trap the signals `HUP`, `INT`, `QUIT`, `TERM`, and `PIPE`,
and delete the target file if it exists when the signal is received.


From the
[Makefile](https://github.com/torvalds/linux/blob/master/Makefile)
of
[Linux Kernel](https://github.com/torvalds/linux),
we know that this file is the top-level Makefile for the Linux kernel build system.
It defines the build process, including the kernel version, architecture,
and various build options.

The Makefile uses recursive builds.



---





Call a source code checker (by default, "sparse") as part of the
C compilation.

Use 'make C=1' to enable checking of only re-compiled files.
Use 'make C=2' to enable checking of *all* source files, regardless
of whether they are re-compiled or not.

See the file "Documentation/dev-tools/sparse.rst" for more details,
including where to get the "sparse" utility.

`C=1` and `C=2`

---

`printk`

---

```c
/* Used in tsk->__state: */
#define TASK_RUNNING			0x00000000
#define TASK_INTERRUPTIBLE		0x00000001
#define TASK_UNINTERRUPTIBLE		0x00000002
#define __TASK_STOPPED			0x00000004
#define __TASK_TRACED			0x00000008
/* Used in tsk->exit_state: */
#define EXIT_DEAD			0x00000010
#define EXIT_ZOMBIE			0x00000020
#define EXIT_TRACE			(EXIT_ZOMBIE | EXIT_DEAD)
/* Used in tsk->__state again: */
#define TASK_PARKED			0x00000040
#define TASK_DEAD			0x00000080
#define TASK_WAKEKILL			0x00000100
#define TASK_WAKING			0x00000200
#define TASK_NOLOAD			0x00000400
#define TASK_NEW			0x00000800
#define TASK_RTLOCK_WAIT		0x00001000
#define TASK_FREEZABLE			0x00002000
#define __TASK_FREEZABLE_UNSAFE	       (0x00004000 * IS_ENABLED(CONFIG_LOCKDEP))
#define TASK_FROZEN			0x00008000
#define TASK_STATE_MAX			0x00010000

#define TASK_ANY			(TASK_STATE_MAX-1)
```
