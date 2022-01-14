#  OpenBSD - 跟随 -current 和使用快照

活跃的 OpenBSD 开发被称为 [-current](https://www.openbsd.org/faq/faq5.html#Flavors) 分支。 这些源代码经常被编译成称为快照（snapshots）的版本。

我们有时会在此分支中推送激进的更改，并且在编译最新代码或从以前的时间快照点升级时可能会出现复杂情况。本页解释了克服这些障碍的一些步骤。在使用 -current 和以下说明之前，请确保你已阅读并了解[如何从源代码构建系统](https://www.openbsd.org/faq/faq5.html)。

一般来说，使用快照要容易得多，因为开发人员已经为你解决了很多麻烦。

你应该**始终**使用快照作为运行 `-current` 的起点。此过程通常包括运行带有 `-s` 标志的 [sysupgrade(8)](https://man.openbsd.org/sysupgrade)。或者，从首选镜像的 `/snapshots/` 目录下载（并验证）合适的 `bsd.rd` 文件，并从该文件引导，然后在提示符处选择 `(U)pgrade`。任何已安装的软件包都应在引导至新系统后进行升级。

不鼓励除专家之外的所有人通过编译自己的源代码升级到 `-current`，因为困难的构建时交叉点经常发生，并且我们不会提供任何帮助。如果出现故障，请使用快照进行恢复。

大多数这些更改都必须以 root 身份执行。

## 2021/11/01 - sndio(7) 描述符格式更改

如果你将音频设备描述符（descriptors）显式传递给程序（通过程序选项或通过 `AUDIODEVICE` 环境变量），则可能需要按如下方式更新描述符。否则，请跳过本节。

[sndio(8)](https://man.openbsd.org/sndiod) 暴露的音频设备不再绑定到物理音频设备，因此 [sndio(7)](https://man.openbsd.org/sndio) 描述符的物理音频设备编号组件被删除。例如，如果服务器以以下方式启动：

```
# sndiod -f rsnd/0 -s foo -f rsnd/1 -s bar
```

那么程序将需要使用 “`snd/foo`” 和 “`snd/bar`”（而不是 “`snd/0.foo`” 和 “`snd/1.bar`”）。

默认情况下，程序将尝试使用 “`snd/default`”（而不是 “`snd/0`”）。除非使用 `-s default` 选项，否则 “`snd/default`” 会自动创建并附加到命令行中指定的第一个物理音频设备（即第一个 `-f` 或 `-F` 选项）。如果需要，可以在运行时使用 [sndioctl(1)](https://man.openbsd.org/sndioctl) 公开的 `server.device` 控件更改此设置。

对于服务器管理的每个物理设备（即每个 `-f` 或 `-F` 选项），服务器还公开绑定到相同编号的物理设备的 “`snd/<number>`” 描述符。因此，“`snd/0`”、“`snd/1`” ……等仍然可以用来指代特定的物理设备。

## 2021/11/03 - xterm(1) 默认禁用鼠标支持

[xterm(1)](https://man.openbsd.org/xterm) 中默认禁用鼠标支持。可以通过将以下内容添加到 .Xresources 来重新启用它：

```
xterm*allowMouseOps: true
```