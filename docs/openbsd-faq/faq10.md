# OpenBSD FAQ - 系统管理  

## 安全更新  
当一个关键性的Bug被发现后，修复将会被尽可能快地提交到当前源码树中（并在[快照构建](https://www.openbsd.org/faq/faq5.html#Flavors)中可用）。该修复将会以[补丁(errata)](https://www.openbsd.org/errata.html)的形式被向后移植至最新的两个OpenBSD发行版中并且相关细节将会被发送到[公告(announce)](https://lists.openbsd.org/cgi-bin/mj_wwwusr?func=lists-long-full&extra=announce)邮件列表中。  
有以下获得这些修复的方式：

- 应用二进制补丁（在amd64，arm64，i386架构上可用）
[syspatch(8)](https://man.openbsd.org/syspatch)工具可以在任何一个支持的OpenBSD发行版中为任意文件升级安全或可靠性补丁。这是最快速也是最简单地将基础系统更新到最新的方式  
- 升级系统到稳定（stable）版本
通过[CVS](https://www.openbsd.org/anoncvs.html)拉取（或升级）您的源代码树，然后[重新编译](https://www.openbsd.org/stable.html)内核和用户程序。  
- 分别为受影响的文件应用补丁
虽然通过应用在补丁页上的修复往往要比通过CVS的检出和重构建更快一些，但是这种方式没有亘古不变的方法来做到这一点。有时候你必须为一个应用打补丁并且重新编译之，有时还要更麻烦一些。  
- 升级到当前（current）版本的系统
由于所有的修复都被应用到当前版本的代码库中，将系统升级到最新的快照版本（snapshot）是一个不错的一次性完成所有修复的方式。但是快照版本的系统并不适用于所有人。

为通过[包](https://www.openbsd.org/faq/faq15.html)安装的第三方软件提供的安全更新通常仅仅被向后移植到最近的发行版。为了应用它们，进行以下操作之一：

- 使用二进制包（在amd64，arm64，i386，sparc64上可用）
为发行版（release）和稳定版（stable）系统提供的更新的二进制应用包被提供来定位安全问题或者进行其他主要的修复。您可以带有-u选项调用[pkg_add(1)](https://man.openbsd.org/pkg_add)来获得更新的软件包。  
- 使用稳定版的ports源码树
拉取（或者更新）您的ports源码树，运行``/usr/ports/infastructure/bin/pkg_outdated``脚本来列出需要重新构建的软件包，然后在受到影响的port软件包目录中调用``make update``。某些情况下，已经存在的ports软件包需要在重新构建前被卸载。  
- 将您的系统升级到当前版本
提供给当前版本系统的软件包都会被定期重新构建，这些新的软件包将会包含全部安全补丁。  

考虑跟进[ports-changes](https://lists.openbsd.org/cgi-bin/mj_wwwusr?func=lists-long-full&amp;extra=ports-changes)邮件列表来收到关于port软件包更新的提醒。  
## 系统守护进程  
系统守护进程（换而言之，“服务”）由[rc(8)](https://man.openbsd.org/rc)脚本通过[rc.d(8)](https://man.openbsd.org/rc.d)启动，停止和控制。  
大多数随OpenBSD发布的守护进程和服务由在[/etc/rc.conf(8)](https://man.openbsd.org/rc.conf.8)定义的变量在启动时控制。你将会看到类似以下的内容：
```
httpd_flags=NO
```
这表示[httpd(8)](https://man.openbsd.org/httpd.8)不会在启动时由[rc(8)](https://man.openbsd.org/rc.8)启动。每一行都含有对于此守护进程或服务的惯用例描述。  
不要直接更改[rc.conf(8)](https://man.openbsd.org/rc.conf)文件。取而代之地，使用[rcctl(8)](https://man.openbsd.org/rcctl.8)工具来维护``/etc/rc.conf.local``文件。这样可以简化将来的系统升级，因为更改都发生在一个不会在升级时被修改的文件中。  
例如，为了启动[apmd(8)](https://man.openbsd.org/apmd.8)守护进程来进行CPU监控，可以进行以下操作：
```
# rcctl enable apmd
# rcctl set apmd flags -A
# rcctl start apmd
```

## 以其他用户身份执行命令  
[doas(1)](https://man.openbsd.org/doas.1)工具给予系统管理员授权特定用户以其他用户的身份运行某些命令的方式。普通用户可以运行管理员命令，而只需要验证自己的身份，不必得知root密码。  
例如，如果doas工具被正确配置了，以下的命令将会显示root用户的[crontab(5)](https://man.openbsd.org/crontab.5)文件：
```
$ doas -u root crontab -l
```
[doas(1)](https://man.openbsd.org/doas.1)调用命令的记录默认储存在``/var/log/secure``文件。阅读[doas.conf(5)](https://man.openbsd.org/doas.conf.5)手册页来获得配置样例。  

## 编辑密码文件  
OpenBSD的主要密码文件位于``/etc/master.passwd``，此文件仅root可读。[pwd_mkdb(8)](https://https://man.openbsd.org/pwd_mkdb)工具从主文件中生成通用可读的``/etc/passwd``文件和密码数据库（``/etc/pwd.db``以及``/etc/spwd.db``)。文件的格式位于[passwd(5)](https://man.openbsd.org/passwd.5)手册页中。  

## 重置Root密码  
如果root密码被遗忘了，基本的重新获得访问权限的方式是通过启动单用户模式，挂载`/`和`/usr`分区再运行[passwd(1)](https://man.openbsd.org/passwd)来更改root用户的密码。  

- 启动到单用户模式。这一部分过程因平台而异。对于amd64及i386平台而言，[第二阶段引导程序(second stage boot loader)](https://www.openbsd.org/faq/faq14.html#BootAmd64)为提供更改内核启动参数的机会而会等待几秒钟时间。在这里可以向[boot(8)](https://man.openbsd.org/OpenBSD-current/i386/boot)传入-s参数：
```
probing: pc0 com0 com1 mem[638K 1918M a20=on]
disk: hd0+ hd1+
>> OpenBSD/amd64 BOOT 3.33
boot> boot -s
```   
- 挂载分区。``/``和``/usr``都需要以可读写方式被挂载。
```
# fsck -p / && mount -uw /
# fsck -p /usr && mount /usr
```   
- 更改root密码。由于系统正运行在单用户模式下，你已经拥有了root特权，所以你不会再被要求提供当前密码。
```
# passwd
```   
- 启动到多用户模式。这可以通过按下CTRL-D组合键继续普通启动进程或者键入``reboot``来完成。  

## 时钟同步  
[OpenNTPD](https://www.openntpd.org/)是一个安全、简单且NTP兼容的在你的计算机上提供准确时间的方法。[ntpd(8)](https://https://man.openbsd.org/ntpd)守护进程默认被激活且将会基于从NTP同伴收到的数据设置系统时间。一旦时间被正确地设置，其将会通过在[ntpd.conf(5)](https://man.openbsd.org/ntpd.conf)中配置的时间服务器高精度地维持。在启动时，``ntpd``只会向前调整时间。如果您的时间已经被向后调整了，请使用[date(1)](https://man.openbsd.org/date)手动调整时间。  
若要将OpenNTPD作为服务器，请在[ntpd.conf(5)](https://man.openbsd.org/ntpd.conf)文件中加入一行``listen on *``并重新启动守护进程。您也可以指定其仅监听某一个特定的地址或者接口。  
当您将[ntpd(8)](https://man.openbsd.org/ntpd)用于监听时，其他设备可能不能立刻同步它们的时间。这是因为直到本地时钟以一个合理的稳定水平被同步后，ntpd才会提供时间信息。一旦本地时钟达到此水平，一个"clock now synced"（时间已被同步）信息将会被写入``/var/log/daemon``。

## 时区  
默认OpenBSD假设您的硬件时间被设置为协调世界时（UTC）而不是本地时间。这在[多启动(multibooting)](https://www.openbsd.org/faq/faq4.html#Multibooting)时会产生问题。许多其他的操作系统可以进行相同的配置进而一同解决问题。  
如果将硬件时间设置为UTC时间确实是一个问题，您可以通过[sysctl.conf(5](https://man.openbsd.org/sysctl.conf)更改OpenBSD的默认行为。例如，将下面这一行写入``/etc/sysctl.conf``可以配置OpenBSD将硬件时间视为美国东部标准时间（慢于UTC时间5个小时，即减去300分钟）：
```
kern.utc_offset=-300
```   
查看[sysctl(2)](https://man.openbsd.org/sysctl.2)手册页来获取更多信息。  
请注意硬件时间必须在OpenBSD启动前已经按照以上的偏移量被设置，否则系统时间将会在启动时被不正确地调整。  
一般地，时区（time zone）在安装过程中被设置。如果你需要更改时区，你可以创建一个对在``/usr/share/zoneinfo``中适当时区文件的符号链接。例如，以下命令可以将设备设置为使用EST5EDT时区作为新的本地时区。  
```
# ln -fs /usr/share/zoneinfo/EST5EDT /etc/localtime
```
亦见[date(1)](https://man.openbsd.org/date)的手册页。  

## 字符集（Character Sets）及区域  
OpenBSD基础系统完全支持ASCII字符集和编码，并且部分支持Unicode字符集的部分UTF-8编码。基础系统不支持其他编码，但是ports可以被用来处理它们。UTF-8支持的程度和默认编码配置受程序与库的影响很大。  
为了在支持的地方使用Unicode字符集，将``LC_CTYPE``环境变量的值设置为``en_US.UTF-8``：

- 如果您通过[xenodm](https://man.openbsd.org/xenodm)登陆，请在启动窗口管理器(window manager)前向``~/.xsession``文件添加``export LC_CTYPE="en_US.UTF-8"``。对于详细的解释，请见[自定义X系统](https://www.openbsd.org/faq/faq11.html#CustomizingX)。  
- 如果您通过文本控制台(text console)登录，向您的``~/.profile``文件中添加``export LC_CTYPE="en_US.UTF-8"``。文本控制台的UTF-8编码支持尚在开发之中，因此一些非ASCII字符可能显示不正确。  

当您通过[ssh(1)](https://man.openbsd.org/ssh)登录到远程系统时，``LC_TYPE``环境变量不会从远程系统传播至本地。您必须确保本地终端的字符编码在连接前已被设置为远程系统所用的字符编码。如果远程系统的字符编码未知或者不被OpenBSD所支持，请确保您正使用默认的[xterm(1)](https://man.openbsd.org/xterm)配置并在连接后在远程Shell中将字符编码设置为``LC_CTYPE=en_US.UTF-8``。  
OpenBSD基础系统完全忽略除了``LC_CTYPE``之外的一切地区相关环境变量；即使``LC_ALL``与``LANG``也仅仅影响字符编码。一些ports源码包可能会使用其他的``LC_*``环境变量，但是这种行为以及将``LC_CTYPE``环境变量设置为除了``C``，``POSIX``或``en_US.UTF-8``之外的任何值都是不被提倡的。  

## 使用S/Key  
S/Key是一套“一次性密码（one-time password）”验证系统。它通过一个哈希函数（hash function）从用户的密钥生成一个一次性密码并接受来自服务器的验证。这些哈希函数包括：[md5](https://man.openbsd.org/md5)，[sha1](https://man.openbsd.org/sha1)与[rmd160](https://man.openbsd.org/rmd160)。  
**警告**：一次性密码系统**仅仅**保护验证信息。窃听者仍然可以访问私有信息。此外，如果您正在访问一个安全的系统A，建议您通过另一个可信的系统B来进行这一点。这样可以确保没有人能够通过记录您的击键或者记录、伪造您的终端设备来得到对系统A的访问权限。  

### 设置S/Key  
在开始之前，``/etc/skey``目录必须被创建。如果这个目录不存在，通过以下命令以超级用户身份创建它：
```
# skeyinit -E
```  
然后使用[skeyinit(1)](https://man.openbsd.org/skyeinit)来初始化您的S/Key。您首先需要输入您的登录密码，此后您会被要求设置您的S/Key口令，此口令必须至少10字符长。  
```
$ skeyinit
Password:
[Adding ericj with md5]
Enter new secret passphrase:
Again secret passphrase:

ID ericj skey is otp-md5 100 oshi45820
Next login password: HAUL BUS JAKE DING HOT HOG
```
请注意在最后两行中的信息。程序用来创建您的S/Key密钥的算法是[otp-md5(1)](https://man.openbsd.org/otp-md5)，序列号是100并且密钥是``oshi45820``。这六个短短的单词``HAUL BUS JAKE DING HOT HOG``构成了具有序列号100的S/Key密钥。

### 生成S/Key密码  
为了为下一次登录生成S/Key密码，使用[skeyinfo(1)](https://man.openbsd.org/skeyinfo)命令来找到我们要输入的密钥。  
```
$ skeyinfo -v
otp-md5 95 oshi45820
$ otp-md5 95 oshi45820
Enter secret passphrase:
NOOK CHUB HOYT SAC DOLE FUME
```
为了生成一组S/Key密钥，运行：
```
$ otp-md5 -n 5 95 oshi45820
Enter secret passphrase:
91: SHIM SET LEST HANS SMUG BOOT
92: SUE ARTY YAW SEED KURD BAND
93: JOEY SOOT PHI KYLE CURT REEK
94: WIRE BOGY MESS JUDE RUNT ADD
95: NOOK CHUB HOYT SAC DOLE FUME
```   

### 使用S/Key密钥登录 
这是一个使用S/Key登录到在``localhost``的ftp服务器的示例。为了启用S/Key登录，您需要在您的登录名后添加``:skey``后缀。  
```
$ ftp localhost
Connected to localhost.
220 oshibana.shin.ms FTP server (Version 6.5/OpenBSD) ready.
Name (localhost:ericj): ericj:skey
331- otp-md5 93 oshi45820
331 S/Key Password: JOEY SOOT PHI KYLE CURT REEK
[...]
230 User ericj logged in.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> quit
221 Goodbye.
```  
相似的，对于[ssh(1)](https://man.openbsd.org)：  
```
$ ssh -l ericj:skey localhost
otp-md5 91 oshi45821
S/Key Password: SHIM SET LEST HANS SMUG BOOT
Last login: Thu Apr  7 12:21:48 on ttyp1 from 156.63.248.77
$
```   

## 目录服务  
OpenBSD可以被用做存储用户凭据，用户组信息和其他网络相关信息的数据库。  
当然，您也可以在OpenBSD上使用各种各样的目录服务。但是YP是唯一一个可以直接从类似[getpwent(3)](https://man.openbsd.org/getpwent)，[getgrent(3)](https://man.openbsd.org/getgrent)，[gethostbyname(3)](https://man.openbsd.org/gethostbyname)等的标准C库函数中被调用的。因此，如果您将您的数据保存在一个YP数据库中，您不再需要将它拷贝到类似[master.passwd(5)](https://man.openbsd.org/master.passwd)的文件中就可以直接使用它，例如验证系统用户。  
YP是一个Sun Microsystems NIS（网络信息系统，Network Information System）兼容的目录服务。请见[yp(8)](https://man.openbsd.org/yp)来得到可用的手册页的总览。请仔细阅读，因为一些其他包含目录服务的操作系统可能使用了相似的名字但是却是不兼容的，例如NIS+。  
为了使用除YP之外的目录服务，您可以从目录中将数据抽取到本地配置文件中，或者使用一个到目录的YP前端。例如，您可以使用``sysutils/login_ldap``port实现前者，而[ypldap(8)](https://man.openbsd.org/ypldap)守护进程提供了后者。  
对于一些应用而言，简单的在几台设备之间使用类似[rdist(1)](https://man.openbsd.org/rdist)，[cron(8)](https://man.openbsd.org/cron)，[scp(1)](https://man.openbsd.org/scp)或``rsync``（在ports源中可用）的工具同步少量配置文件也可以被看作一个简单且健壮的替代方案。  

### YP安全策略  
为了兼容性因素，所有构建金OpenBSD的YP实现中的安全特性默认都被切换为*off*（即关闭）。即使所有安全特性全部被打开，NIS协议由于两个原因依然是不安全的：所有的数据，包括像密码哈希值一样的敏感数据都在网络中以明文传输，并且服务器和客户端都不能可靠地验证对方的身份。  
因此，在设置任何YP服务器前，您应当考虑这些故有的安全缺点是否在您的应用环境中是可接受的。特别地，如果可能的攻击者能够在物理上访问您的网络，YP的缺点就更加显著了。任何拥有在您传输IP流量的网段中的设备的root权限的人都可以绑定（bind）到您的YP域并访问其数据。在一些情况下，将YP数据通过SSL或者IPSec通道传输可能是一个可用的选项。  

## 设置一个YP服务器  
一个YP服务器服务于一组被叫做“域”（domain）的客户端。您应该首先选择一个域名（domain name）；它应该是一个不以任何方式与DNS域名相关的任意字符串。选择一个像是“Eepoo5vi”的随机名字能够略微增强安全性，尽管这种安全效果主要来源于名字的模糊性。在您需要维护几个不同的YP域时，使用类似“sales”，“marketing”和“research”的描述性名字可能是更好的选择，这样可以预防系统管理员因为模糊性而产生的问题。而且请注意：一些版本的SunOS要求使用主机的DNS域名，所以您的选择可能在包含这些主机的网络中受到限制。  
使用[domainname(1)](https://man.openbsd.org/domainname)工具来设置域名，并将它添加到[defaultdomain(5)](https://man.openbsd.org/defaultdomain)文件中来使其在系统启动时被自动设置。  
```
# echo "puffynet" > /etc/defaultdomain
# domainname `cat /etc/defaultdomain`
```
使用以下交互式命令来初始化YP服务器：
```
# ypinit -m
```
此时，可能还没有必要设置次服务器（slave server）。为了添加次服务器，您可以此后再次带有``-u``选项运行[ypinit(8)](https://man.openbsd.org/ypinit)命令。为每一个域至少设置一台次服务器对于避免服务中断是一个有用的方法。例如，如果主服务器（master server）宕机或丢失网络连接，试图访问YP映射（YP map）的客户端进程将会无限期阻塞直到它们收到请求的信息。因此，YP服务中断通常会导致直到YP服务恢复，客户端主机完全不可用。  
决定储存源文件的位置来生成您的YP映射。保持服务器配置文件与被用于提供服务的配置文件分离有助于控制被共享和不会被共享的信息，所以默认的``/etc/``目录通常不是最好的选择。  
更改源目录最大的不便利之处是您不能再使用像是[user(8)](https://man.openbsd.org/user)和[group(8)](https://man.openbsd.org/group)的工具添加，删除和修改该用户。取而代之地，您必须使用一个文本编辑器来编辑它们。  
要定义源目录，修改文件``/var/yp/`domainname`/Makefile``并更改``DIR``变量，例如：
```
DIR=/etc/yp/src/puffynet
```  
考虑自定义在``/var/yp/`domainname`/Makefile``中的其它变量。细节请见[Makefile.yp(8)](https://man.openbsd.org/Makefile.yp)。  
例如，即使在您使用默认的源目录``/etc``时，您通常在客户端主机上也不需要所有存在在服务器上的用户和组。特别地，不共享root账户并保密root密码的哈希值经常是有益于安全的。检查``MINUID``，``MAXUID``，``MINGID``和``MAXGID``的值并根据您的需要调整它们。  
如果您的YP客户端全部运行在OpenBSD或FreeBSD系统上，通过在`/var/yp/`domainname`/Makefile``中设置```UNSECURE=""``来从``passwd``映射中移除加密的密码。  
不再建议曾经直接修改``/var/yp/Makefile.yp``的做法。对该文件的修改仅影响修改之后初始化的域，却不影响在此之前已经初始化的域。所以这是一种容易出错的方式：您正冒着期望的更改不起作用和忘记它们已对从未期望着生效的域产生影响的风险。  
您可以创建源目录并且向其中写入您需要的配置文件。详见[Makefile.yp(8)](https://man.openbsd.org/Makefile.yp)来了解YP映射需要哪些源文件。参照[passwd(5)](https://man.openbsd.org/passwd)，[group(5)](https://man.openbsd.org/group)，[hosts(5)](https://man.openbsd.org/hosts)等等以及查阅``/etc/``中的例子来查看单独的配置文件格式。  
使用以下的命令创建您初始版本的YP映射：
```
# cd /var/yp
# make
```  
不要在意现在来自[yppush(8)](https://man.openbsd.org/yppush)的错误信息。YP服务器还没有运行。  
YP使用[rpc(3)](https://man.openbsd.org/rpc)（远程过程调用，remote procedure calls）与客户端通信，所以有必要激活[portmap(8)](https://man.openbsd.org/portmap)。为了完成此目的，使用[rcctl(8)](https://man.openbsd.org/rcctl)。
```
# rcctl enable portmap
# rcctl start portmap
```  
考虑使用[securenet(5)](https://man.openbsd.org/securenet)或YP服务器的守护进程的安全特性[ypserv.acl(5)](https://man.openbsd.org/ypsev.acl)。但请意识到这两者都仅提供基于IP的访问控制。因此，它们只能在可能的攻击者既没有对传输您的YP流量的网段中的硬件的物理访问权限也没有对任何连接到这些网络的设备的root访问权限的情况下才能起到作用。  
最终，启动YP服务器守护进程：
```
# rcctl enable ypserv
# rcctl start ypserv
```
为了测试新的服务器，考虑让其作为自己的客户端，按照下一节的第一部分。在您不希望服务器使用它自己的映射时，您可以在测试后使用以下命令禁用客户端部分：
```
# rcctl stop ypbind
# rcctl disable ypbind
```  
记得每一次您修改了源为YP映射的文件后，您必须重新生成您的YP映射。  
```
# cd /var/yp
# make
```  
这将更新在``/var/yp/`domainname```中的一切数据库文件，只有一个例外：``ypservers.db``，该文件列出了与该域关联的一切YP主服务器和次服务器并由``ypinit -m``命令直接创建且仅由``ypinit -u``命令修改。当您意外地删除了它时，运行``ypinit -u``来从修改记录中重新创建它。  

## 设置一个YP客户端  
设置一个YP客户端主要包含独立的两部分。首先，您必须启动客户端的YP守护进程。完成下面的步骤将会使您能够从YP服务器访问数据，但是这些数据不会被系统所使用。  
像是在服务端一样，您必须设置域名并激活端口映射器（portmapper）：
```
# echo "puffynet" > /etc/defaultdomain
# domainname `cat /etc/defaultdomain`
# rcctl enable portmap
# rcctl start portmap
```  
我们建议您在``/etc/yp`domainname```配置文件中提供一个YP服务器的列表。否则，YP客户端守护进程将使用网络广播来查找它自己所属的域的服务器。显式指定服务器使得服务更加健壮也略微减少了被攻击的可能性。如果您还没有设置任何从服务器，请仅将主服务器的主机名（host name）写入``/etc/yp/`domainname```文件中。  
通过[ypbind(8)](https://man.openbsd.org/ypbind)激活和启动YP客户端守护进程。  
```
# rcctl enable ypbind
# rcctl start ypbind
```   
如果一切运行良好，您应该能够使用[ypcat(1)](https://man.openbsd.rog/ypcat)向YP服务器查询并得到您的passwd映射。  
```
# ypcat passwd
bob:*:5001:5000:Bob Nuggets:/home/bob:/usr/local/bin/zsh
...
```  
另一个用来调试您的YP设置的有用工具是[ypmatch(1)](https://man.openbsd.org/ypmatch)。  
配置YP客户端的第二部分需要编辑本地配置文件来使不同的系统设施使用特定的YP映射。不是所有的服务器共享的所有标准映射都能被操作系统所使用，同时一些服务器也提供附加的非标准映射，但是您并非必须使用它们的全部。哪些可用的映射应该或不应该使用，以及应该出于什么目的被使用完全取决于客户端主机系统的管理员。  
阅读[Makefile.yp(8)](https://man.openbsd.org/Makefile.yp)来得到一个包含标准YP映射和它们标准用法的列表。  
如果您想要引入来自YP域的全部用户账号，在主密码文件中添加默认YP标记（default YP marker）并重新构建密码数据库：
```
# echo '+:*::::::::' >> /etc/master.passwd
# pwd_mkdb -p /etc/master.passwd
```   
关于详细的挑选要引入的用户账户的方式，请见[passwd(5)](https://man.openbsd.org/passwd.5)的手册页。使用[id(1)](https://man.openbsd.org/id)工具来测试选中的账户是否真正生效。  
如果您想要在客户端系统上引入来自YP域的全部用户组，在用户组配置文件中添加默认YP标记：  
```
# echo '+:*::' >> /etc/group
```
