# OpenBSD 升级向导：6.9 至 7.0

[[FAQ Index](https://www.openbsd.org/faq/index.html)] | [[6.8 -> 6.9](https://www.openbsd.org/faq/upgrade69.html)]

*仅支持从一个版本升级到紧随其后的版本。*

***在尝试之前通读并理解此过程。对于重要的或物理远程机器，首先在相同的本地系统上对其进行测试。***

## 在使用任何升级方法之前

- <strong>检查 /usr 中的可用磁盘空间。</strong>确认 /usr 分区的大小至少为 1.1G。如果空间较小，升级可能会失败，你应该考虑重新安装系统。

- <strong>阅读配置和语法更改以及软件包升级说明。</strong>在开始升级之前，可能需要计划一些[配置更改](https://www.openbsd.org/faq/upgrade70.html#ConfigChanges)和[软件包更改](https://www.openbsd.org/faq/upgrade70.html#SpecialPackages)。

## 升级方法

- <strong>无人值守升级：</strong>最简单的方法是使用 [sysupgrade(8)](https://man.openbsd.org/OpenBSD-7.0/sysupgrade.8) 进行无人值守升级。该程序将下载所有安装文件集（install sets），验证其签名，然后重新启动以自动执行升级。无人值守升级完成后，继续[下面](https://www.openbsd.org/faq/upgrade70.html#AfterSets)的操作。

- <strong>交互式升级：</strong>如果你坚持省略一些安装集，你将需要执行[交互式升级](https://www.openbsd.org/faq/upgrade70.html#InteractiveUpgrade)。（带有所有安装集的 sysupgrade 升级。）

- <strong>手动升级：</strong>最后一个选项是[手动升级系统](https://www.openbsd.org/faq/upgrade70.html#NoInstKern)。（不建议这样做，因为它是最容易出错的方法。）

----

## 交互式升级

获取并验证 `bsd.rd`。为你的机器下载相应架构的 ramdisk 内核和加密签名的校验和文件。

| 文件名 | 对应架构的下载地址 |
| ------ | ------| 
| `bsd.rd` | [alpha](https://cdn.openbsd.org/pub/OpenBSD/7.0/alpha/bsd.rd) [amd64](https://cdn.openbsd.org/pub/OpenBSD/7.0/amd64/bsd.rd) [arm64](https://cdn.openbsd.org/pub/OpenBSD/7.0/arm64/bsd.rd) [armv7](https://cdn.openbsd.org/pub/OpenBSD/7.0/armv7/bsd.rd) [hppa](https://cdn.openbsd.org/pub/OpenBSD/7.0/hppa/bsd.rd) [i386](https://cdn.openbsd.org/pub/OpenBSD/7.0/i386/bsd.rd) [landisk](https://cdn.openbsd.org/pub/OpenBSD/7.0/landisk/bsd.rd) [loongson](https://cdn.openbsd.org/pub/OpenBSD/7.0/loongson/bsd.rd) [luna88k](https://cdn.openbsd.org/pub/OpenBSD/7.0/luna88k/bsd.rd) [macppc](https://cdn.openbsd.org/pub/OpenBSD/7.0/macppc/bsd.rd) [octeon](https://cdn.openbsd.org/pub/OpenBSD/7.0/octeon/bsd.rd) [powerpc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/powerpc64/bsd.rd) [sparc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/sparc64/bsd.rd) |
| `SHA256.sig` | [alpha](https://cdn.openbsd.org/pub/OpenBSD/7.0/alpha/SHA256.sig) [amd64](https://cdn.openbsd.org/pub/OpenBSD/7.0/amd64/SHA256.sig) [arm64](https://cdn.openbsd.org/pub/OpenBSD/7.0/arm64/SHA256.sig) [armv7](https://cdn.openbsd.org/pub/OpenBSD/7.0/armv7/SHA256.sig) [hppa](https://cdn.openbsd.org/pub/OpenBSD/7.0/hppa/SHA256.sig) [i386](https://cdn.openbsd.org/pub/OpenBSD/7.0/i386/SHA256.sig) [landisk](https://cdn.openbsd.org/pub/OpenBSD/7.0/landisk/SHA256.sig) [loongson](https://cdn.openbsd.org/pub/OpenBSD/7.0/loongson/SHA256.sig) [luna88k](https://cdn.openbsd.org/pub/OpenBSD/7.0/luna88k/SHA256.sig) [macppc](https://cdn.openbsd.org/pub/OpenBSD/7.0/macppc/SHA256.sig) [octeon](https://cdn.openbsd.org/pub/OpenBSD/7.0/octeon/SHA256.sig) [powerpc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/powerpc64/SHA256.sig) [sparc64](https://cdn.openbsd.org/pub/OpenBSD/7.0/sparc64/SHA256.sig) |

使用 [signify(1)](https://man.openbsd.org/OpenBSD-7.0/signify) 验证 `bsd.rd` 和 `SHA256.sig`：

```
$ signify -C -p /etc/signify/openbsd-70-base.pub -x SHA256.sig bsd.rd
Signature Verified
bsd.rd: OK
```

接下来，从上一步中获取的安装内核 `bsd.rd` 启动。把它放在你的文件系统的根目录下，并指示引导加载程序启动这个内核。启动此内核后，选择 `(U)pgrade` 选项并按照提示进行操作。

安装文件集后，系统将使用升级的内核重新启动。现在继续[下一步](https://www.openbsd.org/faq/upgrade70.html#AfterSets)：

## 在升级后

升级文件集后，系统将使用升级的内核重新启动，并在引导期间运行 [sysmerge(8)](https://man.openbsd.org/OpenBSD-7.0/sysmerge)。在某些情况下，若无法自动修改配置文件。则运行以下命令：

```
# sysmerge
```

检查并执行这些[配置更改](https://www.openbsd.org/faq/upgrade70.html#ConfigChanges)。

接下来[删除旧文件](https://www.openbsd.org/faq/upgrade70.html#RmFiles)。最后使用 `pkg_add -u` 升级软件包。

你可能需要检查[勘误表页面](https://www.openbsd.org/errata70.html)以了解任何发布后的修复。

----

## 手动升级（没有安装内核）

<span style="color:red">这不是推荐的过程。如果可能，请使用无人值守或交互式升级方法！</span>

有时，你需要对无法进行正常无人值守或交互式升级过程的机器进行升级。

### 准备

- <strong>将安装文件放在一个适当的位置</strong>。确保你有足够的空间！远程升级时空间不足可能是……不幸的。请注意，使用 softdep 会加剧这种情况，因为删除和覆盖的文件不会立即释放它们的空间。考虑在 `/etc/fstab` 中禁用 `softdep` 挂载选项并在进行手动升级之前重新启动。建议在 `/usr` 上至少有 500MB 可用空间。

- <strong>成为 root 用户</strong>。虽然在每个命令之前使用 [doas(1)](https://man.openbsd.org/OpenBSD-7.0/doas) 通常是一个好习惯，但该命令可能会被最后的步骤破坏，因此你应该在开始此过程之前获得 root 权限。此时最好使用 doas 以外的方法验证你对 root 的访问权限，即直接登录或使用 [su(1)](https://man.openbsd.org/OpenBSD-7.0/su)。

- <strong>停止和/或禁用任何适当的应用程序</strong>。在此过程中，所有用户态应用程序都会被替换，但可能无法运行，因此可能会发生奇怪的事情。在第一次重新启动期间，你可能还会遇到 DNS 解析问题，因此依赖于 DNS 的 PF 规则和 NFS 挂载可能会导致启动问题。可能还有其他的应用程序，你希望在升级后不要立即运行；也要停止并禁用它们。

- <strong>安装新的引导块</strong>。这实际上应该在任何升级结束时完成。如果这被忽略了，那么现在不这样做可能会破坏串行控制台或其他东西，具体取决于你的平台。使用 [installboot(8)](https://man.openbsd.org/OpenBSD-7.0/amd64/installboot) 进行配置，假设 `sd0` 是你的启动盘：

```
# installboot sd0
```

### 手动升级

**安装新内核**。复制主内核的额外步骤是为了确保磁盘上始终存在有效的内核。

如果使用多处理器内核：

```
# cd /usr/rel    # where you put the release files
# ln -f /bsd /obsd && cp bsd.mp /nbsd && mv /nbsd /bsd
# cp bsd.rd /
# cp bsd /bsd.sp
```

如果使用单处理器内核：

```
# cd /usr/rel    # where you put the release files
# ln -f /bsd /obsd && cp bsd /nbsd && mv /nbsd /bsd
# cp bsd.rd bsd.mp /    # may give a harmless warning
```

**启用 KARL**。存储内核的校验和：

```
# sha256 -h /var/db/kernel.SHA256 /bsd 
```

**安装新的用户空间。**保存reboot(8) 的副本，解压缩并安装发行版包，重新启动。最后安装 `base70.tgz`，因为新的基本系统，特别是 [tar(1)](https://man.openbsd.org/OpenBSD-7.0/tar)、[gzip(1)](https://man.openbsd.org/OpenBSD-7.0/gzip) 和 [reboot(8)](https://man.openbsd.org/OpenBSD-7.0/reboot)，将无法与旧内核一起使用。手动解压所需的文件集：

```
# cp /sbin/reboot /sbin/oreboot
# tar -C / -xzphf xshare70.tgz
# tar -C / -xzphf xserv70.tgz
# tar -C / -xzphf xfont70.tgz
# tar -C / -xzphf xbase70.tgz
# tar -C / -xzphf man70.tgz
# tar -C / -xzphf game70.tgz
# tar -C / -xzphf comp70.tgz
# tar -C / -xzphf base70.tgz    # Install last!
# /sbin/oreboot
```

或者，如果你使用 [ksh(1)](https://man.openbsd.org/OpenBSD-7.0/ksh)，你可以：

```
# cp /sbin/reboot /sbin/oreboot
# for _f in [!b]*70.tgz base70.tgz; do tar -C / -xzphf "$_f" || break; done
# /sbin/oreboot
```

请注意，[tar(1)](https://man.openbsd.org/OpenBSD-7.0/tar) 每次调用只能扩展一个存档，因此简单的 glob 不起作用。

**重启后，更新 `/dev`**。运行 [MAKEDEV(8)](https://man.openbsd.org/OpenBSD-7.0/amd64/MAKEDEV)：

```
# cd /dev
# ./MAKEDEV all
```

**更新引导加载程序**。仍然假设 `sd0` 是你的启动盘：

```
# installboot sd0
```

**更新系统配置文件**。运行 [sysmerge(8)](https://man.openbsd.org/OpenBSD-7.0/sysmerge)：

```
# sysmerge
```

**更新固件**。你的系统可能有新固件。用 [fw_update(1)](https://man.openbsd.org/OpenBSD-7.0/fw_update) 更新它：

```
# fw_update
```

**完成**。查看引导的控制台输出（使用 `dmesg -s`）并根据需要更正任何故障。以下[配置更改](https://www.openbsd.org/faq/upgrade70.html#ConfigChanges)后的所有步骤也适用于手动升级。最后，删除 `/sbin/oreboot` 并更新软件包：`pkg_add -u`。 再次重新启动以确保你使用最新的固件文件并在你自己的 KARL 生成的内核上运行。

----

## 配置和语法更改

**snmpd(8)**。默认安全设置已收紧。

1. 默认协议已更改为 SNMPv3。
2. 默认的 `public` 和 `private` 社区已被删除。现在必须明确设置社区。
3. `seclevel` 默认值从 `none` 更改为 `enc`。
4. 默认 SNMPv3 加密已更改为 AES。
5. `trap receiver`（陷阱接收器，用于向另一台主机发送陷阱）的协议版本不再默认为 SNMPv2。

要配置 SNMPv3，你需要在配置中添加一个或多个用户，例如：

```
user "manager" authkey "XblueQ300ZyAbUIbndmWjfl" auth hmac-sha1 enc aes enckey "tVadj9jxq8rdJ"
```

如果你需要恢复 SNMPv1/v2c，你可以在 snmpd.conf(5) 中添加如下内容：

```
listen on any snmpv1 snmpv2c read
read-only community U9PeBY1694bcxMnm
seclevel none
```

社区名称不应该是常见的或容易被暴力破解的，特别是如果暴露在互联网上。

**snmp(1)**
1. 默认协议版本已从 v2c 更改为 v3。
2. 默认加密已更改为 AES。
3. 默认身份验证已更改为 SHA1。
4. 默认社区 `public` 已被删除。

## 删除的文件

dmx 库已被删除。
    
```
# rm -f /usr/X11R6/lib/libdmx.* \
/usr/X11R6/include/X11/extensions/dmxext.h \
/usr/X11R6/lib/pkgconfig/dmx.pc \
/usr/X11R6/man/man3/DMX*.3
```

你可以借助 sysclean 包进行更详细的清理。

## 特殊的软件包

**net/freeradius**。FreeRADIUS 在 3.0.22 中删除了 LEAP。 这是以前的默认配置，因此如果你启用了 EAP，你可能需要更新 `/etc/raddb/mods-available/eap` 并删除`leap { ... }` 行。

[[FAQ Index](https://www.openbsd.org/faq/index.html)] | [[6.8 -> 6.9](https://www.openbsd.org/faq/upgrade69.html)]