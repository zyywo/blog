<!--markdown-->
查看当前的PATH
>echo $PATH

# Debian10

### 改变使X11窗口系统环境的PATH

在55xfce4-session文件前创建一个文件如：/etc/X11/Xsession.d/49add-user-path    

```
/etc/X11/Xsession.d$ cat 49add-user-path 
[ -d "$HOME/bin" ] && PATH="$HOME/bin:$PATH"
```

### 改变其他的环境的PATH

When bash is invoked as an interactive login shell, 
or as a non-interactive shell with the `--login` option,
it first reads and executes commands from the file `/etc/profile`,
if that file exists. After reading that file,
it looks for `~/.bash_profile`, `~/.bash_login`, and `~/.profile`,
in that order, and reads and executes commands from the first one that
exists and is readable. The `--noprofile` option may be used when the 
shell is started to inhibit this behavior.

When an interactive shell that is not a login shell is started, bash reads
and executes commands from `/etc/bash.bashrc` and `~/.bashrc`, if these files exist.
This may be inhibited by using the `--norc` option. The `--rcfile` file option will
force bash to read and execute commands from file instead of `/etc/bash.bashrc` and `~/.bashrc`.
	