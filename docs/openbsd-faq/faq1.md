# OpenBSD FAQ - 对OpenBSD的简介  

## 关于OpenBSD
[OpenBSD项目](https://openbsd.org/index.html)提供了一个自由可用的基于4.4BSD系统的多平台UNIX系操作系统。我们聚焦于正确性、安全性、标准化和可移植性。  

### 为什么我要使用OpenBSD？  
这是我们认为OpenBSD是一个很有用操作系统的原因：

- OpenBSD可以运行在许多不同的硬件平台上  
- OpenBSD在无数的源代码审查后，被各安全专家视为最安全的UNIX系操作系统  
- OpenBSD是一个具有免费且具有完整特性的UNIX系操作系统，并同时以源代码和二进制格式提供  
- OpenBSD集成了最先进适用的安全技术，可以被用作构建防火墙以及在分布式环境中构建私有网络服务  
- OpenBSD受益于在各个区域强大的持续开发进程，提供了与新兴技术、国际化开发者社区和最终用户合作的机会  
- OpenBSD努力最小化自定义和微调的需求。对于庞大的主要用户群体而言，OpenBSD**就在**他们的硬件上为应用而良好工作  

## OpenBSD真的是自由的吗？  
OpenBSD完全是自由的。二进制文件是自由的，源代码也一样。OpenBSD的所有组成部分都有为自由的再发行提供的合理的版权许可条约。更多关于OpenBSD的版权政策可以[在此](https://www.openbsd.org/policy.html)被找到。  
OpenBSD的主要维护者自掏腰包维护此项目。这包括为他们为编程花费的时间，用来支持许多移植的设备，用来为您发行OpenBSD所用的网络资源以及解答问题和研究Bug报告的时间。OpenBSD的开发者们并不富有，不过小小的时间，设备以及资源的投入也可以产生极大的作用。

### 基本系统中包含什么？ 
OpenBSD发行时带有大量的第三方软件包，包括：

- X.org  
- LLVM/Clang  
- GCC  
- Perl  
- NSD以及Unbound  
- ncurses  
- binutils  
- gdb  
- libfido2  

OpenBSD团队经常为第三方产品提供补丁，这通常是为了提升代码的安全性或是质量。很多自制的软件也被包含了。附加的应用可以作为[package](https://www.openbsd.org/faq/faq15.html)而可用。  

### 为什么xxx被包含了/没被包含？
人们经常询问为什么某个软件被包含或者没有被包含在OpenBSD中。答案主要基于两个点：开发者们的愿望和与此项目目标的兼容性。授权协议经常是最大的问题：我们希望OpenBSD保持对全世界任何人对于任何目的的可用性。  

### 下一个发行版本何时发布？  
OpenBSD团队大约每六个月发布一个新版本，预计时间为五月和十一月。更多关于开发周期的信息可以[在此](https://www.openbsd.org/faq/faq5.html#Flavors)被找到  

## 硬件支持  
OpenBSD可以运行在以下平台上：

- alpha  
- amd64  
- arm64  
- armv7  
- hppa  
- i386  
- landisk  
- luna88k  
- macppc  
- octeon  
- powerpc64  
- riscv64  
- sparc64  

详细的硬件支持细节位于各自的平台页面上。  

## 手册页  
OpenBSD带有大量man页面形式的手册页。它们是OpenBSD权威的信息源，因此相当多的工作被进行来确保他们最新且精确无误。对系统进行更改的开发者应当随他们对代码的更改更新手册页。而对于用户，在寻求帮助前也应当先查看手册页。  
这是一个一些对于新用户有帮助的手册页的列表：

- afterboot(8) - 在第一次完全启动后应当检查的东西  
- help(1) - 对于新用户和管理员的帮助  
- hier(7) - 文件系统的结构  
- man(1) - 显示手册页  
- adduser(8) 以及 rmuser(8) - 添加和移除新用户  
- reboot(8),halt(8) 以及 shutdown(8) 停止和重启系统  
- syspatch(8) - 应用安全和可靠性更新  
- sysupgrade(8) - 更新到下一个OpenBSD发行版或者一个更新的快照版  
- dmesg(8) - 重新显示内核启动信息  
- doas(1) - 以另一个用户的身份运行命令  
- tmux(1) - 终端复用器  
- ifconfig(8) - 配置网络接口参数  
- ftp(1) - 从网络上下载文件(支持FTP/HTTP/HTTPS协议)  
- login.conf(5) - 登陆配置文件的格式  
- sendbug(1) - 报告一个你找到的Bug  

全部的OpenBSD手册页可以在man.openbsd.org网站或man70.tgz文件中被找到。  
总之，如果您知道一个命令或者手册页的名字，你可以通过运行``man 命令``来阅读它。如果你不知道命令的名字，或者如果``man 命令``没有找到手册页，你可以通过运行``apropos 一些东西``或者``man -k 一些东西``来查找数据库，“一些东西”就是一个很可能在您要查找的手册页标题中出现的单词。 

```
$ apropos "time zone"
tzfile(5) - time zone information
zdump(8) - time zone dumper
zic(8) - time zone compiler
```

这些括号里面数字表示
