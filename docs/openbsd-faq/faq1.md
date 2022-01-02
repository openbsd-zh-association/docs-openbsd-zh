# OpenBSD FAQ - 对 OpenBSD 的简介  

## 关于 OpenBSD

[OpenBSD 项目](https://openbsd.org/index.html)提供了一个自由可用的基于 4.4BSD 系统的多平台 UNIX 系操作系统。我们的[目标](https://www.openbsd.org/goals.html)聚焦于正确性、安全性、标准化和可移植性。  

### 为什么我要使用OpenBSD？  

这是我们认为 OpenBSD 是一个很有用操作系统的原因：

- OpenBSD 可以运行在许多不同的硬件平台上  
- OpenBSD 在无数的源代码审查后，被各安全专家视为最[安全](https://www.openbsd.org/security.html)的 UNIX 系操作系统  
- OpenBSD 是一个具有免费且具有完整特性的 UNIX 系操作系统，并同时以源代码和二进制格式提供  
- OpenBSD 集成了最先进适用的安全技术，可以被用作[构建防火墙](https://www.openbsd.org/faq/pf/example1.html)以及在分布式环境中构建私有网络服务  
- OpenBSD 受益于在各个区域强大的持续开发进程，提供了与新兴技术、国际化开发者社区和最终用户合作的机会  
- OpenBSD 努力最小化自定义和微调的需求。对于庞大的主要用户群体而言，OpenBSD **就在**他们的硬件上为应用而良好工作  

## OpenBSD 真的是自由的吗？  

OpenBSD 完全是自由的。二进制文件是自由的，源代码也一样。OpenBSD 的所有组成部分都有为自由的再发行提供的合理的版权许可条约。更多关于 OpenBSD 的版权政策可以[在此](https://www.openbsd.org/policy.html)被找到。  

OpenBSD 的主要维护者自掏腰包维护此项目。这包括为他们为编程花费的时间，用来支持许多移植的设备，用来为您发行 OpenBSD 所用的网络资源以及解答问题和研究 Bug 报告的时间。OpenBSD 的开发者们并不富有，不过小小的时间，设备以及资源的投入也可以产生极大的作用。

### 基本系统中包含什么？ 

OpenBSD发行时带有大量的第三方软件包，包括：

- [X.org](https://www.x.org/wiki)  
- [LLVM/Clang](https://clang.llvm.org/)  
- [GCC](https://gcc.gnu.org/)  
- [Perl](https://www.perl.org/)  
- [NSD](https://www.nlnetlabs.nl/projects/nsd) 以及 [Unbound](https://www.nlnetlabs.nl/projects/unbound)  
- [ncurses](https://www.gnu.org/software/ncurses)  
- [binutils](https://www.gnu.org/software/binutils)  
- [gdb](https://www.gnu.org/software/gdb/gdb.html)  
- [libfido2](https://github.com/Yubico/libfido2)  

OpenBSD 团队经常为第三方产品提供补丁，这通常是为了提升代码的安全性或是质量。很多[自制的软件](https://www.openbsd.org/innovations.html)也被包含其中。附加的应用可以作为 [package](https://www.openbsd.org/faq/faq15.html) 而可用。  

### 为什么 xxx 被包含了/没被包含？

人们经常询问为什么某个软件被包含或者没有被包含在 OpenBSD 中。答案主要基于两个点：开发者们的愿望和与此项目[目标](https://www.openbsd.org/goals.html)的兼容性。授权协议经常是最大的问题：我们希望 OpenBSD 保持对全世界任何人对于任何目的的可用性。  

### 下一个发行版本何时发布？  

OpenBSD 团队大约每六个月发布一个新版本，预计时间为五月和十一月。更多关于开发周期的信息可以[在此](https://www.openbsd.org/faq/faq5.html#Flavors)被找到  

## 硬件支持

OpenBSD 可以运行在以下[平台](https://www.openbsd.org/plat.html)上：

- [alpha](https://www.openbsd.org/alpha.html)  
- [amd64](https://www.openbsd.org/amd64.html)  
- [arm64](https://www.openbsd.org/arm64.html)  
- [armv7](https://www.openbsd.org/armv7.html)  
- [hppa](https://www.openbsd.org/hppa.html)  
- [i386](https://www.openbsd.org/i386.html)  
- [landisk](https://www.openbsd.org/landisk.html)  
- [luna88k](https://www.openbsd.org/luna88k.html)  
- [macppc](https://www.openbsd.org/macppc.html)  
- [octeon](https://www.openbsd.org/octeon.html)  
- [powerpc64](https://www.openbsd.org/powerpc64.html)  
- [riscv64](https://www.openbsd.org/riscv64.html)  
- [sparc64](https://www.openbsd.org/sparc64.html)  

详细的硬件支持细节位于各自的平台页面上。  

## 手册页  
OpenBSD带有大量man页面形式的手册页。它们是OpenBSD权威的信息源，因此相当多的工作被进行来确保他们最新且精确无误。对系统进行更改的开发者应当随他们对代码的更改更新手册页。而对于用户，在寻求帮助前也应当先查看手册页。  
这是一个一些对于新用户有帮助的手册页的列表：

- [afterboot(8)](https://man.openbsd.org/afterboot) - 在第一次完全启动后应当检查的东西  
- [help(1)](https://man.openbsd.org/help) - 对于新用户和管理员的帮助  
- [hier(7)](https://man.openbsd.org/hier) - 文件系统的结构  
- [man(1)](https://man.openbsd.org/man) - 显示手册页  
- [adduser(8)](https://man.openbsd.org/adduser) 以及 [rmuser(8)](https://man.openbsd.org/rmuser) - 添加和移除新用户  
- [reboot(8)](https://man.openbsd.org/reboot), [halt(8)](https://man.openbsd.org/halt) 以及 [shutdown(8)](https://man.openbsd.org/shutdown) 停止和重启系统  
- [syspatch(8)](https://man.openbsd.org/syspatch) - 应用安全和可靠性更新  
- [sysupgrade(8)](https://man.openbsd.org/sysupgrade) - 更新到下一个 OpenBSD 发行版或者一个更新的快照版  
- [dmesg(8)](https://man.openbsd.org/dmesg) - 重新显示内核启动信息  
- [doas(1)](https://man.openbsd.org/doas) - 以另一个用户的身份运行命令  
- [tmux(1)](https://man.openbsd.org/tmux) - 终端复用器  
- [ifconfig(8)](https://man.openbsd.org/ifconfig) - 配置网络接口参数  
- [ftp(1)](https://man.openbsd.org/ftp) - 从网络上下载文件(支持 FTP/HTTP/HTTPS 协议)  
- [login.conf(5)](https://man.openbsd.org/login.conf) - 登陆配置文件的格式  
- [sendbug(1)](https://man.openbsd.org/sendbug) - 报告一个你找到的 Bug  

全部的OpenBSD手册页可以在 [man.openbsd.org](https://man.openbsd.org/) 网站或 `man70.tgz` 文件中被找到。  

总之，如果您知道一个命令或者手册页的名字，你可以通过运行 ``man 命令`` 来阅读它。如果你不知道命令的名字，或者如果 ``man 命令`` 没有找到手册页，你可以通过运行 ``apropos 一些东西`` 或者 ``man -k 一些东西`` 来查找数据库，“一些东西”就是一个很可能在您要查找的手册页标题中出现的单词。 

```
$ apropos "time zone"
tzfile(5) - time zone information
zdump(8) - time zone dumper
zic(8) - time zone compiler
```

这些括号里面数字表示该手册页存在的章节。在一些情况下，您可能会遇到分散在不同章节中却有相同标识符的手册页。例如，假设您想要知道cron守护进程的配置文件格式。只要您知道您想要的手册页所在的章节，您可以运行 ``man n 命令``，其中n是手册章节序号。

```
$ man -k cron
cron(8) - clock daemon
crontab(1) - maintain crontab files for individual users
crontab(5) - tables for driving cron
$ man 5 crontab
```

## 邮件列表  
OpenBSD 项目包含几个你可以订阅和跟进的邮件列表。比较受欢迎的邮件列表有：

- announce - 公告和安全警报  
- bugs - 通过[sendbug(1)](https://man.openbsd.org/sendbug)收到的bug反馈及相关讨论  
- misc - 总的用户问题和解答  
- ports - 讨论ports源码树
- source-changes - 自动构建的CVS源码树改动
- tech - OpenBSD开发者和高级用户对技术话题的讨论  

在向任何邮件列表发送问题之前，请检查邮件列表的归档中已经被无数次问过的问题。即使您可能是第一次遇到此问题，邮件列表中的其他订阅者可能在上周刚见过这个问题几次，不太希望再见到它。如果询问的问题可能与硬件相关，请一定在邮件中包含完整的 [dmesg(8)](https://man.openbsd.org/dmesg) 输出。  
你可以找到几个邮件列表的归档，其他的指导和更多的信息位于[邮件列表页面](https://www.openbsd.org/mail.html)上。对于邮件列表的订阅可以很容易通过[Web界面](https://lists.openbsd.org/)完成。  

## （从其他系统）迁移到 OpenBSD  

如果您已经读过其他关于 Unix 类系统的[优秀的著作](https://www.openbsd.org/books.html)，理解了 Unix 哲学并也将你的知识面扩展到了某一个特定平台上，您将会对 OpenBSD 感到非常熟悉。

这里有一些 OpenBSD 和其他 Unix 变种之间最经常被遇到的差别：

- OpenBSD 是一个 BSD 风格的 Unix 系统，紧密遵循这 4.4BSD 的设计思想。Linux 与 Solaris 是 System V 风格的操作系统。一些 Unix 系操作系统混合着 System V 与 BSD 的风格。一个因此经常造成混乱的地方是[启动脚本](https://www.openbsd.org/faq/faq10.html#rc)。OpenBSD 使用[rc(8)](https://man.openbsd.org/rc)系统。  
- OpenBSD 是一个完整的系统，各个部分应当保持同步。它并不是可以被独立升级的、一个内核和一套工具的聚合体。  
- OpenBSD 维护着一套[ports源码树](https://www.openbsd.org/faq/ports/index.html)来提供第三方软件。预编译的[包(packages)](https://www.openbsd.org/faq/faq15.html)由 OpenBSD ports 团队创建和维护。  
- OpenBSD 使用 CVS 来追踪代码更改。OpenBSD 提倡[匿名 CVS](https://www.openbsd.org/anoncvs.html)，其允许任何人在任何时刻提取 OpenBSD 任何版本的完整源代码。其也包含一个 [Web 界面](https://cvsweb.openbsd.org/)。  
- OpenBSD 已经通过了重度不间断的安全性审查来确保代码的质量与安全性。  
- OpenBSD 不支持日志文件系统。相反，我们使用快速文件系统(Fast File System,FFS)的[软同步](https://www.openbsd.org/faq/faq14.html#SoftUpdates)特性。  
- OpenBSD 带有[包过滤器(Packet Filter,PF)](https://www.openbsd.org/faq/pf/index.html)。这意味着网络地址转换(NAT)，队列等待与包过滤由 [pfctl(8)](https://man.openbsd.org/pfctl)，[pf(4)](https://man.openbsd.org/pf) 和 [pf.conf(5)](https://man.openbsd.org/pf.conf) 处理。  
- OpenBSD 的默认 Shell 是 [ksh](https://man.openbsd.org/ksh)，其基于位于公有领域的 Korn Shell。例如 bash 的其他 Shell 可以从[包](https://www.openbsd.org/faq/faq15.html)中被添加。  
- 设备由驱动程序命名，而不是通过设备类型命名。换而言之，没有 ``eth0`` ``eth1`` 设备。对于一个 Intel PRO/1000 以太网卡，命名可能是 ``em0``；对于 Broadcom BCM57xx 或者 BCM590x 以太网设备，命名可能是 bge0；对于一个 RaLink 无线设备，命名可能是 ral0，诸如此类。  
- OpenBSD/i386，amd64 和其他几个平台使用双层磁盘分区系统，其中第一层是 [fdisk](https://www.openbsd.org/faq/faq14.html#fdisk) BIOS 可见的分区，第二层是盘标（[disklabel](https://www.openbsd.org/faq/faq14.html#disklabel)）  
- 一些其他的操作系统鼓励你为你的机器定制内核。OpenBSD 用户则被鼓励着使用开发者提供的标准通用内核。  

## 报告 Bug  

报告 Bug 是最终用户最重要的责任之一。为了确定较严重的问题，需要极其详细的有关细节。例如，以下即是一份恰当的 bug 报告：

```
From: user@example.com
To: bugs@openbsd.org
Subject: 3.3-beta panics on a SPARCStation2

OpenBSD 3.2 installed from an official CD-ROM installed and ran fine
on this machine.

After doing a clean install of 3.3-beta from a mirror, I find the
system randomly panics after a period of use, and predictably and
quickly when starting X.

This is the dmesg output:

[...]

This is the panic I got when attempting to start X:

panic: pool_get(mclpl): free list modified: magic=78746572; page 0xfaa93000;
 item addr 0xfaa93000
Stopped at      Debugger+0x4:   jmpl            [%o7 + 0x8], %g0
https://www.openbsd.org/ddb.html describes the minimum info required in bug
reports. Insufficient info makes it difficult to find and fix bugs.
ddb> trace
[...]

Thank you!

```

另见[此页面](https://www.openbsd.org/report.html)来得到关于创建和提交 Bug 报告的细节。良好的 Bug 报告应该包括关于发生了什么，您计算机的精确配置和重现问题的方式。只要可能，请使用 [sendbug(1)](https://man.openbsd.org/sendbug) 来报告您的问题。否则，请至少包括您系统的 [dmesg(8)](https://man.openbsd.org/dmesg) 输出。[sendbug(1)](https://man.openbsd.org/dmesg) 命令需要你的系统能够发送邮件。  

OpenBSD 邮件服务器使用 [spamd(8)](https://man.openbsd.org/spamd) 实现灰名单垃圾邮件处理，所以在邮件服务器接受您的 Bug 报告前可能有半个小时左右的时间。请耐心等待。  
在提交一个 Bug 反馈之后，您可能会被开发者联系来寻求附加的信息或者是希望您测试的补丁。您也可以关注 [bugs@openbsd.org](bugs@openbsd.org) 邮件列表的归档 - 详情见[邮件列表](https://www.openbsd.org/mail.html)。  

## 支持本项目  

我们对曾经为 OpenBSD 项目作出贡献的人和组织表示极大的感激。 

OpenBSD 项目总是需要来自社区的几种不同种类的支持。如果您觉得 OpenBSD 很有用，您可以找一种方式以为它做出贡献：

- [捐赠资金](https://www.openbsd.org/donations.html)。这个项目总是需要资金来支付设备和网络链接等等。即使是很小的一笔捐赠也能产生深远的影响。  
- [捐赠设备和部分](https://www.openbsd.org/want.html)。这个项目总是需要普通的和特定的硬件。  
- [捐赠您的时间和技术](https://www.openbsd.org/faq/faq5.html#Diff)热爱编写操作系统的程序员很自然总是很受欢迎的，但也有其他的方式可以让人们做出贡献。  
- 跟进[邮件列表](https://www.openbsd.org/mail.html)并为其他人答疑解惑。
- 通过向 [misc@openbsd.org](misc@openbsd.org) 发送新的 FAQ 素材来帮助维护文档。  
- 组织一个本地的[用户组](https://www.openbsd.org/groups.html)并让您的朋友们痴迷于 OpenBSD。
- 通过您的雇主为在工作上使用 OpenBSD 创造一个机会。如果您是一名学生，与您的教授讨论将 OpenBSD 作为一个学习计算机科学或者工程学的工具的可能性。