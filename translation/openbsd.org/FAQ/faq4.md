# OpenBSD FAQ - 安装指南

## 安装过程总览

OpenBSD 安装程序使用一个特殊的 ramdisk 内核（`bsd.rd`），生成一个完全在内存中运行的实时环境。它包含了安装脚本和少量执行完整安装所需的工具。这些实用程序也可用于灾难恢复。

可以从许多不同的来源引导 ramdisk 内核：

- CD/DVD
- USB 驱动器
- 一个已经存在的分区
- 通过网络（[PXE](https://www.openbsd.org/faq/faq6.html#PXE) 或其他[网络启动选项](https://man.openbsd.org/diskless)）
- 软盘

并非每个[平台](https://www.openbsd.org/plat.html)都支持所有这些选项。

如果你有一个正在运行的 OpenBSD 系统，`bsd.rd` 是你重新安装或升级到更新版本所需的全部内容。为此，请[下载并验证](https://www.openbsd.org/faq/faq4.html#Download)新的 `bsd.rd`，将其放置在现有文件系统上，然后从中启动。引导 `bsd.rd` 的一般方法是通过平台上使用的任何方式将引导内核从 `/bsd` 更改为 `/bsd.rd`。

在 amd64 系统上从 `bsd.rd` 引导可以这样完成：

```shell
Using drive 0, partition 3.
Loading......
probing: pc0 com0 com1 mem[638K 1918M a20=on]
disk: hd0+ hd1+
>> OpenBSD/amd64 BOOT 3.33
boot> bsd.rd
```

这将从第一个识别的硬盘的第一个分区引导名为 `bsd.rd` 的内核。

如果你需要指定不同的驱动器或分区，只需在内核名称前加上其位置即可。以下示例将从第二个硬盘驱动器的第四个分区启动：

```shell
Using drive 0, partition 3.
Loading......
probing: pc0 com0 com1 mem[638K 1918M a20=on]
disk: hd0+ hd1+
>> OpenBSD/amd64 BOOT 3.33
boot> boot hd1d:/bsd.rd
```

OpenBSD 引导加载器（boot loaders）在特定架构的 [boot(8)](https://man.openbsd.org/boot.8) 手册页面中有记录。

## 安装前检查清单

在开始之前，您应该对自己想要的结果有所了解。一些值得事先考虑的事情：

- 主机名
- 已安装并可用的硬件：
  - 验证 OpenBSD 与你的硬件的兼容性。你可能需要查阅特定于平台的安装说明，尤其是当你使用非 x86 CPU 架构时。它们包含详细说明和任何可能的警告：</p>
  [[alpha](https://ftp.openbsd.org/pub/OpenBSD/7.0/alpha/INSTALL.alpha)] [[amd64](https://ftp.openbsd.org/pub/OpenBSD/7.0/amd64/INSTALL.amd64)] [[arm64](https://ftp.openbsd.org/pub/OpenBSD/7.0/arm64/INSTALL.arm64)] [[armv7](https://ftp.openbsd.org/pub/OpenBSD/7.0/armv7/INSTALL.armv7)] [[hppa](https://ftp.openbsd.org/pub/OpenBSD/7.0/hppa/INSTALL.hppa)] [[i386](https://ftp.openbsd.org/pub/OpenBSD/7.0/i386/INSTALL.i386)] [[landisk](https://ftp.openbsd.org/pub/OpenBSD/7.0/landisk/INSTALL.landisk)] [[luna88k](https://ftp.openbsd.org/pub/OpenBSD/7.0/luna88k/INSTALL.luna88k)] [[macppc](https://ftp.openbsd.org/pub/OpenBSD/7.0/macppc/INSTALL.macppc)] [[octeon](https://ftp.openbsd.org/pub/OpenBSD/7.0/octeon/INSTALL.octeon)] [[powerpc64](https://ftp.openbsd.org/pub/OpenBSD/7.0/powerpc64/INSTALL.powerpc64)] [[riscv64](https://ftp.openbsd.org/pub/OpenBSD/7.0/riscv64/INSTALL.riscv64)] [[sparc64](https://ftp.openbsd.org/pub/OpenBSD/7.0/sparc64/INSTALL.sparc64)]

  - 如果无线互联网是你接入网络唯一的选择，你的网卡是否需要[额外的固件](http://firmware.openbsd.org/firmware)？如果是这样，你需要手动将其下载到 USB 驱动器或类似设备，然后在安装 OpenBSD 后使用 [fw_update(1)](https://man.openbsd.org/fw_update) 启用它。
- 要使用的安装方法
- 所需的磁盘布局：
  - 现有数据是否需要保存至别处？
  - OpenBSD 会在这个系统上与另一个操作系统共存吗？如果是这样，每个系统将如何启动？你需要安装引导管理器吗？
  - 将整个磁盘用于安装 OpenBSD，还是要保留现有分区/操作系统（或未来划分的磁盘空间）？
  - 你希望如何对磁盘的 OpenBSD 部分进行子分区？
  - 你需要[磁盘加密](https://www.openbsd.org/faq/faq14.html#softraidFDE)吗？
-  网络设置（如果不使用 DHCP）：
 -  域名和 DNS 地址
 -  每个 NIC 的 IP 地址和子网掩码
 -  网关地址

## 下载 OpenBSD

| 文件名     | 下载方式 |
| ----------- | ----------- |
| install70.img | 可以写入 USB 闪存驱动器或类似设备的磁盘映像。包括[文件集（File Sets）](https://www.openbsd.org/faq/faq4.html#FilesNeeded)。</p>[amd64](https://cdn.openbsd.org/pub/OpenBSD/7.0/amd64/install70.img)、[arm64](https://cdn.openbsd.org/pub/OpenBSD/7.0/arm64/install70.img)、[i386](https://cdn.openbsd.org/pub/OpenBSD/7.0/i386/install70.img)、[octeon](https://cdn.openbsd.org/pub/OpenBSD/7.0/octeon/install70.img)、[powerpc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/powerpc64/install70.img)、[riscv64](https://cdn.openbsd.org/pub/OpenBSD/7.0/riscv64/install70.img)、[sparc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/sparc64/install70.img)|
| miniroot70.img | 同上，但不包括文件集。它们可以从 Internet 或本地磁盘下载。</p>[alpha](https://cdn.openbsd.org/pub/OpenBSD/7.0/alpha/miniroot70.img)、[amd64](https://cdn.openbsd.org/pub/OpenBSD/7.0/amd64/miniroot70.img)、[arm64](https://cdn.openbsd.org/pub/OpenBSD/7.0/arm64/miniroot70.img)、[armv7](https://cdn.openbsd.org/pub/OpenBSD/7.0/armv7/)、[i386](https://cdn.openbsd.org/pub/OpenBSD/7.0/i386/miniroot70.img)、[landisk](https://cdn.openbsd.org/pub/OpenBSD/7.0/landisk/miniroot70.img)、[luna88k](https://cdn.openbsd.org/pub/OpenBSD/7.0/luna88k/miniroot70.img)、[octeon](https://cdn.openbsd.org/pub/OpenBSD/7.0/octeon/miniroot70.img)、[powerpc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/powerpc64/miniroot70.img)、[riscv64](https://cdn.openbsd.org/pub/OpenBSD/7.0/riscv64/miniroot70.img)、[sparc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/sparc64/miniroot70.img)|
| install70.iso | 可用于创建安装 CD/DVD 的 ISO 9660 镜像文件。包括文件集。</p>[alpha](https://cdn.openbsd.org/pub/OpenBSD/7.0/alpha/install70.iso)、[amd64](https://cdn.openbsd.org/pub/OpenBSD/7.0/amd64/install70.iso)、[hppa](https://cdn.openbsd.org/pub/OpenBSD/7.0/hppa/install70.iso)、[i386](https://cdn.openbsd.org/pub/OpenBSD/7.0/i386/install70.iso)、[macppc](https://cdn.openbsd.org/pub/OpenBSD/7.0/macppc/install70.iso)、[powerpc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/powerpc64/install70.iso)、[sparc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/sparc64/install70.iso)|
| cd70.iso | 同上，但不包括文件集。</p>[alpha](https://cdn.openbsd.org/pub/OpenBSD/7.0/alpha/cd70.iso)、[amd64](https://cdn.openbsd.org/pub/OpenBSD/7.0/amd64/cd70.iso)、[hppa](https://cdn.openbsd.org/pub/OpenBSD/7.0/hppa/cd70.iso)、[i386](https://cdn.openbsd.org/pub/OpenBSD/7.0/i386/cd70.iso)、[macppc](https://cdn.openbsd.org/pub/OpenBSD/7.0/macppc/cd70.iso)、[sparc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/sparc64/cd70.iso)|
| floppy70.img | 支持一些缺少其他启动选项的旧机器。</p>[amd64](https://cdn.openbsd.org/pub/OpenBSD/7.0/amd64/floppy70.img)、[i386](https://cdn.openbsd.org/pub/OpenBSD/7.0/i386/floppy70.img)、[sparc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/sparc64/floppy70.img)|

你也可以从许多备用[镜像站点](https://www.openbsd.org/ftp.html)下载镜像文件。

包含校验和的 `SHA256` 文件可以在与安装文件相同的目录中找到。你可以使用 `sha256(1)` 命令确认下载的文件在传输过程中都没有被破坏。

```shell
$ sha256 -C SHA256 miniroot*.img
(SHA256) miniroot70.img: OK
```

或者，如果你使用带有 GNU coreutils 的操作系统：

```shell
$ sha256sum -c --ignore-missing SHA256
miniroot70.img: OK
```

但是，这只检查意外损坏。你可以使用 `signify(1)` 和 `SHA256.sig` 文件对下载的镜像文件进行加密验证。

```shell
$ signify -Cp /etc/signify/openbsd-70-base.pub -x SHA256.sig miniroot*.img
Signature Verified
miniroot70.img: OK
```

请注意，其他操作系统上的 signify 包可能不包含所需的公钥，或者它可能安装在其他位置。

`install70.iso` 和 `install70.img` 映像不包含 `SHA256.sig` 文件，因此安装程序会提示它无法检查包含集的签名：

```
Directory does not contain SHA256.sig. Continue without verification? [no]
```

这是因为安装程序验证它们毫无意义。如果有人要制作流氓安装映像，他们当然可以更改安装程序以说明文件是合法的。如果你事先验证了镜像的签名，则在该提示下回答 “Yes” ，表示是安全的。

## 创建安装介质

### 闪存驱动器

你可以通过连接目标设备并使用 [dd(1)](https://man.openbsd.org/dd) 复制镜像来创建可引导的 USB 闪存驱动器。

假定你使用 OpenBSD ，且设备被识别为 sd6：

```shell
# dd if=install*.img of=/dev/rsd6c bs=1m
```

请注意，使用的是**原始 I/O 设备**：`rsd6c` 而不是 `sd6c`。

其他平台上的操作细节会有所不同。如 GNU 版本的 `dd` 将需要 `bs=1M`（注意大写的 M）。 如果您使用不同的操作系统，请务必选择适当的设备名称：例如，Linux 上的 `/dev/sdX` 或 macOS 上的 `/dev/rdiskX`。

### CD-ROMs

你可以使用 [cdio(1)](https://man.openbsd.org/cdio) 在 OpenBSD 上创建可引导 CD-ROM。

```shell
# cdio tao cd*.iso
```

## 执行简单安装

如果你需要有关从你偏好的安装介质启动的说明，请查看你的机器的相关[平台页面](https://www.openbsd.org/plat.html)。

安装程序旨在以非常有用的默认配置安装 OpenBSD，并且需要最少的用户干预。事实上，你通常只需按 `Enter` 键即可获得良好安装的 OpenBSD，仅在输入 root 密码时需将手移到键盘的其余部分即可。

显示 `dmesg(8)` 后，你将看到第一个安装程序问题：

```shell
...
root on rd0a swap on rd0b dump on rd0b
erase ^?, werase ^W, kill ^U, intr ^C, status ^T

Welcome to the OpenBSD/amd64 7.0 installation program.
(I)nstall, (U)pgrade, (A)utoinstall or (S)hell?
```

选择 `(I)nstall` 并按照说明进行操作。

## 文件集

完整的 OpenBSD 安装分为多个文件集（File Sets）：

| 名称 | 用途 |
|----|----|
| `bsd` | 操作系统内核（必需的）|
| `bsd.mp ` | 多处理器内核（仅在某些平台上）|
| `bsd.rd ` | [ramdisk 内核](https://www.openbsd.org/faq/faq4.html#bsd.rd) |
| `base70.tgz` | 基础系统（必需的）|
| `comp70.tgz` | 编译器集合、头文件和库 |
| `man70.tgz` | 手册页 |
| `game70.tgz` | 文字游戏 |
| `xbase70.tgz` | X11 的基础库和实用程序（需要 `xshare70.tgz`）
| `xfont70.tgz` | X11 使用的字体 |
| `xserv70.tgz` | X11 的 X 服务器 |
| `xshare70.tgz` | X11 的手册页、本地化设置和头文件 |

建议新用户全部安装。

一些来自 `xbase70.tgz` 的库，如 freetype 或 fontconfig，可以在 X 之外由操作文本或图形的程序使用。此类程序通常需要来自 `xfont70.tgz` 或字体包的字体。为简单起见，开发人员决定不维护允许大多数非 X 端口运行的最小 `xbase70.tgz` 集。

### 安装后添加文件集

如果你选择在安装时跳过某些文件集，你可能稍后会意识到你确实确实需要它们。要安装额外的文件集，只需从你的根文件系统启动 [bsd.rd](https://www.openbsd.org/faq/faq4.html#bsd.rd) 并选择 `(U)pgrade`。当你进入文件集列表时，选择你需要的文件集。

## 磁盘分区

OpenBSD 可以安装在 512MB 的小空间中，但是使用这么小的设备是一个只适合有经验的高级用户的方案。在你有一定经验之前，建议使用 8GB 或更多磁盘空间。

与其他一些操作系统不同，OpenBSD 鼓励用户将他们的磁盘分成多个分区，而不是一两个大的分区。这样做的一些原因是：

- 安全性：OpenBSD 的一些默认安全功能依赖于文件系统[挂载选项](https://man.openbsd.org/mount#o)，例如 `nosuid`、`nodev`、`noexec` 或 `wxallowed`。
- 稳定性：如果用户或行为不端的程序具有写入权限，则他们可以用垃圾填满文件系统。 你所希望能在不同的文件系统上运行的关键程序不会因此被打断。
- [fsck(8)](https://man.openbsd.org/fsck)：你可以把从来不需要或很少需要写入的分区挂载为 `readonly`（只读），这将消除在崩溃或电源中断后进行文件系统检查的需要。

安装程序将根据你的硬盘大小创建分区计划。虽然这不是适合所有人的完美布局，但它为弄清楚你的需求提供了一个很好的起点。在做出关于自定义分区方案的决定之前，请阅读 disklabel 的[自动磁盘分配](https://man.openbsd.org/disklabel#AUTOMATIC_DISK_ALLOCATION)的默认值和 [hier(7)](https://man.openbsd.org/hier) 手册页。

- 由于一些[包](https://www.openbsd.org/faq/faq15.html)需要从 `wxallowed` 文件系统启动，建议划分一个单独的 `/usr/local` 分区。
- 当你需要升级时，非常小的分区会变得很麻烦。
- 一个独立 `/home` 分区是不错的主意。需要新版本的操作系统？保持你的 `/home` 分区不变，擦除并重新加载其他一切。
- 你可能还需要创建一个 [altroot 分区](https://www.openbsd.org/faq/faq14.html#altroot)来备份你的根文件系统。
- 暴露在互联网上的系统应该有一个单独的 `/var` 分区，甚至可能需要一个单独的 `/var/log` 分区。
- 从源代码编译某些 [ports](https://www.openbsd.org/faq/ports/index.html) 会在你的 `/usr` 和 `/tmp` 分区上占用大量的空间。

## 在安装后发送你的 dmesg

安装成功后，看看 `dmesg(8)` 命令的输出，看看是否有任何异常。如果一个设备显示为 `not configured`（未配置），这意味着它目前不被内核所支持。这可能会在将来通过发送 dmesg 而得到改善。引自</p>`/usr/src/etc/root/root.mail`：

```shell
If you wish to ensure that OpenBSD runs better on your machines, please do us
a favor (after you have your mail system configured!) and type something like:

# (dmesg; sysctl hw.sensors) | \
   mail -s "Sony VAIO 505R laptop, apm works OK" dmesg@openbsd.org

so that we can see what kinds of configurations people are running.  As shown,
including a bit of information about your machine in the subject or the body
can help us even further.  We will use this information to improve device driver
support in future releases.  (Please do this using the supplied GENERIC kernel,
not for a custom compiled kernel, unless you're unable to boot the GENERIC
kernel.  If you have a multi-processor machine, dmesg results of both GENERIC.MP
and GENERIC kernels are appreciated.)  The device driver information we get from
this helps us fix existing drivers. Thank you!
```

译文：

```shell
如果你希望确保 OpenBSD 在您的机器上运行得更好，请帮我们个忙（在你配置好邮件系统之后！）并键入以下内容：

# (dmesg; sysctl hw.sensors) | \
   mail -s "Sony VAIO 505R laptop, apm works OK" dmesg@openbsd.org

这样我们就可以看到人们正在运行什么样的配置。如图所示，在主题或正文中包括一点关于你的机器的信息可以帮助我们更进一步。我们将使用这些信息来改进未来版本中的设备驱动支持。(请使用提供的 GENERIC 内核来做这个，而不是自定义编译的内核，除非你无法启动 GENERIC 内核。如果你有一台多处理器的机器，请提供 GENERIC.MP 和 GENERIC 内核的 dmesg 结果）。我们从中得到的设备驱动信息有助于我们修复现有的驱动。谢谢你！
```

或者，将你的 dmesg 输出保存到文本文件并将其内容发送给我们：

```shell
$ (dmesg; sysctl hw.sensors) > ~/dmesg.txt
```

请配置你的电子邮件客户端以使用纯文本。特别地，不要使用 HTML 格式或强制换行。 请将 dmesg 放入邮件正文中，而不是作为附件发送。

## 自定义安装过程

### `site70.tgz` 文件集

OpenBSD 安装和升级脚本允许选择一个用户创建的名为 `site70.tgz` 的集合。与官方[文件集](https://www.openbsd.org/faq/faq4.html#FilesNeeded)一样，这是一个 [tar(1)](https://man.openbsd.org/tar) 存档，以 / 为父目录并使用 `-xzphf` 选项解压。它是最后安装的文件集，因此可用于补充和修改默认安装中的文件。此外，可以使用名为 `site70-$(hostname -s).tgz` 的主机名相关集。注意：如果你打算通过 HTTP(s) 提供文件集，请将 `site70.tgz` 放在你的源目录中并将其包含在你的 `index.txt` 中。 然后它将是安装时的一个选项。

### `install.site` 和 `upgrade.site` 脚本

如果 `site70.tgz` 文件集包含一个可执行文件 `/install.site`，安装程序会在新安装的系统根目录下使用 [chroot(8)](https://man.openbsd.org/chroot) 运行它。同样，升级脚本运行 `/upgrade.site`。后者可以放在系统的根目录下，然后重新启动进行升级。

用法示例：

- 设置系统时间。
- 在将新系统公开给世界其他地方之前，立即对其进行备份/存档。
- 在第一次启动后运行一组任意命令。如果 `install.site` 用于将任何此类命令附加到 [rc.firsttime(8)](https://man.openbsd.org/rc.firsttime) 文件（附加到此文件是必要的，因为安装程序本身可能会写入此文件），则会发生这种情况。在启动时，`rc.firsttime` 被执行一次然后被删除。

## 多重引导

多重引导是在一台计算机上拥有多个操作系统，并通过某种方式选择要引导的操作系统。 在开始之前，你可能希望熟悉 [OpenBSD 引导过程](https://www.openbsd.org/faq/faq14.html#BootAmd64)。[fdisk(8)](https://man.openbsd.org/fdisk) 的简要介绍在[使用 OpenBSD 的 fdisk](https://www.openbsd.org/faq/faq14.html#fdisk) 部分。

如果你要将 OpenBSD 添加到现有系统，您可能需要在安装 OpenBSD 之前创建一些空闲空间。除了现有系统的本机工具外，[gparted](https://gparted.org/) 还可用于删除或调整现有分区的大小。最好使用四个主要 MBR 分区之一来引导 OpenBSD。扩展分区可能不起作用。

据报道，[rEFInd](https://www.rodsbooks.com/refind/) 通常有效；[GRUB](https://www.gnu.org/software/grub/) 通常会失败。无论哪种情况，你都完全靠自己决定如何操作。

### Windows

启动配置数据 (Boot Configuration Data, BCD) 存储允许通过 `bcdedit` 启动多个版本的 Windows。你可以在[这篇文章](https://technet.microsoft.com/en-us/library/cc721886%28WS.10%29.aspx)中找到一个很好的介绍。如果你想要一个 GUI 替代品，你可能想需要 [EasyBCD](https://neosmart.net/EasyBCD/)。

你将需要一份 OpenBSD 安装的[分区引导记录 (PBR)](https://www.openbsd.org/faq/faq14.html#BootAmd64) 的副本。你可以使用类似于以下的过程将其复制到文件中：

```shell
# dd if=/dev/rsd0a of=openbsd.pbr bs=512 count=1
```

其中 `sd0a` 是你的启动设备，你需要将 `openbsd.pbr` 文件放到你的 Windows 系统分区。

一旦 OpenBSD 的 PBR 被复制到 Windows 系统分区，你需要一个具有管理权限的终端来运行以下命令：

```shell
C:\Windows\system32> bcdedit /create /d "OpenBSD/i386" /application bootsector
The entry {0154a872-3d41-11de-bd67-a7060316bbb1} was successfully created.
C:\Windows\system32> bcdedit /set {0154a872-3d41-11de-bd67-a7060316bbb1} device boot
The operation completed successfully.
C:\Windows\system32> bcdedit /set {0154a872-3d41-11de-bd67-a7060316bbb1} path \openbsd.pbr
The operation completed successfully.
C:\Windows\system32> bcdedit /set {0154a872-3d41-11de-bd67-a7060316bbb1} device partition=c:
The operation completed successfully.
C:\Windows\system32> bcdedit /displayorder {0154a872-3d41-11de-bd67-7060316bbb1} /addlast
The operation completed successfully.
```

请注意，OpenBSD 希望计算机的实时时钟设置为协调世界时 (UTC)。 有关更多信息，请参阅[此部分](https://www.openbsd.org/faq/faq10.html#TimeZone)。