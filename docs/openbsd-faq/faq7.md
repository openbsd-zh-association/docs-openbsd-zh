# OpenBSD FAQ - 键盘和显示控制

## 在控制台上工作

### 重新映射键盘

[kbd(8)](https://man.openbsd.org/kbd) 实用程序可用于更改键盘编码（keyboard encoding）。大多数键盘选项都可以使用 [wsconsctl(8)](https://man.openbsd.org/wsconsctl) 实用程序进行控制。例如，要使用 [wsconsctl(8)](https://man.openbsd.org/wsconsctl) 更改键映射，可以执行以下操作：

```
# wsconsctl keyboard.encoding=uk
```

重新映射 [Caps Lock] 为左控制键 [Control L]：

```
# wsconsctl keyboard.map+="keysym Caps_Lock = Control_L"
```

要永久更改，请使用 [wsconsctl.conf(5)](https://man.openbsd.org/wsconsctl.conf) 文件。

### 控制台鼠标支持

对于 [alpha](https://www.openbsd.org/alpha.html)、[amd64](https://www.openbsd.org/amd64.html) 和 [i386](https://www.openbsd.org/i386.html) 平台，OpenBSD 提供 [wsmoused(8)](https://man.openbsd.org/wsmoused)。它可以使用 [rcctl(8)](https://man.openbsd.org/rcctl) 实用程序启用，在 [rc 脚本](https://www.openbsd.org/faq/faq10.html#rc)的 FAQ 中也有描述。

### 切换控制台

在大多数 alpha、macppc、amd64 和 i386 系统上，OpenBSD 默认提供六个虚拟终端：`/dev/ttyC0` 到 `/dev/ttyC5`。你可以使用 `[CTRL]+[ALT]` 和 `[F1]` 到 `[F6]` 在它们之间切换。虚拟终端 `ttyC4` 保留供 [X 窗口系统](https://www.openbsd.org/faq/faq11.html)使用。

## VGA 硬件上的控制台

注意：本节讨论 [vga(4)](https://man.openbsd.org/vga) 驱动程序的功能。并非所有视频卡都支持它们。以下说明**不适用于使用 [drm(4)](https://man.openbsd.org/drm) 驱动程序的现代图形硬件。**

### 回滚缓冲区

在一些平台和硬件组合上，OpenBSD 提供了一个控制台回滚缓冲区。这使你可以查看已经滚动过屏幕的信息。要在缓冲区中上下移动，请使用 `[SHIFT]+[PGUP]` 和 `[SHIFT]+[PGDN]`。你可以向上移动和查看的页面数为 8。切换控制台将清除回滚缓冲区。由于空间限制，安装内核没有这个功能。

### 添加更多虚拟控制台

如果您希望拥有多于默认数量的虚拟控制台，请使用 [wsconscfg(8)](https://man.openbsd.org/wsconscfg) 命令为 `ttyC6`、`ttyC7` 及更多的虚拟控制台创建屏幕。例如：

```
# wsconscfg -t 80x25 6       # this will not work on systems using drm(4)
```

这将为 `ttyC6` 创建一个虚拟终端，通过 `[CTRL]+[ALT]+[F7]` 访问。要在新创建的虚拟控制台上获得 `login:` 提示，你需要在 [ttys(5)](https://man.openbsd.org/ttys) 中将其设置为 `on`，然后重新启动或使用 [kill(1)](https://man.openbsd.org/kill) 向 [init(8)](https://man.openbsd.org/init) 发送 HUP 信号。如果下次启动计算机时需要额外的屏幕，请将此命令添加到 [rc.local(8)](https://man.openbsd.org/rc.local)。

### 更改控制台字体和分辨率

许多 alpha、amd64 和 i386 上的 VGA 视频卡能够显示 50 行的更高文本分辨率，而不是通常的 25 行。标准的 80x25 文本屏幕使用 8x16 像素字体。为了使行数加倍，我们首先使用 [wsfontload(8)](https://man.openbsd.org/wsfontload) 加载一个 8x8 像素的字体。然后我们使用 [wsconscfg(8)](https://man.openbsd.org/wsconscfg) 删除并重新创建一个具有所需屏幕分辨率的虚拟控制台。这可以通过在 [rc.local(8)](https://man.openbsd.org/rc.local) 脚本的末尾添加以下命令在启动时自动完成：

```
wsfontload -h 8 -e ibm /usr/share/misc/pcvtfonts/vt220l.808	# load 8x8 font
wsconscfg -dF 5			# delete screen 5 accessed by [CTRL]+[ALT]+[F6]
wsconscfg -t 80x50 5		# add screen 5 with 50 lines of 80 characters
```

如果你希望修改其他屏幕，只需为你希望以 80x50 分辨率运行的任何屏幕重复以上删除和添加屏幕步骤。你无法更改通过 `[CTRL]+[ALT]+[F1]` 访问的主控制台设备 `ttyC0` 的分辨率。请避免更改 X 用作图形屏幕的屏幕 4。

## 清空非活动控制台

如果你希望在一段时间不活动后不使用 X 清空控制台，请修改以下 [wscons(4)](https://man.openbsd.org/wscons) 变量：

- `display.screen_off` 确定以毫秒为单位的消隐时间（the blanking time）。
- `display.kbdact` 如果设置为 `on`，键盘活动将取消空白屏幕。
- `display.msact` 如果设置为 `on`，[控制台鼠标](https://www.openbsd.org/faq/faq7.html#ConsoleMouse)活动将取消空白屏幕。
- `display.outact` 如果设置为 `on`，屏幕输出将取消空白屏幕。
- `display.vblank` 如果设置为 `on` 将禁用垂直同步脉冲。这将导致许多显示器进入节能模式。

例如：

```
# wsconsctl display.screen_off=60000
display.screen_off -> 60000
```

通过编辑 [wsconsctl.conf(5)](https://man.openbsd.org/wsconsctl.conf) 永久设置它们。当 `display.kbdact` 或 `display.outact` 设置为 `on` 时，消隐器被激活。请注意，这两者之一必须关闭。

## 配置串口控制台

OpenBSD 在大多数平台上都支持串口控制台，但是不同的平台之间的细节差异很大。除了允许用户登录外，它们还可用于记录控制台输出。

在 OpenBSD 系统上获得功能齐全的串口控制台有两个部分：

- 启用串口作为交互终端，以便用户在运行多用户时可以登录。
- 配置 OpenBSD 以使用你的串口端口作为状态和单用户模式的控制台。这部分非常依赖平台特定的配置。

### 更改 `/etc/ttys` 以获取登录提示

终端会话由 [ttys(5)](https://man.openbsd.org/ttys) 文件控制。在 OpenBSD 给你一个 `login:` 提示之前，它必须在 `/etc/ttys` 中启用。默认情况下，串行终端在通常连接有键盘和屏幕的平台上是禁用的。我们将使用 `amd64` 平台作为示例。在这种情况下，你必须编辑以下行：

```
tty00   "/usr/libexec/getty std.9600"   unknown off
```

修改成如下文本：

```
tty00   "/usr/libexec/getty std.9600"   vt220   on secure
```

在这里，`tty00` 是我们用作控制台的串行端口，而 `vt220` 是与你的终端匹配的 [termcap(5)](https://man.openbsd.org/termcap.5) 条目。其他可能的选项可能包括 `vt100`、`xterm` 等。`on` 通过为该串行端口激活 [getty(8)](https://man.openbsd.org/getty) 来启用登录提示。`secure ` 允许在此控制台上进行 `root` 登录。`9600` 是终端波特率。

请注意，你可以在不执行此步骤的情况下使用串行控制台进行安装，因为系统在单用户模式下运行，而不是使用 `getty` 进行登录。

### amd64 和 i386

要将启动过程配置为使用串行端口作为控制台，你的 [boot.conf(5)](https://man.openbsd.org/amd64/boot.conf) 文件应包含以下行：

```
set tty com0
```

请将这个文件放在你的启动驱动器上，它也放在你的安装介质中。如果你需要 9600bps 以外的波特率，请使用 `stty` 选项。

某些系统可能能够在机器中没有视频卡的情况下运行，但肯定不是所有系统——许多人认为这是一种错误情况。其他人能够通过配置选项将所有 BIOS 键盘和屏幕活动重定向到串行端口，因此可以完全通过串行端口维护机器。你的结果可能会有所不同。使用此功能时，某些 BIOS 实现可能会阻止引导加载程序看到串行端口，因此不会告诉内核使用它。可能有一个 BIOS 选项 “Continue Console Redirection after POST.”。这应该设置为 `OFF`，以便引导加载程序和内核可以处理他们自己的控制台。

要在多用户模式下使用机器，你需要按[上述说明](https://www.openbsd.org/faq/faq7.html#SerContty)编辑 `/etc/ttys`。

### sparc64

这些机器设计为可通过串行控制台进行完全维护。只需从机器上取下键盘，系统就会串行运行。

在某些系统上，串行端口标记为 `ttya`、`ttyb`、`ttyh0` 或 `ttyh1`。在多用户模式下使用串行控制台不需要对 `/etc/ttys` 进行任何更改。

一些 sparc64 系统将控制台端口的 BREAK 信号解释为与 STOP-A 命令相同。这将使系统回到 Forth 提示符，并在此时停止任何应用程序和操作系统。这在需要时是很方便的，但不幸的是，一些串行终端在断电时和一些 RS-232 开关设备会发送一些被计算机解释为中断信号的东西，使机器停止工作。请在你进入生产之前进行测试。

如果您连接了键盘和显示器，您仍然可以通过在 `ok` 提示符下使用以下命令来强制使用串行控制台：

```
ok setenv input-device ttya
ok setenv output-device ttya
ok reset
```

如果 `ttyC0` 在 `/etc/ttys` 中处于活动状态，[如上所述](https://www.openbsd.org/faq/faq7.html#SerContty)，你可以在 X 中使用键盘和监视器。

### macppc

macppc 机器通过 OpenFirmware 配置为串行控制台。使用命令：

```
ok setenv output-device scca
ok setenv input-device scca
ok reset-all
```

将您的串行控制台设置为 57600bps，8N1。

不幸的是，在大多数 MacPPC 上不能直接使用串行控制台。虽然这些机器中的大多数都有串行硬件，但在机器外部无法访问。幸运的是，一些公司为几种 Macintosh 型号提供了附加设备，这将使该端口可用作串行控制台。

你必须将 `/etc/ttys` 中的 `tty00` 更改为 `on` 并将速度设置为 `57600` 而不是默认的 `9600`，如[上面](https://www.openbsd.org/faq/faq7.html#SerContty)在单用户模式中详述的那样，然后才能启动多用户并让串行控制台正常工作。

### 尝试使用我的 tty 设备时出现输入/输出错误

对于从 OpenBSD 系统发起的连接，您需要使用 `/dev/cuaXX`。`/dev/ttyXX` 设备仅用于终端或拨入使用。有关更多详细信息，请参阅 [cua(4)](https://man.openbsd.org/cua) 手册。