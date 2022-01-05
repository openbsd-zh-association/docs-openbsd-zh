#  OpenBSD FAQ - X 窗口系统

## 简介

X Window System（X 窗口系统，有时简称为 “X” 或 “X11”）是为 OpenBSD 和其他类 Unix 系统提供图形服务的环境。X 本身提供的服务非常少，因为还必须有一个窗口管理器来呈现用户界面。OpenBSD 提供了 [cwm(1)](https://man.openbsd.org/cwm)、[fvwm(1)](https://man.openbsd.org/fvwm) 和 [twm(1)](https://man.openbsd.org/twm) 窗口管理器，尽管许多其他的窗口管理器也可以作为软件包使用。

在没有任何图形支持的系统上运行 X 客户端是可能的。例如，可以在 ARM 系统上运行一个应用程序（X 客户端），将其输出显示在 amd64 的图形显示器（X 服务器）上。由于 X 是一个定义明确的跨平台协议，甚至可以让运行在 Linux 机器上的 X 应用程序使用 OpenBSD 机器进行显示。客户端和服务器也可以运行在同一台机器上，在本节的大部分内容中，这将是一个假设。

## 配置

对于最常见平台上的大多数硬件，X 根本不需要配置。

手动配置 X 的细节在不同的平台上有很大的不同。

## 启动

推荐的运行 X 的方法是使用 [xenodm(1)](https://man.openbsd.org/xenodm) 显示管理器。它比传统的 [startx(1)](https://man.openbsd.org/startx) 命令提供了一些重要的安全优势。

如果 [xenodm(1)](https://man.openbsd.org/xenodm) 在安装时没有启用，可以像其他系统守护程序一样在以后启用。

```
# rcctl enable xenodm
# rcctl start xenodm
```

在某些平台上，你需要禁用控制台 [getty(8)](https://man.openbsd.org/getty) 来使用它。这在 amd64、i386 或 macppc 上是不需要的。

## 自定义

OpenBSD 的默认 X 环境功能齐全，但你可能希望对其进行自定义。当一个 X 会话被启动时，用户主目录下的 shell 脚本可以用来启动任意多的程序。这些脚本中的大多数程序应该在后台运行，但最后一个程序（通常是窗口管理器）应该在前台运行。当窗口管理器退出时，脚本也将退出，X 将返回到 [xenodm(1)](https://man.openbsd.org/xenodm) 的登录提示。

在用户从 [xenodm(1)](https://man.openbsd.org/xenodm) 登录后，`/etc/X11/xenodm/Xsession` 脚本会检查是否有一个 `$HOME/.xsession` 脚本。在最简单的情况下，用户的 `~/.xsession` 脚本将只包含一行，指定要启动的首选窗口管理器。然而，它可以包含任何数量的其他命令:

```
export ENV=$HOME/.kshrc
xsetroot -solid grey &
xterm -bg black -fg white +sb &
cwm
```

注意，窗口管理器 [cwm(1)](https://man.openbsd.org/cwm) 没有在后台运行。这意味着 X 会一直运行，直到它退出。