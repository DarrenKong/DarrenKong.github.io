---
title: Mac PATH
date: 2024-01-26 15:39:53
tags:
    - PATH
categories:
    - Mac
---
在 Mac 上工作, 有时候会卸载掉某个 App，可能是软件卸载姿势不对，需要手动排查 PATH 环境变量。
首先, 我们先看下当前的 PATH 变量:
```zsh
echo $PATH
/Users/darren/.rbenv/shims:/opt/theos/bin:/Users/darren/flutter/bin:/usr/local/opt/tcl-tk/bin:/usr/local/opt/icu4c/sbin:/usr/local/opt/icu4c/bin:/usr/local/opt/sqlite/bin:/usr/local/opt/python@3.8/bin:/usr/local/opt/libxml2/bin:/usr/local/opt/openssl@1.1/bin:/usr/local/opt/ruby/bin:/usr/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin:/Library/Apple/usr/bin:/Applications/Wireshark.app/Contents/MacOS

``` 

咦，出现了 Wireshark.app，这个 App 已经删除很久了，怎么环境变量还存在。

这里面有好多路径, 那么我们好奇系统启动的过程中, 最初的 PATH 是什么呢?

- MAC 上最初的 PATH 是从 /etc/paths 文件读的:
```zsh
cat /etc/paths
/usr/local/bin
/usr/bin
/bin
/usr/sbin
/sbin

```

- 然后, 它会把 /etc/path.d 里面的路径逐个添加到后面, 比如我们可以看到我的 Wireshark 路径在上面的 PATH 里:
```zsh
ls -l /etc/paths.d/
total 24
-rw-r--r--  1 root  wheel  23 Sep 27  2022 100-rvictl
-rw-r--r--  1 root  wheel  18 Mar 20  2020 go
-rw-r--r--  1 root  wheel  43 Oct 19  2021 Wireshark

cat /etc/paths.d/Wireshark
/Applications/Wireshark.app/Contents/MacOS
```

- 因为目前最近几个版本的 Mac 系统使用的是 Zsh 作为默认 shell，在个人的 HOME 目录, 还有 `~/.zshrc` 文件会影响 PATH 的值。
```zsh
# 可以通过下面这个命令查看使用说明
man zsh

...
STARTUP/SHUTDOWN FILES
       Commands are first read from /etc/zshenv; this cannot be overridden.  Subsequent behaviour is modified by the RCS and GLOBAL_RCS options; the former affects all startup files, while the second only
       affects global startup files (those shown here with an path starting with a /).  If one of the options is unset at any point, any subsequent startup file(s) of the corresponding type will not be read.
       It is also possible for a file in $ZDOTDIR to re-enable GLOBAL_RCS. Both RCS and GLOBAL_RCS are set by default.

       Commands are then read from $ZDOTDIR/.zshenv.  If the shell is a login shell, commands are read from /etc/zprofile and then $ZDOTDIR/.zprofile.  Then, if the shell is interactive, commands are read from
       /etc/zshrc and then $ZDOTDIR/.zshrc.  Finally, if the shell is a login shell, /etc/zlogin and $ZDOTDIR/.zlogin are read.

       When a login shell exits, the files $ZDOTDIR/.zlogout and then /etc/zlogout are read.  This happens with either an explicit exit via the exit or logout commands, or an implicit exit by reading
       end-of-file from the terminal.  However, if the shell terminates due to exec'ing another process, the logout files are not read.  These are also affected by the RCS and GLOBAL_RCS options.  Note also
       that the RCS option affects the saving of history files, i.e. if RCS is unset when the shell exits, no history file will be saved.

       If ZDOTDIR is unset, HOME is used instead.  Files listed above as being in /etc may be in another directory, depending on the installation.

       As /etc/zshenv is run for all instances of zsh, it is important that it be kept as small as possible.  In particular, it is a good idea to put code that does not need to be run for every single shell
       behind a test of the form `if [[ -o rcs ]]; then ...' so that it will not be executed when zsh is invoked with the `-f' option.

       Any of these files may be pre-compiled with the zcompile builtin command (see zshbuiltins(1)).  If a compiled file exists (named for the original file plus the .zwc extension) and it is newer than the
       original file, the compiled file will be used instead.

FILES
       $ZDOTDIR/.zshenv
       $ZDOTDIR/.zprofile
       $ZDOTDIR/.zshrc
       $ZDOTDIR/.zlogin
       $ZDOTDIR/.zlogout
       ${TMPPREFIX}*   (default is /tmp/zsh*)
       /etc/zshenv
       /etc/zprofile
       /etc/zshrc
       /etc/zlogin
       /etc/zlogout    (installation-specific - /etc is the default)
...

```