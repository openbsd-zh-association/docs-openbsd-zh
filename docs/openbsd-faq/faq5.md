# OpenBSD FAQ - 从源码构建系统

## OpenBSD 的 flavors

OpenBSD 共有三个版本：

- **-release**：每六个月发布一次的 OpenBSD 版本。
- **-current**：开发分支。每六个月，-current 被标记并成为下一个 -release。
- **-stable**：-release 分支，以及在[勘误页面](https://www.openbsd.org/errata.html)上找到的补丁。当对 -current 进行非常重要的修复时，它们会向后移植到受支持的 -stable 分支。

只有两个最新的 OpenBSD 版本才能获得基本系统的[安全性和可靠性修复](https://www.openbsd.org/faq/faq10.html#Patches)。

新用户应该运行 -stable 或 -release。话虽如此，许多人确实在生产系统上运行 -current 以帮助捕获错误和测试新功能。

## 开发版快照

在 OpenBSD 新版本正式发布前，-current 分支的快照会被提供。这些是基于当时在代码树上的所有代码构建的快照。

最近发布的快照通常是你运行 -current 所需的全部内容。如果你希望从源代码构建它，则需要从最新的快照开始。 检查[以下 -current 和 using 快照页面](https://www.openbsd.org/faq/current.html)以了解从源代码构建所需的任何配置更改或额外步骤。

你可能会发现快照中的错误。这是它们被构建和分发的原因之一。如果你发现错误，请确保[已报告](https://www.openbsd.org/report.html)该错误。

## 从源码构建 OpenBSD

从源代码构建 OpenBSD 涉及许多步骤：

- 升级到最新的可用二进制文件
- 获取源代码
- 构建新内核和用户空间，如 [release(8)](https://man.openbsd.org/release) 的第 2 步和第 3 步所述
- 可选地，[发布](https://www.openbsd.org/faq/faq5.html#Release)并[构建 X](https://www.openbsd.org/faq/faq5.html#Xbld)

此常见问题部分旨在帮助您进行必要的准备。主要步骤请参考 `release(8)`。

### 升级到最新的可用二进制文件

不要试图通过从源代码编译来从一个版本转到另一个版本。

确保您安装了最新的可用二进制文件。如果你想构建 OpenBSD x.y-stable，请升级到最新的 OpenBSD x.y-release；如果你想要构建 -current，请升级到 -current 的最新快照。

### 获取源码

OpenBSD 使用 [CVS](https://savannah.nongnu.org/projects/cvs) 版本控制系统来管理其源代码。[cvs(1)](https://man.openbsd.org/cvs) 程序用于将所需源的副本下载到本地机器进行编译。cvs(1) 的介绍和获取源代码树的详细说明位于[匿名 CVS](https://www.openbsd.org/anoncvs.html) 页面。首先，你必须使用 `cvs checkout` 检出源代码树。之后，你可以通过运行 `cvs update` 将更新的文件拉到本地树来维护树。你还可以使用 `reposync` 程序维护本地 CVS 存储库，该程序作为一个[包](https://www.openbsd.org/faq/faq15.html)提供。[匿名 CVS](https://www.openbsd.org/anoncvs.html#rsync) 页面上也解释了设置存储库的镜像。

#### 避免使用 Root 权限

避免以 root 身份运行 cvs(1)。 默认情况下，`/usr/src` 目录（您的源代码通常所在的位置）可由 `wsrc` 组写入，因此请将需要使用 `cvs(1)` 的用户添加到该组。

```shell
# user mod -G wsrc <你的用户名>
```

此更改在下次登录时生效。

如果要以该用户身份获取 xenocara 或 ports，则必须手动创建目录并设置其权限。

```shell
# cd /usr
# mkdir -p   xenocara ports
# chgrp wsrc xenocara ports
# chmod 775  xenocara ports
```

#### 获取 -stable

要获取 -stable `src` 树，请使用 -r 标志指定所需的分支：

```shell
$ cd /usr
$ cvs -qd anoncvs@anoncvs.example.org:/cvs checkout -rOPENBSD_7_0 -P src
```

检出树后，你可以稍后更新它：

```shell
$ cd /usr/src
$ cvs -q up -Pd -rOPENBSD_7_0
```

根据需要将 `src` 替换为 `xenocara` 或 `ports`。由于 OpenBSD 的所有部分都必须保持同步，因此你使用的所有树都应该同时检出和更新。

#### 获取 -current

要获取 -current `src` 树，您可以使用以下命令：

```shell
$ cd /usr
$ cvs -qd anoncvs@anoncvs.example.org:/cvs checkout -P src
```

使用下列指令更新树：

```shell
$ cd /usr/src
$ cvs -q up -Pd -A
```

将 `src` 替换为您想要的模块，例如 `xenocara` 或 `ports`。

### 构建 OpenBSD

此时，你已准备好从源代码构建 OpenBSD。

如果你正在构建 -current，请查看[此页面](https://www.openbsd.org/faq/current.html)上列出的更改和特殊构建说明。

按照 [release(8)](https://man.openbsd.org/release) 的步骤 2 和 3 中的详细说明进行操作。

### 进一步阅读构建过程

- [mk.conf(5)](https://man.openbsd.org/mk.conf)
- [src/Makefile](https://cvsweb.openbsd.org/src/Makefile)
- [/usr/share/mk/bsd.README](https://cvsweb.openbsd.org/src/share/mk/bsd.README)
- [config(8)](https://man.openbsd.org/config)
- [options(4)](https://man.openbsd.org/options) 

## 制作一个发行版

发行版（release）是可用于在另一个系统上安装或升级 OpenBSD 的完整文件集。一个示例用途是在一台快速的机器上构建 -stable，然后发布一个安装在所有其他机器上的版本。如果你只有一台运行 OpenBSD 的计算机，你真的没有任何理由发布一个版本，因为上面的构建过程会完成你需要的一切。

关于发布的说明可在 [release(8)](https://man.openbsd.org/release) 中找到。发布过程使用上面构建过程中 `/usr/obj` 目录中创建的二进制文件。

注意：如果你希望通过 HTTP(s) 分发生成的文件集以供升级或安装脚本使用，你需要添加一个 `index.txt` 文件，其中包含新创建的版本中所有文件的列表。

```shell
# ls -nT > index.txt
```

如果你想对你创建的集合进行加密签名，[signify(1)](https://man.openbsd.org/signify) 手册页提供了有关如何执行此操作的详细信息。

### 设置你的系统

发布发行版需要一个 `noperm` 分区。这允许构建基础设施在大部分过程中使用非特权 `build` 用户。

使用 `noperm` [mount(8)](https://man.openbsd.org/mount) 选项集在 `/dest` 上创建文件系统。 相应的 [fstab(5)](https://man.openbsd.org/fstab) 行可能如下所示：

```shell
c73d2198f83ef845.m /dest ffs rw,nosuid,noperm 1 2
```

此文件系统的根目录必须由具有 700 权限的 `build` 用户拥有：

```shell
# chown build /dest
# chmod 700   /dest
```

为 base 和 xenocara 创建 DESTDIR 目录：

```shell
# mkdir /dest/{,x}base
```

你的 `RELEASEDIR` 不需要位于 `noperm` 文件系统上。确保它归 `build` 所有并且至少具有 `u=rwx` 权限。

### 使用一个 mfs noperm 分区

你可能希望使用 [mfs](https://man.openbsd.org/mount_mfs) 分区而不是物理磁盘。在 `/etc/fstab` 中添加类似于此的行：

```shell
swap /dest mfs rw,nosuid,noperm,-P/var/dest,-s1.5G,noauto 0 0
```

创建原始的 DESTDIR 目录：

```shell
# mkdir -p /var/dest/{,x}base
# chown -R build /var/dest
# chmod -R 700   /var/dest
```

现在你可以在发布之前挂载 `/dest`：

```shell
# mount /dest
```

## 构建 X

从 [X.Org](https://www.x.org/wiki/) v7 开始，X 转变为模块化构建系统，将 X.Org 源代码树拆分为三百多个或多或少的独立包。

为了简化 OpenBSD 用户的操作，OpenBSD 开发了一个名为 [Xenocara](https://xenocara.org/) 的元构建。该系统将 X 转换回一棵大树，以便在单一过程中完成构建。作为一个额外的好处，这个构建过程比以前的版本更类似于 OpenBSD 其余部分使用的构建过程。

构建 X 的官方说明存在于 [xenocara/README](https://cvsweb.openbsd.org/xenocara/README) 文件和 [release(8)](https://man.openbsd.org/release) 的第 5 步中。

## 编译时的常见问题

大多数情况下，构建过程中的问题是由于没有仔细遵循指示造成的。从最新的快照构建 -current 偶尔会出现真正的问题，但是构建 -release 或 -stable 时的失败几乎总是用户操作失误。

大多数问题通常是以下情况之一：

- 无法从[合适的二进制文件](https://www.openbsd.org/faq/faq5.html#BldBinary)启动。
- [检查](https://www.openbsd.org/faq/faq5.html#BldGetSrc)树的错误分支。
- 在构建 -current 之前忘记阅读 [following -current](https://www.openbsd.org/faq/current.html)。
- 在构建和安装新内核后跳过重启步骤。

### 我忘了在 `make build` 之前 `make obj`

通过在执行 `make obj` 之前执行 `make build`，你最终会得到分散在 `/usr/src` 目录中的目标文件。这是一件坏事。如果你希望避免再次重新获取整个 `src` 树，你可以尝试以下操作来清除 obj 文件：

```shell
$ cd /usr/src
$ find . -type l -name obj -delete
$ make cleandir
$ rm -rf /usr/obj/*
$ make obj
```

### 构建因为 “Signal 11” 错误而停止

从源代码构建 OpenBSD 和其他程序是一项对硬件要求较高的任务，需要大量使用 CPU、磁盘和内存。Signal 11 故障通常是由硬件问题引起的。

你可能会发现最好修理或更换引起故障的组件，因为问题将来可能会以其他方式出现。

有关更多信息，请参阅 [Sig11 常见问题解答](https://www.bitwizard.nl/sig11/)。

## 其他问题和提示

### 将你的用户添加到 `wobj` 组

如果你打算在源代码树中编译单个程序——例如，进行开发——你需要将你的用户添加到 `wobj` 组。这将允许你在 `/usr/obj` 读写文件。

### 给文件打标签

作为开发人员的编辑器，[mg(1)](https://man.openbsd.org/mg) 和 [vi(1)](https://man.openbsd.org/vi) 内置了对 [ctags(1)](https://man.openbsd.org/ctags) 文件的支持，允许你快速浏览源代码树。

在大多数程序或库源目录中，您可以通过运行下列指令来创建 `./tags` 文件：

```shell
$ make tags
```

在构建和安装 `libc` 时，还会创建一个 `/var/db/libc.tags` 文件。

默认情况下，每个架构的内核标签（kernel tags）位于 `/sys/arch/$(machine)/`。 你可以使用 `/sys/kern` 中的 `make` 标记创建这些文件。你可能希望以 root 身份运行 `make links` 以在每个目录和 `/var/db/` 中放置指向架构内核标记文件的符号链接。

### 如何跳过构建树的某些部分？

使用 [mk.conf(5)](https://man.openbsd.org/mk.conf) 的 `SKIPDIR` 选项。

### 我可以使用交叉编译吗？

OpenBSD 系统中包含交叉编译工具，供开发人员使用以创建新平台。但是，它们不是为了一般用途而维护的。

当开发人员提出对新平台的支持时，第一个重大测试之一就是原生构建（native-build）。从源代码构建系统会给操作系统和机器带来相当大的负载，并且可以很好地测试系统的实际工作情况。出于这个原因，OpenBSD 在构建所使用的平台上执行所有构建过程。

## 定制化内核

自定义内核的三种方式：

- 使用 [boot_config(8)](https://man.openbsd.org/boot_config) 临时启动时配置
- 使用 [config(8)](https://man.openbsd.org/config) 永久修改已编译的内核
- 编译自定义内核

### Boot-Time 配置

OpenBSD 的引导时（boot-time）内核配置 [boot_config(8)](https://man.openbsd.org/boot_config) 允许管理员修改某些内核设置，例如启用或禁用对各种设备的支持，而无需重新编译内核本身。

要进入用户内核配置（User Kernel Config, UKC），请在启动时使用 `-c` 选项：

```shell
Using drive 0, partition 3.
Loading......
probing: pc0 com0 com1 mem[638K 1918M a20=on]
disk: hd0+ hd1+
>> OpenBSD/amd64 BOOT 3.33
boot> boot hd0a:/bsd -c
```

这样做会弹出一个 UKC 提示。 键入 `help` 以获取可用命令的列表。

使用 [boot_config(8)](https://man.openbsd.org/boot_config) 仅提供临时更改，这意味着每次重新启动时都必须重复该过程。下一节将解释如何使更改永久化。

### 使用 config(8) 更改内核选项

使用 `-e` 标志调用 [config(8)](https://man.openbsd.org/config) 允许您在正在运行的系统上进入 UKC。所做的任何更改将在下次重新启动时生效。`-u` 标志测试以查看在引导期间是否对正在运行的内核进行了任何更改，这意味着您在引导系统时使用了 `boot -c` 进入 UKC。

为避免用损坏的内核覆盖正在工作的内核的风险，请考虑使用 `-o` 标志将更改写入单独的内核文件以进行测试：

```shell
# config -e -o /bsd.new /bsd
```

这会将您的更改写入 `/bsd.new` 文件。 一旦你从这个新内核启动并验证一切正常，就可以将所需的更改放置在 [bsd.re-config(5)](https://man.openbsd.org/bsd.re-config) 中使其永久化。这样做消除了在启动时选择内核的需要，并确保休眠和内核重新链接（kernel relinking）继续工作。

### 构建一个自定义内核

OpenBSD 团队仅支持 `GENERIC` 和 `GENERIC.MP` 内核。`GENERIC` 内核配置是 `/sys/arch/$(machine)/conf/GENERIC` 和 `/sys/conf/GENERIC` 中选项的组合。报告自定义内核的问题几乎总是会导致你被告知尝试使用 `GENERIC` 内核重现该问题。

首先阅读 [config(8)](https://man.openbsd.org/config) 和 [options(4)](https://man.openbsd.org/options) 手册页。以下步骤是编译自定义内核的一部分：

```shell
$ cd /sys/arch/$(machine)/conf
$ cp GENERIC CUSTOM
$ vi CUSTOM    # make your changes
$ config CUSTOM
$ cd ../compile/CUSTOM
$ make
```

## 准备 Diff

如果你对要与开发人员共享的源代码进行了更改，请遵循以下约定：

- 将你的 Diff（更改）基于 -current 进行总结。
- 使用 `cvs diff -uNp` 生成 diff。
- 通过电子邮件将你的 diff 以电子邮件的形式发送到**技术**[邮件列表](https://www.openbsd.org/mail.html)。请确保你的电子邮件客户端不会将空白处打乱。

如果你使用的是 OpenBSD 源代码树的 git 镜像，请设置：

```shell
$ git config diff.noprefix true
```

在你的存储库中并像这样生成你的 diff：

```shell
$ git diff --relative .
```