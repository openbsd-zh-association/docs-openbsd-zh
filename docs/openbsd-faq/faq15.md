# OpenBSD FAQ - 软件包管理

## 介绍

在 OpenBSD 系统上可能有许多应用程序需要使用。为了使这个软件更易于安装和管理，它被移植（port）到 OpenBSD 并打包。软件包系统的目的是跟踪安装了哪些软件，以便可以轻松更新或删除它。借助该系统的帮助，在几分钟内可以获取并安装大量软件包，并将所有内容放在正确的位置。

Ports collections 并没有经过与 OpenBSD 基本系统相同的全面安全审计。尽管我们努力保持软件包的高品质，但我们只是没有足够的资源来确保相同水平的稳健性和安全性。

OpenBSD ports 团队认为软件包（package）才是他们移植工作的目标， 而不是 port 本身。一般来说，我们建议你使用软件包，而不是从 ports 构建一个应用程序。

借助多个实用程序，可以轻松管理软件包：

- [pkg_add(1)](https://man.openbsd.org/pkg_add) - 用于安装和更新软件包
- [pkg_check(8)](https://man.openbsd.org/pkg_check) - 用于检查已安装软件包的一致性
- [pkg_delete(1)](https://man.openbsd.org/pkg_delete) - 用于删除已安装的软件包
- [pkg_info(1)](https://man.openbsd.org/pkg_info) - 用于显示有关包的信息

为了正常运行应用程序 X，可能需要安装其他应用程序 Y 和 Z。 应用程序 X 被称为依赖于这些其他应用程序，这就是为什么 Y 和 Z 被称为 X 的依赖关系。反过来，Y 可能需要其他应用程序 P 和 Q，而 Z 可能需要应用程序 R 才能正常运行。这样，就形成了一个完整的依赖树。

软件包看起来像简单的 `.tgz` 包。基本上它们就是这样，但有一个关键的区别：它们包含一些额外的打包信息。这些信息可被 `pkg_add(1)` 用于多种用途：

- 差异性检查：软件包是否已经安装，或者是否与其他已安装的包或文件名冲突？
- 在继续安装包之前，系统上不存在的依赖项会自动获取并安装。
- 有关包的信息记录在默认情况下位于 `/var/db/pkg` 的中心存储库中。除其他外，这将防止在包本身被删除之前删除包的依赖项。这有助于确保应用程序不会被粗心的用户意外破坏。

## 选择镜像

[pkg_add(1)](https://man.openbsd.org/pkg_add) 将在两个位置查找软件包：[installurl(5)](https://man.openbsd.org/installurl) 文件 (`/etc/installurl`) 或 `PKG_PATH` 环境变量。前者是首选方法，在安装系统时被配置为默认方案。

如果需要使用多个镜像，`PKG_PATH` 允许通过冒号分隔的列表执行此操作：

```shell
# export PKG_PATH=scp://user@company-build-server/usr/ports/packages/%a/all:https://trusted-public-server/%m:installpath
```

虽然默认设置对大多数人来说应该适用，但你可以在[镜像页面](https://www.openbsd.org/ftp.html)上找到备用位置列表。

## 查询软件包

大量预编译包可用于最常见的计算机架构。

要搜索任何给定的包名称，请使用 [pkg_info(1)](https://man.openbsd.org/pkg_info) 的 `-Q` 标志（flag）。

```shell
$ pkg_info -Q unzip
lunzip-1.8
unzip-6.0p9
unzip-6.0p9-iconv
```

查找你要查找的内容的另一种方法是使用 `pkglocate` 命令，该命令可从 `pkglocatedb` 包中获得。

```shell
$ pkglocate mutool
mupdf-1.11p1-js:textproc/mupdf,js:/usr/local/bin/mutool
mupdf-1.11p1-js:textproc/mupdf,js:/usr/local/man/man1/mutool.1
mupdf-1.11p1:textproc/mupdf:/usr/local/bin/mutool
mupdf-1.11p1:textproc/mupdf:/usr/local/man/man1/mutool.1
```

如果你正在寻找特定的文件名，它可用于查找包含该文件的包。

你会注意到某些软件包有几种不同的种类。这些被称为 **flavors**。[Ports 常见问题解答](https://www.openbsd.org/faq/ports/ports.html#PortsFlavors)详细解释了 flavors，但这基本上意味着它们配置了不同的选项集。例如，一个包可能有可选的数据库支持、对没有 X11 的系统的支持等。一些包也被分成可以单独安装的子包（subpackages）。

## 安装软件包

[pkg_add(1)](https://man.openbsd.org/pkg_add) 实用程序用于安装软件包。如果存在多种版本的软件包，系统将提示你选择要安装的版本。

```shell
# pkg_add rsync
Ambiguous: choose package for rsync
a       0: <None>
        1: rsync-3.1.2p0
        2: rsync-3.1.2p0-iconv
Your choice:
```

在这里，如果你需要标准包，请选择 **1**，如果需要 iconv 支持，请选择 **2**。 你还可以使用 `pkg_add rsync--`（对于默认值）或 `pkg_add rsync--iconv`（对于 iconv 版本）直接在命令行上选择 flavor。

你可以在一行中指定多个包名，然后一次性安装所有包及其依赖项。你还可以指定包的绝对位置，无论是本地文件还是远程 URL。 支持的 URL 前缀是 http、https、ftp 和 scp。

对于某些软件包，将提供有关应用程序配置或使用的重要附加信息。

```shell
# pkg_add jove
jove-4.16.0.73p0: ok
--- +jove-4.16.0.73p0 -------------------
See /usr/local/share/jove/README about changes to /etc/rc or
/etc/rc.local so that the system recovers jove files
on reboot after a system crash
```

此外，某些软件包在位于 `/usr/local/share/doc/pkg-readmes` 的文件中提供配置和其他信息。

为了你的安全，如果你正在安装之前安装并删除的软件包，则不会覆盖已修改的配置文件。更新包时也是如此。

有时你可能会遇到类似以下示例中的错误：

```shell
# pkg_add xv
xv-3.10ap4:jpeg-6bp3: ok
xv-3.10ap4:png-1.2.14p0: ok
xv-3.10ap4:tiff-3.8.2p0: ok
Can't install xv-3.10ap15 because of libraries
|library X11.16.1 not found
| not found anywhere
Direct dependencies for xv-3.10ap15 resolve to png-1.6.31 jasper-1.900.1p5 tiff-4.0.8p1 jpeg-1.5.1p0v0
Full dependency tree is png-1.6.31 tiff-4.0.8p1 jasper-1.900.1p5 jpeg-1.5.1p0v0
```

软件包中捆绑的打包信息包括有关该包希望安装的共享库的信息。如果找不到所需的库之一，则不会安装该软件包，因为它无论如何都不会工作。

有几件事要检查：

- 你的系统可能不完整：你没有安装含有待安装的软件包所需的依赖库的文件集。
- 你的系统（或软件包）可能已过时：你有所需库的旧版本。确保基本系统和任何已安装的软件包都是最新的。
- 如果你正在运行 `-current`，基础系统和包快照可能会稍微不同步。你可以等待镜像站同步后并重试。

## 升级软件包

可以使用 [pkg_add(1)](https://man.openbsd.org/pkg_add) 更新已安装的软件包，如下所示：

```shell
# pkg_add -u
```

这将尝试更新所有已安装的软件包，包括它们的依赖项。

## 移除软件包

要删除包，只需使用 [pkg_delete(1)](https://man.openbsd.org/pkg_delete)：

```shell
# pkg_delete screen
```

同样，已修改的配置文件不会被删除。

不再需要的依赖项之后可以使用 `-a` 标志删除：

```shell
# pkg_delete -a
```

## 在另一台机器上复制已安装的软件包

使用与旧机器相同的一组软件包安装新的 OpenBSD 系统是一个相当常见的用例。 [pkg_info(1)](https://man.openbsd.org/pkg_info) 的 `-mz` 标志将产生适当的结果以使此任务更容易。

- `-m` 标志仅选择手动安装的软件包。不记录依赖项，因为它们是自动拉入的。
- `-z` 标志从输出中排除版本信息。只显示 flavor 和分支，确保未来的软件包安装将选择适当的版本。

例如：

```shell
$ pkg_info -mz | tee list
abcde--
mpv--
python--%3.6
vim--no_x11
```

将 “list” 文件复制到另一台机器上并运行：

```shell
# pkg_add -l list
```

每个包规范都有一个附加到其名称的 flavor（`-- ` 作为默认值），并且在多个版本中共存的包也具有分支信息。在这种情况下，后续的 [pkg_add(1)](https://man.openbsd.org/pkg_add) 命令将选择 `3.6` 版本分支的当前 python 包。

## 不完整的软件包安装或删除

在一些奇怪的情况下，由于与其他文件冲突，你可能会发现一个包没有完全添加。不完整的安装通常在包名前加上 “partial-” 标记。 例如，当你在安装过程中碰巧按下 CTRL+C 终止安装时，就会发生这种情况。安装可以稍后完成，“partial-*” 包会消失，也可以用 [pkg_delete(8)](https://man.openbsd.org/pkg_delete) 删除。

更严重的系统故障，例如文件系统问题，可能会导致 `/var/db/pkg` 损坏或不一致。

[pkg_check(8)](https://man.openbsd.org/pkg_check) 实用程序可以帮助清理。