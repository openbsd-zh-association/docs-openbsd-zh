#  OpenBSD FAQ - 配置磁盘

## 磁盘和分区

在 OpenBSD 中设置磁盘的细节因[平台](https://www.openbsd.org/plat.html)而异，因此你应该阅读你部署的平台的 `INSTALL.<arch>` 文件中的说明。

### 驱动器识别

在大多数平台上，OpenBSD 使用两个驱动程序处理大容量存储：

- [wd(4)](https://man.openbsd.org/wd)：类似 IDE 的磁盘：IDE、SATA、MFM 或 ESDI 磁盘，或连接到 wdc(4) 或 [pciide(4)](https://man.openbsd.org/pciide) 接口的闪存设备。
- [sd(4)](https://man.openbsd.org/sd)：类 SCSI 磁盘：使用 SCSI 命令的设备、USB 磁盘、连接到 [ahci(4)](https://man.openbsd.org/ahci) 接口的 SATA 磁盘以及连接到 RAID 控制器的磁盘阵列。

设备按启动时检测到的顺序编号，从零开始。因此，第一个类似 IDE 的磁盘将是 `wd0`，而第三个类似 SCSI 的磁盘将是 `sd2`。请注意，OpenBSD 不一定会按照与引导 ROM 相同的顺序对驱动器进行编号。

### 分区和文件系统

术语“分区”在 OpenBSD 中可以表示两种不同的含义：

- [disklabel(8)](https://man.openbsd.org/disklabel) 分区，也称为文件系统分区。
- [fdisk(8)](https://man.openbsd.org/fdisk) 分区，有时称为主引导记录 (MBR) 分区。

所有 OpenBSD 平台都使用 disklabel 程序作为管理文件系统分区的主要方式。在使用 fdisk 的平台上，一个 MBR 分区可保存所有 OpenBSD 文件系统。这个分区可以分成 16 个 disklabel 分区，标记为 `a` 到 `p`。有几个标签很特别：

- a：启动盘的 a 分区就是你的根分区。
- b: 启动盘的 b 分区通常是交换分区。
- c：c 分区始终是整个磁盘。

要在 disklabel 分区上创建新文件系统，请使用 [newfs(8)](https://man.openbsd.org/newfs) 命令：

```
# newfs sd2a
```

因此，一个设备名加上一个 disklabel 标识了一个 OpenBSD 文件系统。 例如，标识符 `sd2a` 指的是第三个 `sd` 设备的 `a` 分区上的文件系统。它的设备文件将是块设备的 `/dev/sd2a` 和原始（字符）设备的 `/dev/rsd2a`。 记住一个很少使用的命令是需要块设备还是字符设备是困难的。因此，许多命令都使用 [opendev(3)](https://man.openbsd.org/opendev) 函数，它会根据需要自动将 `sd0` 扩展为 `/dev/rsd0c` 或 `/dev/sd0c`。

### Disklabel 唯一标识符

默认情况下，磁盘由 [fstab(5)](https://man.openbsd.org/fstab) 文件中的 Disklabel 唯一标识符 (DUID) 标识。 DUID 是在首次创建磁盘标签时生成的 16 位十六进制数字随机数。它们由 [diskmap(4)](https://man.openbsd.org/diskmap) 设备管理。要显示所有磁盘的 DUID，请执行以下操作：

```
$ sysctl hw.disknames
hw.disknames=wd0:bfb4775bb8397569,cd0:,wd1:56845c8da732ee7b,wd2:f18e359c8fa2522b
```

你可以通过附加句点和分区号来指定磁盘上的分区。例如，`f18e359c8fa2522b.d` 是磁盘 `f18e359c8fa2522b` 的 `d` 分区，并且将始终引用同一个存储块，无论设备连接到系统的顺序如何，或者它连接到什么样的接口。如果你把数据放在 `wd2d` 上，然后从系统中删除 `wd1` 并重新启动，你的数据现在在 `wd1d` 上，因为你的旧 `wd2` 现在是 `wd1`。 但是，驱动器的 DUID 在启动后不会改变。

## 使用 fdisk

[fdisk(8)](https://man.openbsd.org/fdisk) 实用程序在某些平台（i386、amd64 和 macppc）上用于创建系统引导 ROM 识别的分区。通常，只有一个 OpenBSD fdisk 分区会被放置在一个磁盘上，然后该分区将被细分为 disklabel 分区。

运行下列指令查看你的分区表：

```
# fdisk sd0
Disk: sd0       geometry: 553/255/63 [8883945 Sectors]
Offset: 0       Signature: 0xAA55
         Starting       Ending       LBA Info:
 #: id    C   H  S -    C   H  S [       start:      size   ]
------------------------------------------------------------------------
 0: 12    0   1  1 -    2 254 63 [          63:       48132 ] Compaq Diag.
 1: 00    0   0  0 -    0   0  0 [           0:           0 ] unused
 2: 00    0   0  0 -    0   0  0 [           0:           0 ] unused
*3: A6    3   0  1 -  552 254 63 [       48195:     8835750 ] OpenBSD
```

在这里，标有 * 的 OpenBSD 分区 (id `A6`) 表示它是可引导分区。

一个完全空白的磁盘需要在引导前将主引导记录的引导代码写入磁盘。通常，你需要做的就是：

```
# fdisk -iy sd0
```

或者，在交互模式下使用 reinit 或 update 命令:

使用 `-e` 标志启动交互式编辑模式：

```
# fdisk -e sd0
Enter 'help' for information
fdisk: 1>
```

请注意，`quit` 会保存更改并退出程序，而 `exit` 会退出而不保存。这与许多人现在在其他环境中习惯的情况相反。另请注意，`fdisk` 在保存更改之前不会发出警告。

如果你的系统有维护或诊断分区，建议你在保留该分区或安装 OpenBSD **之前**将其安装。

## BSD 盘标

BSD 盘标（Disk labels，disklabel）用于管理 OpenBSD 文件系统分区。它们包含有关磁盘的某些详细信息，例如 [disklabel(5)](https://man.openbsd.org/disklabel.5) 手册页中的详细描述的驱动器几何结构和文件系统信息。你可以使用 [disklabel(8)](https://man.openbsd.org/disklabel) 命令编辑标签。

这可以帮助克服某些架构的磁盘分区限制。例如，在 i386 上，只有四个主 MBR 分区可用。使用 BSD 盘标 ，这些主分区之一包含你所有的 OpenBSD 分区，而其他三个仍可用于其他操作系统。

在使用 fdisk 的平台上，你应该在 BSD 盘标和 fdisk 中保留未使用的第一个逻辑磁道。出于这个原因，默认是在块 64 处启动第一个分区。

不要将 swap 分区放在 sparc64 上磁盘的最开头。虽然 Solaris 经常这样做，但 OpenBSD 要求引导分区位于磁盘的开头。

### 删除 BSD 盘标后恢复分区

如果你的分区表已损坏，你可以尝试多种方法来恢复它。

作为日常系统维护的一部分，每个磁盘的 BSD 盘标副本都保存在 `/var/backups` 中。 假设你仍然有 `/var` 分区，你可以简单地读取输出，并使用 `-R` 标志将其放回 BSD 盘标中。

如果你无法再看到该分区，则有两种选择：修复磁盘足够多的部分以便你可以看到它，或者修复磁盘足够多的部分以便你可以取出数据。[scan_ffs(8)](https://man.openbsd.org/scan_ffs) 实用程序将查看磁盘以查找分区。你可以使用它找到的信息重新创建 BSD 盘标。如果你只想恢复 `/var`，你可以为 `/var` 重新创建分区，然后恢复备份的标签并从中添加其余部分。[disklabel(8)](https://man.openbsd.org/disklabel) 实用程序将更新内核对 BSD 盘标的理解并尝试将标签写入磁盘。因此，即使包含 BSD 盘标的磁盘区域不可读，你也可以挂载直到下次重新启动。

## amd64 启动过程

有关 amd64 引导程序的详细信息，请参见 [boot_amd64(8)](https://man.openbsd.org/boot_amd64) 手册页。启动过程如下：

- 主引导记录 (MBR) 和 GUID 分区表 (GPT)。[fdisk(8)](https://man.openbsd.org/fdisk) 手册页包含详细信息。
- 分区引导记录 (PBR)。引导盘的 OpenBSD 分区的前 512 字节包含第一阶段引导加载程序 [biosboot(8)](https://man.openbsd.org/biosboot)。 它由 [installboot(8)](https://man.openbsd.org/installboot) 实用程序安装。
- 第二阶段引导加载程序 `/boot`。PBR 加载 [boot(8)](https://man.openbsd.org/boot.8) 程序，该程序具有定位和加载内核的任务。

因此，启动过程的一开始可能如下所示：

```
Using drive 0, partition 3.                      <- MBR
Loading......                                    <- PBR
probing: pc0 com0 com1 mem[638K 1918M a20=on]    <- /boot
disk: hd0+ hd1+
>> OpenBSD/amd64 BOOT 3.33
boot>
booting hd0a:/bsd 4464500+838332 [58+204240+181750]=0x56cfd0
entry point at 0x100120

[ using 386464 bytes of bsd ELF symbol table ]
Copyright (c) 1982, 1986, 1989, 1991, 1993       <- Kernel
        The Regents of the University of California.  All rights reserved.
```

## 软更新

软更新（Soft Updates）基于 [Greg Ganger 和 Yale Patt](https://www.ece.cmu.edu/~ganger/papers/CSE-TR-254-95/) 提出并由 [Kirk McKusick](https://www.mckusick.com/softdep/) 为 FreeBSD 开发的想法。软更新对缓冲区缓存操作进行了部分排序，这允许从 FFS 代码中删除同步写入目录条目的要求。因此，可以看到磁盘写入性能的大幅提升。

要启用软更新必须使用 mount-time 选项。 使用 [mount(8)](https://man.openbsd.org/mount) 实用程序挂载分区时，你可以指定希望在该分区上启用软更新。下面是一个示例 [fstab(5)](https://man.openbsd.org/fstab) 条目，它有一个我们希望安装有软更新的分区 `sd0a`。

```
/dev/sd0a / ffs rw,softdep 1 1
```

## 根分区备份（/altroot）

OpenBSD 在 [daily(8)](https://man.openbsd.org/daily) 脚本中提供了一个 `/altroot` 工具。 如果在 `/etc/daily.local` 或 root 的 [crontab(5)](https://man.openbsd.org/crontab.5) 中设置了环境变量 `ROOTBACKUP=1`，并且在 [fstab(5)](https://man.openbsd.org/fstab) 中将分区指定为挂载到 `/altroot` 并使用 `xx` 挂载选项，则每天晚上整个根分区的内容将复制到 `/altroot` 分区。

假设你要将根分区备份到 DUID 为 `bfb4775bb8397569.a` 指定的分区，在 `/etc/fstab` 中添加以下内容：

```
bfb4775bb8397569.a /altroot ffs xx 0 0
```

然后，在 `/etc/daily.local` 中设置适当的环境变量：

```
# echo ROOTBACKUP=1 >>/etc/daily.local
```

由于 `/altroot` 进程将捕获你的 `/etc` 目录，这将确保每天更新同步根分区的任何配置更改。这是使用 [dd(1)](https://man.openbsd.org/dd) 完成的 “磁盘镜像” 副本，而不是逐个文件的副本，因此你的 `/altroot` 分区应至少与根分区的大小相同。通常，你需要将 `/altroot` 分区放置在已配置为在主磁盘出现故障时完全可引导的不同磁盘上。

## 复制文件系统

要使用 [dump(8)](https://man.openbsd.org/dump) 和 [restore(8)](https://man.openbsd.org/restore) 将目录 `/SRC` 下的所有内容复制到目录 `/DST`，请执行以下操作：

```
# cd /SRC && dump 0f - . | (cd /DST && restore -rf - )
```

或者使用 [tar(1)](https://man.openbsd.org/tar)

```
# cd /SRC && tar cf - . | (cd /DST && tar xpf - )
```

## 磁盘配额

配额用于限制某些用户和组可用的磁盘空间量。

使用关键字 `userquota` 和 `groupquota` 来标记 [fstab(5)](https://man.openbsd.org/fstab) 中要对其实施配额的每个文件系统。默认情况下，文件 `quota.user` 和 `quota.group` 将在这些文件系统的根目录下创建。这是一个示例 `/etc/fstab` 行：

```
0123456789abcdef.k /home ffs rw,nodev,nosuid,userquota 1 2
```

要设置用户的配额，请使用 [edquota(8)](https://man.openbsd.org/edquota)。例如:

```
# edquota ericj
```

并编辑软限制和硬限制：

```
Quotas for user ericj:
/home: KBytes in use: 62, limits (soft = 1000000, hard = 1500000)
        inodes in use: 25, limits (soft = 0, hard = 0)
```

在本例中，软限制设置为 1000000k，硬限制设置为 1500000k。由于相应的软限制和硬限制都设置为 0，因此不会强制执行对文件总数（inode）数量的限制。超过软限制的用户会收到警告并获得宽限期，以使他们的磁盘使用量低于限制。可以使用 [edquota(8)](https://man.openbsd.org/edquota) 上的 `-t` 选项设置宽限期。宽限期结束后，软限制作为硬限制处理。这通常会导致分配失败。

使用 [quotaon(8)](https://man.openbsd.org/quotaon) 启用配额：

```
# quotaon -a
```

这将扫描 [fstab(5)](https://man.openbsd.org/fstab) 并使用配额选项在文件系统上启用配额。使用 [quota(1)](https://man.openbsd.org/quota) 查看配额统计信息。

## 访问其他文件系统

从 [mount(8)](https://man.openbsd.org/mount) 手册开始，其中包含解释如何挂载一些最常用文件系统的示例。可以通过以下方式获得支持的文件系统和相关命令的部分列表：

```
$ man -k -s 8 mount
```

请注意，支持可能仅限于只读操作。

## 挂载磁盘镜像

要在 OpenBSD 中挂载磁盘镜像，你必须配置 [vnd(4)](https://man.openbsd.org/vnd) 设备。 例如，如果你有一个位于 `/tmp/ISO.image` 的 ISO 镜像，你将执行以下步骤来挂载该镜像：

```
# vnconfig vnd0 /tmp/ISO.image
# mount -t cd9660 /dev/vnd0c /mnt
```

由于这是 CD 和 DVD 使用的 ISO 9660 镜像格式，因此在安装它时必须指定 `cd9660`的类型。

要卸载映像并取消配置 vnd(4) 设备，请执行以下操作：

```
# umount /mnt
# vnconfig -u vnd0
```

有关更多信息，请参阅 [vnconfig(8)](https://man.openbsd.org/vnconfig) 和 [mount(8)](https://man.openbsd.org/mount)。

## 磁盘分区扩容

如果现有分区后跟未分配的可用空间，你可以使用 growfs(8) 实用程序增加其大小。确保当前未挂载该分区。使用 `disklabel -E sd0` 以交互方式编辑你的分区表，并使用 `m` 命令修改分区的大小。你可以使用 [growfs(8)](https://man.openbsd.org/growfs) 调整文件系统以使用整个分区：

```
# growfs sd0h
```

在再次挂载分区之前，必须使用 [fsck(8)](https://man.openbsd.org/fsck) 检查其完整性：

```
# fsck /dev/sd0h
```

## RAID 和磁盘加密

[bioctl(8)](https://man.openbsd.org/bioctl) 命令通过 [bio(4)](https://man.openbsd.org/bio) 层管理硬件和软件 RAID 设备。[softraid(4)](https://man.openbsd.org/softraid) 子系统允许将多个 OpenBSD [disklabel(8)](https://man.openbsd.org/disklabel) 分区组合成一个虚拟 [sd(4)](https://man.openbsd.org/sd) 磁盘。这个虚拟磁盘被视为和任何其他磁盘一样的磁盘，首先用 [fdisk](https://www.openbsd.org/faq/faq14.html#fdisk)（在支持 fdisk 的平台上）分区，然后像往常一样创建 BSD 盘标。

支持的 softraid 规则包括：

- RAID0（带区集）
- RAID1（镜像）
- RAID1C（使用磁盘加密镜像）
- RAID5（带浮动奇偶校验的带区集）
- CONCAT（无冗余连接，类似于 JBOD）
- CRYPTO（磁盘加密）

磁盘设置可能因平台而异，并且**并非所有平台都支持从 softraid 设备启动**。 目前只能从 i386、amd64、arm64 和 sparc64 上的 RAID1 和加密卷启动 softraid。

### 安装至镜像

本节介绍将 OpenBSD 安装到一对镜像硬盘驱动器，并假设你熟悉[安装过程](https://www.openbsd.org/faq/faq4.html)。

在使用安装脚本之前，你将进入一个 shell 并设置一个 [softraid(4)](https://man.openbsd.org/softraid) 设备。

安装内核在启动时只有有限数量的 `/dev` 条目，因此你需要为你的 softraid 设置手动创建所需的磁盘设备。例如，如果你需要支持三个 [sd(4)](https://man.openbsd.org/sd) 设备，则可以从 shell 提示符执行以下操作：

```
Welcome to the OpenBSD/amd64 7.0 installation program.
(I)nstall, (U)pgrade, (A)utoinstall or (S)hell? s
# cd /dev
# sh MAKEDEV sd0 sd1 sd2
```

安装程序现在将完全支持 `sd0`、`sd1` 和 `sd2` 设备。如果要从 USB 驱动器安装这些[套件](https://www.openbsd.org/faq/faq4.html#FilesNeeded)，也不要忘记考虑该设备。

接下来，使用 [fdisk(8)](https://man.openbsd.org/fdisk) 初始化磁盘并使用 [disklabel(8)](https://man.openbsd.org/disklabel) 创建 RAID 分区。

如果你从 MBR 启动，请执行以下操作：

```
# fdisk -iy sd0
# fdisk -iy sd1
```

如果你使用 GPT 进行 UEFI 引导，请执行以下操作：

```
# fdisk -iy -g -b 960 sd0
# fdisk -iy -g -b 960 sd1
```

在第一台设备上创建分区布局（partition layout）：

```
# disklabel -E sd0
Label editor (enter '?' for help at any prompt)
sd0> a a
offset: [64]
size: [39825135] *
FS type: [4.2BSD] RAID
sd0*> w
sd0> q
No label changes.
```

将分区布局复制到第二个设备：

```
# disklabel sd0 > layout
# disklabel -R sd1 layout
# rm layout
```

使用 [bioctl(8)](https://man.openbsd.org/bioctl) 命令组装镜像：

```
# bioctl -c 1 -l sd0a,sd1a softraid0
scsibus1 at softraid0: 1 targets
sd2 at scsibus2 targ 0 lun 0: <OPENBSD, SR RAID 1, 005> SCSI2 0/direct fixed
sd2: 10244MB, 512 bytes/sec, 20980362 sec total
```

这表明我们现在有一个新的 SCSI 总线和一个新的磁盘 sd2。系统启动时将自动检测并组装此卷。

即使你创建了多个 RAID 阵列，设备名称也将始终为 `softraid0`。不会有 `softraid1` 或其他任何东西。

因为新设备可能有很多无用数据，你可能需要使用主引导记录和磁盘标签，所以强烈建议将它的第一个块清零。使用此命令要**非常小心**；在错误的设备上应用它可能会导致非常糟糕的一天。这假设新的 softraid 设备创建为 `sd2`。

```
# dd if=/dev/zero of=/dev/rsd2c bs=1m count=1
```

你现在已准备好在你的系统上安装 OpenBSD。通过在启动介质控制台上调用 “安装” 或 “退出” 来正常执行安装。在新的 softraid 磁盘（此处示例中的 sd2）上创建所有分区，而不是在 `sd0` 或 `sd1`（非 RAID 磁盘）上。

要检查镜像的状态，请运行以下命令：

```
# bioctl sd2
```

每晚检查状态的 cron 作业可能是个好主意。

#### 重建镜像

当驱动器发生故障时，你将更换故障驱动器，创建 RAID 和其他 BSD 盘标分区，然后重建镜像。假设你的 RAID 卷是 `sd2` 并且你正在用 `sd1m` 替换出现故障的设备，以下命令应该可以工作：

```
# bioctl -R /dev/sd1m sd2
```

这也可以在[单用户模式](https://www.openbsd.org/faq/faq10.html#LostPW)下或从[安装内核](https://www.openbsd.org/faq/faq4.html#bsd.rd)执行。

### 全盘加密

与 RAID 非常相似，OpenBSD 中的全盘加密由 [softraid(4)](https://man.openbsd.org/softraid) 子系统和 [bioctl(8)](https://man.openbsd.org/bioctl) 命令处理。本节介绍了将 OpenBSD 安装到单个加密磁盘的过程，这与上一节非常相似。请注意，此时**不支持** “堆叠” 软 RAID 模式。

在初始提示下选择 (S)hell：

```
Welcome to the OpenBSD/amd64 7.0 installation program.
(I)nstall, (U)pgrade, (A)utoinstall or (S)hell? s
```

从这里开始，你将在实时（live）环境中获得一个 shell 来操作磁盘。在这个例子中，我们将安装到 `sd0` SATA 驱动器，擦除它之前的所有内容。

由于安装程序默认没有很多设备节点，请确保 `/dev/sd0` 设备存在：

```
# cd /dev && sh MAKEDEV sd0
```

首先，你可能需要使用以下内容将随机数据写入驱动器：

```
# dd if=/dev/urandom of=/dev/rsd0c bs=1m
```

这可能是一个非常耗时的过程，具体取决于 CPU 和磁盘的速度以及磁盘的大小。 如果你不向整个设备写入随机数据，攻击者可能会推断出实际使用了多少空间。

接下来，使用 [fdisk(8)](https://man.openbsd.org/fdisk) 初始化磁盘并使用 [disklabel(8)](https://man.openbsd.org/disklabel) 创建 softraid 分区。

如果你从 MBR 启动，请执行以下操作：

```
# fdisk -iy sd0
```

如果你使用 GPT 进行 UEFI 引导，请执行以下操作：

```
# fdisk -iy -g -b 960 sd0
```

接下来，创建分区布局：

```
# disklabel -E sd0
Label editor (enter '?' for help at any prompt)
sd0> a a			
offset: [64]
size: [39825135] *
FS type: [4.2BSD] RAID
sd0*> w
sd0> q
No label changes.
```

我们将使用整个磁盘，但请注意，加密设备可以拆分为多个分区，就好像它是一个普通的硬盘驱动器一样。

现在我们可以在我们的 “a” 分区上构建加密设备。

```
# bioctl -c C -l sd0a softraid0
New passphrase:
Re-type passphrase:
sd1 at scsibus2 targ 1 lun 0: <OPENBSD, SR CRYPTO, 005> SCSI2 0/direct fixed
sd1: 19445MB, 512 bytes/sector, 39824607 sectors
softraid0: CRYPTO volume attached as sd1
```

你可能想要使用[密钥盘](https://www.openbsd.org/faq/faq14.html#softraidFDEkeydisk)（keydisk）而不是密码短语：

确保 `/dev/sd1` 设备被考虑在内：

```
# cd /dev && sh MAKEDEV sd1
```

写入 `sd1` 的所有数据现在都将在 XTS 模式下使用 AES 加密。

与前面的示例一样，我们将覆盖新的伪设备（pseudo-device）的第一兆字节。

```
# dd if=/dev/zero of=/dev/rsd1c bs=1m count=1
```

键入 exit 返回主安装程序，然后选择此新设备作为你的安装设备。

```
[...]
Available disks are: sd0 sd1.
Which disk is the root disk? ('?' for details) [sd0] sd1
```

启动时将提示你输入密码，但所有其他操作都应透明处理。

#### 使用密钥盘

作为使用密码短语的替代方法，可以使用存储在单独设备（例如 U 盘）上的密钥来解锁你的加密磁盘。

使用 [fdisk(8)](https://man.openbsd.org/fdisk) 初始化你的密钥盘（keydisk），然后使用 [disklabel(8)](https://man.openbsd.org/disklabel) 为密钥数据创建一个 1 MB 的 RAID 分区。 如果你的密钥盘是 `sd1` 而你要加密的驱动器是 `sd0`，则输出将如下所示：

```
# bioctl -c C -k sd1a -l sd0a softraid0
sd2 at scsibus3 targ 1 lun 0: <OPENBSD, SR CRYPTO, 005> SCSI2 0/direct fixed
sd2: 19445MB, 512 bytes/sector, 39824607 sectors
softraid0: CRYPTO volume attached as sd2
```

系统不会提示你输入密码，因为你使用的是密钥盘。必须在启动时插入密钥盘。

你可以使用 [dd(1)](https://man.openbsd.org/dd) 备份和恢复你的密钥盘：

```
# dd bs=8192 skip=1 if=/dev/rsd1a of=backup-keydisk.img
# dd bs=8192 seek=1 if=backup-keydisk.img of=/dev/rsd1a
```

### 加密外部磁盘

本节说明如何为外部 USB 驱动器设置加密软 RAID 卷。如果你已经阅读了有关全盘加密的部分，那么应该非常熟悉。步骤概要如下：

- 用随机数据覆盖驱动器的内容
- 使用 [disklabel(8)](https://man.openbsd.org/disklabel) 创建所需的 RAID 类型分区
- 使用 [bioctl(8)](https://man.openbsd.org/bioctl) 加密驱动器
- 将新伪分区的第一兆字节归零
- 使用 [newfs(8)](https://man.openbsd.org/newfs) 在伪设备上创建文件系统
- 解锁并使用 [mount(8)](https://man.openbsd.org/mount) 挂载新的伪设备
- 根据需要访问文件
- 卸载驱动器并分离加密容器

下面是这些步骤的快速示例，其中 `sd3` 是 USB 驱动器：

```
# dd if=/dev/urandom of=/dev/rsd3c bs=1m
# fdisk -iy sd3
# disklabel -E sd3 # make an "a" partition of type RAID
# bioctl -c C -l sd3a softraid0
New passphrase:
Re-type passphrase:
softraid0: CRYPTO volume attached as sd4
# dd if=/dev/zero of=/dev/rsd4c bs=1m count=1
# fdisk -iy sd4
# disklabel -E sd4 # make an "i" partition
# newfs sd4i
# mkdir -p /mnt/secretstuff
# mount /dev/sd4i /mnt/secretstuff
# mv somefile /mnt/secretstuff/
# umount /mnt/secretstuff
# bioctl -d sd4
```

用于创建卷的相同 [bioctl(8)](https://man.openbsd.org/bioctl) 命令可用于稍后附加它。