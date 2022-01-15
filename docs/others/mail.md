#  OpenBSD - 邮件列表

邮件列表是 OpenBSD 用户和开发人员之间交流的重要手段。除**公告**外，这些列表均未经过审核。我们有意限制不同邮件列表的数量。这有助于减少交叉发布的数量，并确保将信息分发给广泛的受众。

## 网络礼仪

考虑邮件列表中的其他订阅者。

- **纯文本，每行 72 个字符。**</p>
    许多订阅者和开发人员在基于文本的邮件程序（如 [mail(1)](https://man.openbsd.org/mail)、emacs 或 mutt）上阅读他们的邮件，并且他们经常发现 HTML 格式的邮件（或超过 72 个字符的行）不可读。大多数 OpenBSD 邮件列表会在将 MIME 内容的消息发送到列表的其余部分之前将它们剥离。如果你不使用纯文本，你的邮件将被重新格式化。如果它们不能被重新格式化，它们将被立即拒绝。 唯一允许文件附件的邮件列表是 **bugs**、**ports** 和 **tech** 列表。他们将从其他人的消息中删除。

- **发帖前做好功课。**</p>
    如果你有安装问题，请确保你已阅读相关文档，例如安装目录中的 `INSTALL.*` 文本文件、[FAQ](https://www.openbsd.org/faq/index.html) 和相关手册页（从 [afterboot(8)](https://man.openbsd.org/afterboot) 开始）。 还要检查邮件列表档案。 我们想提供帮助，但我们不想剥夺你宝贵的学习经验，而且没有人希望在一个月内第五次在列表中看到相同的问题。

- **包括一个有用的主题行。**</p>
    主题为空的邮件将被退回到列表管理器，并且需要更长的时间才能显示出来。 在消息中包含相关主题将确保更多人真正阅读你所写的内容。此外，请避免使用过多大写的主题行。 “Help!” 或 “I can't get it to work!” 不是有用的主题行。不要在同一主题上更改主题行。你可能知道它是关于什么的，我们其他每天收到数百条消息的人将不知道。

- **修剪你的签名。**</p>
    将邮件底部的签名行保持在合理的长度。PGP 签名和那些自动地址卡只会很烦人并且被剥离了。法律上的免责声明和忠告也是非常令人讨厌的，而且不适合公开的邮件列表。

- **留在话题上。**</p>
    请保持帖子主题与 OpenBSD 用户相关。

- **包含重要信息。**</p>
    不要用一个无望的不完整的问题浪费大家的时间。除了你之外，没有人拥有解决你的问题所需的信息，提供更多的信息比没有足够的细节要好。所有的问题都应该至少包括 OpenBSD 的[版本](https://www.openbsd.org/faq/faq5.html#Flavors)。任何与硬件相关的问题都应该提到平台(i386, amd64, 等等)并提供完整的 [dmesg(8)](https://man.openbsd.org/dmesg)。不幸的是, 硬件型号并不能说明某台机器或配件的实际内容, 而且对于那些没有确切的机器坐在那里可以轻易辨认的人来说是没有用的。dmesg 的输出告诉我们你的机器里**有什么**，而不是外面有什么贴纸。

- **尊重观点和哲学的差异。**</p>
    聪明的人可能会看到相同的事实并得出截然不同的结论。重复以前不能说服某人的相同观点很少会改变他们的想法，并激怒所有其他读者。

## 垃圾邮件

OpenBSD 邮件列表使用 [spamd(8)](https://man.openbsd.org/spamd) 和 [SpamAssassin](https://spamassassin.apache.org/) 来降低垃圾邮件的数量，但有时也会有一些东西偷偷溜进来——处理它。此外，邮件列表服务器还具有基于正则表达式的规则，可以根据一些常见的垃圾邮件和病毒线索拒绝电子邮件。如果你通过 OpenBSD 邮件列表之一收到垃圾邮件，则无需将副本发送给列表所有者——很可能他已经看到了。另外，请**不要**将通过邮件列表收到的垃圾邮件提交给 [spamcop](https://www.spamcop.net/)，因为这会导致邮件列表服务器被添加到他们的 RBL。对邮件列表中的垃圾邮件进行投诉和评论会适得其反，因为它产生的流量比垃圾邮件本身还多。

请注意，如果你从动态 IP 地址发送邮件，你可能无法投递到邮件列表。 在这种情况下，你应该使用利用 ISP 邮件服务器的智能主机邮件配置。请阅读 [smtpd.conf(5)](https://man.openbsd.org/smtpd.conf) 中的示例以了解如何执行此操作。

## 一般兴趣邮件列表

大多数 OpenBSD 用户都对这些列表感兴趣。

- **announce@openbsd.org** ([Archive](https://marc.info/?l=openbsd-announce)) </p>
    公告和安全建议。

- **misc@openbsd.org** ([Archive](https://marc.info/?l=openbsd-misc)) </p>
    用户问答，一般问题。这是最活跃的列表。请阅读[常见问题解答](https://www.openbsd.org/faq/index.html)和安装文档，并在发布之前查看[如何报告问题](https://www.openbsd.org/report.html)。

- **advocacy@openbsd.org** ([Archive](https://marc.info/?l=openbsd-advocacy)) </p>
    推广使用 OpenBSD。

- **ports@openbsd.org** ([Archive](https://marc.info/?l=openbsd-ports)) </p>
    关于使用和贡献 ports tree 的讨论。

- **misc@opensmtpd.org** ([Archive](https://www.mail-archive.com/misc@opensmtpd.org)) </p>
    关于原生和可移植 OpenSMTPD 的一般讨论、问题和想法。可移植位的补丁应该是 [Github](https://github.com/OpenSMTPD/OpenSMTPD) 上的 pull request。要在那里订阅，请按照 [OpenSMTPD 网站](https://opensmtpd.org/list.html)上的说明进行操作。

- **users@openbgpd.org** ([Archive](https://marc.info/?l=openbgpd-users)) </p>
关于原生和可移植 OpenBGPD 的一般讨论、问题和想法。可移植位的补丁应该是 [Github](https://github.com/openbgpd-portable/openbgpd-portable) 上的 pull request。

## 开发者邮件列表

这些列表用于 OpenBSD 各个方面的技术讨论。它们不适合初学者或普通用户，不适合报告问题（除非你提供一个好的修复程序），也不适合安装问题。如果你对是否应该将消息发布到这些列表中的任何一个有任何疑问，它可能也不适合。请先改用 misc@opensmtpd.org 询问一下。

- **bugs@openbsd.org** ([Archive](https://marc.info/?l=openbsd-bugs)) </p>
    通过 [sendbug(1)](https://man.openbsd.org/sendbug) 发送的错误报告和后续讨论。

- **tech@openbsd.org** ([Archive](https://marc.info/?l=openbsd-tech)) </p>
    为 OpenBSD 开发人员和高级用户讨论技术主题。这不是一个“技术支持”论坛——不要这样使用它。OpenBSD 开发人员通常会通过此列表制作补丁以实现新功能和其他重要变化的补丁，供公众测试。

- **libressl@openbsd.org** ([Archive](https://marc.info/?l=libressl)) </p>
    关于原生和可移植 LibreSSL 的技术讨论。 可移植位的补丁应该是 [Github](https://github.com/libressl-portable/portable) 上的 pull request。

## 报告安全问题

这些私有地址用于向 OpenBSD 团队报告漏洞。

- **security@openbsd.org** </p>
    报告与 OpenBSD 相关的漏洞。

- **openssh@openssh.com**</p>
    报告与 OpenSSH 相关的漏洞。

- **libressl-security@openbsd.org**</p>
    报告与 LibreSSL 相关的漏洞。

- **opensmtpd-security@openbsd.org**</p>
    报告与 OpenSMTPD 相关的漏洞。

## 平台特定邮件列表

这些列表侧重于各个平台上的用户问题和开发。

- **alpha@openbsd.org** ([Archive](https://marc.info/?l=openbsd-alpha)) </p>
    OpenBSD/alpha port

- **arm@openbsd.org** ([Archive](https://marc.info/?l=openbsd-arm)) </p>
    OpenBSD/armv7 和 OpenBSD/arm64 ports

- **hppa@openbsd.org** ([Archive](https://marc.info/?l=openbsd-hppa)) </p>
    OpenBSD/hppa port

- **m88k@openbsd.org** ([Archive](https://marc.info/?l=openbsd-m88k)) </p>
    OpenBSD/luna88k port

- **ppc@openbsd.org** ([Archive](https://marc.info/?l=openbsd-ppc)) </p>
    OpenBSD/macppc 和其他 PowerPC porting 行动 

- **sparc@openbsd.org** ([Archive](https://marc.info/?l=openbsd-sparc)) </p>
    OpenBSD/sparc64 port. 

## CVS 更改邮件列表

每次开发人员提交对 OpenBSD CVS 树的更改时，都会向这些列表的所有订阅者邮寄一条消息，其中包含提交注释。

- **source-changes@openbsd.org** ([Archive](https://marc.info/?l=openbsd-cvs)) </p>
    `src`、`xenocara` 和 `www` 存储库中 CVS 源代码树更改的自动邮件。

- **ports-changes@openbsd.org** ([Archive](https://marc.info/?l=openbsd-ports-cvs)) </p>
    ``ports`` 存储库中 CVS 源代码树更改的自动邮件。

## 镜像相关列表

有关 OpenBSD [镜像](https://www.openbsd.org/ftp.html)的公告和讨论。

- **mirrors-announce@openbsd.org** </p>
    这是一个审核列表，仅用于向 OpenBSD 镜像操作员发布重要公告。

- **mirrors-discuss@openbsd.org**</p>
    有关 OpenBSD 镜像的讨论。

## 通过 Majordomo 管理列表成员

如果你想收到一份完整的列表，其中包含 openbsd.org 上提供的所有邮件列表，请将命令 `lists` 作为邮件正文发送到 [majordomo@openbsd.org](mailto:majordomo@openbsd.org)。

要订阅给定列表，请将邮件发送到 [majordomo@openbsd.org](mailto:majordomo@openbsd.org)，邮件正文为 “subscribe `mailing-list-name`”（其中 `mailing-list-name` 是你首选列表的名称）。

如需进一步的帮助，请将 “help” 作为邮件正文发送至 [majordomo@openbsd.org](mailto:majordomo@openbsd.org)，你将收到概述所有选项的回复。你的域**必须**正确解析，否则邮件将无法通过！

## 通过 Web 管理列表成员资格

你在 OpenBSD 邮件列表中的成员资格也可以通过 [lists.openbsd.org](https://lists.openbsd.org/) 上的 Web 界面进行管理。

## 邮件列表技巧

你可以通过[网页界面](https://lists.openbsd.org/)或通过 [Majordomo](mailto:majordomo@openbsd.org) 来选择许多非常有用的选项。你可以改变你的电子邮件地址，而不必取消订阅和重新订阅，当你去度假时，暂时停止你的信息传递几天，以及更长时间。请你多花一些时间阅读这些选项，你可通过向 [Majordomo](mailto:majordomo@openbsd.org) 发送包含 "help" 作为正文的邮件，或通过[网页界面](https://lists.openbsd.org/)的 "Help" 标签。

例如，如果你要休假两周并且不希望收到数千封电子邮件，你可以在假期期间禁用邮件服务器的邮件传递，并在你预定的返回时自动恢复传递 使用命令：

```
set ALL nomail-14d 
```

这将暂停你对所有邮件列表的订阅 14 天 (`-14d`)。 更多细节和选项可以在 [Majordomo 概览页面](http://lists.openbsd.org/cgi-bin/mj_wwwusr?&user=&passw=&list=GLOBAL&func=help&extra=overview)上看到。

## 摘要

如果你希望查看“摘要”（一段时间内所有消息的合并列表），而不是以“实时”形式单独获取消息，你可以使用以下命令：

```
set misc digest-daily
set source-changes digest-weekly 
```

用于杂项列表（**misc**）的每日摘要和 **source-changes** 列表的每周摘要。是的，你可以在一封 Majordomo 电子邮件中放置多个命令。

## 其他列表

- **pf@benzedrine.ch** (Archive) </p>
    关于 [OpenBSD 包过滤器](https://www.openbsd.org/faq/pf/index.html)的讨论。要订阅，请将消息正文为 “subscribe” 的电子邮件发送到 [pf-request@benzedrine.ch](mailto:pf-request@benzedrine.ch)。 更多信息可以在[这里](https://www.benzedrine.ch/mailinglist.html)找到。

## 非英语列表

几个与 OpenBSD 相关的非英语邮件列表可单独获得。以下是当前已知邮件列表的列表：

- 法语：**blabla@openbsd.fr.eu.org** ([Archive](https://openbsd.fr.eu.org/blabla/)) </p>
    要订阅，请发送消息至 [blabla+subscribe@openbsd.fr.eu.org](mailto:blabla+subscribe@openbsd.fr.eu.org)

- 西班牙语：**OpenBSD-Mexico@googlegroups.com** </p>
    要订阅，请访问以下网址：https://groups.google.com/group/OpenBSD-Mexico

- 乌克兰语：**openbsd@uaoug.org.ua**</p>
    要订阅，请发送空消息至 [openbsd+subscribe@uaoug.org.ua](mailto:openbsd+subscribe@uaoug.org.ua)