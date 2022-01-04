# OpenBSD FAQ - 网络

## 网络配置

OpenBSD 中的网络配置是通过 `/etc` 中的文本文件完成的。这些设置通常最初是在[系统安装过程](https://www.openbsd.org/faq/faq4.html)中配置的。

### 识别和设置网络接口

接口由网卡的类型命名，而不是连接的类型。例如，以下是 Intel 快速以太网网卡的 [dmesg(8)](https://man.openbsd.org/dmesg) 片段：

```
fxp0 at pci0 dev 10 function 0 "Intel 82557" rev 0x0c: irq 5, address 00:02:b3:2b:10:f7
inphy0 at fxp0 phy 1: i82555 10/100 media interface, rev. 4
```

此设备使用 [fxp(4)](https://man.openbsd.org/fxp) 驱动程序并在此处分配编号 0。

[ifconfig(8)](https://man.openbsd.org/ifconfig) 实用程序将显示系统上已识别的网络接口。

```
$ ifconfig
lo0: flags=8049<UP,LOOPBACK,RUNNING,MULTICAST> mtu 33200
        index 3 priority 0 llprio 3
        groups: lo
        inet 127.0.0.1 netmask 0xff000000
fxp0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:02:b3:2b:10:f7
        index 1 priority 0 llprio 3
        media: Ethernet autoselect (100baseTX full-duplex)
        status: active
        inet 10.0.0.38 netmask 0xffffff00 broadcast 10.0.0.255
enc0: flags=0<>
        index 2 priority 0 llprio 3
        groups: enc
        status: active
pflog0: flags=141<UP,RUNNING,PROMISC> mtu 33200
        index 4 priority 0 llprio 3
        groups: pflog
```

此示例仅显示一个物理以太网接口：`fxp0`。该接口配置了 IP，因此值为 `inet 10.0.0.38 netmask 0xffffff00 broadcast 10.0.0.255`。它也配置了 `UP` 和 `RUNNING` 标志。

[netstart(8)](https://man.openbsd.org/netstart) 脚本在引导时使用 [hostname.if(5)](https://man.openbsd.org/hostname.if) 文件配置网络接口，其中 “if” 被每个接口的全名替换。上面的示例将使用文件 `/etc/hostname.fxp0`，其中包含以下选项：

```
inet 10.0.0.38 255.255.255.0
```

这个 `hostname.fxp0` 文件也有一个交互式等效文件：

```
# ifconfig fxp0 10.0.0.38 255.255.255.0
```

默认情况下启用其他几个接口。这些虚拟接口用于提供各种功能。以下手册页描述了它们：

- [enc(4)](https://man.openbsd.org/enc) - 封装接口
- [lo(4)](https://man.openbsd.org/lo) - 环回接口
- [pflog(4)](https://man.openbsd.org/pflog) - 数据包过滤器日志记录接口

你可以使用 [ifconfig(8)](https://man.openbsd.org/ifconfig) 的 `create` 子命令添加其他虚拟接口。

### 默认主机名和网关

`/etc/myname` 和 `/etc/mygate` 文件由 [netstart(8)](https://man.openbsd.org/netstart) 脚本读取。这两个文件由一行字组成，分别指定系统的完全限定域名（fully qualified domain name）和网关主机的地址。`/etc/mygate` 文件不需要在所有系统上都存在。有关更多详细信息，请参阅 [myname(5)](https://man.openbsd.org/myname)。

### DNS 解析

DNS 解析由 [resolv.conf(5)](https://man.openbsd.org/resolv.conf) 文件控制，该文件由 [resolvd(8)](https://man.openbsd.org/resolvd) 管理。

```
$ cat /etc/resolv.conf
search example.com
nameserver 125.2.3.4
nameserver 125.2.3.5
lookup file bind
```

这里默认的域名是 `example.com`，会有两个名称服务器（`125.2.3.4` 和 `125.2.3.5`），并且在名称服务器之前会查询 [hosts(5)](https://man.openbsd.org/hosts) 文件。

### 激活更改

从这里，重新启动或运行 [netstart(8)](https://man.openbsd.org/netstart) 脚本：

```
# sh /etc/netstart
```

如果已经配置了接口，则在运行此脚本时可能会产生一些警告。使用 [ifconfig(8)](https://man.openbsd.org/ifconfig) 确保接口设置正确。

尽管可以在正在运行中的 OpenBSD 系统上完全重新配置网络，但建议在任何重大更改后重新启动。

### 检查路由

可以通过 [netstat(1)](https://man.openbsd.org/netstat) 或 [route(8)](https://man.openbsd.org/route) 检查路由。

```
$ netstat -rn
Routing tables

Internet:
Destination        Gateway            Flags     Refs     Use    Mtu  Prio Iface
default            10.0.0.1           UGS         4       16      -    12 fxp0
224/4              127.0.0.1          URS         0        0  32768     8 lo0
127/8              127.0.0.1          UGRS        0        0  32768     8 lo0
127.0.0.1          127.0.0.1          UH          2       15  32768     1 lo0
10.0.0/24          link#1             UC          1        4      -     4 fxp0
10.0.0.1           aa:0:4:0:81:d      UHL         1       11      -     1 fxp0
10.0.0.38          127.0.0.1          UGHS        0        0      -     1 lo0

$ route show
Routing tables

Internet:
Destination        Gateway            Flags     Refs     Use    Mtu  Prio Iface
default            10.0.0.1           UGS         4       16      -    12 fxp0
base-address.mcast localhost          URS         0        0  32768     8 lo0
loopback           localhost          UGRS        0        0  32768     8 lo0
localhost          localhost          UH          2       15  32768     1 lo0
10.0.0/24          link#1             UC          1        4      -     4 fxp0
10.0.0.1           aa:0:4:0:81:d      UHL         1       11      -     1 fxp0
10.0.0.38          localhost          UGHS        0        0      -     1 lo0
```

### 在接口上设置别名

要在接口上设置 IP 别名，只需编辑其 [hostname.if(5)](https://man.openbsd.org/hostname.if) 文件。

在本例中，两个别名被添加到接口 `dc0`，它被配置为 `192.168.0.2`，网络掩码为 `255.255.255.0`。

```
$ cat /etc/hostname.dc0
inet 192.168.0.2 255.255.255.0
inet alias 192.168.0.3 255.255.255.255
inet alias 192.168.0.4 255.255.255.255
```

创建此文件后，[运行 netstart](https://www.openbsd.org/faq/faq6.html#Setup.activate) 或重新启动。要查看所有别名，请使用 `ifconfig -A`。

## 动态主机设置协议

动态主机设置协议 (DHCP) 是一种自动配置网络接口的方法。OpenBSD 可以是配置其他机器的 DHCP 服务器，也可以是由 DHCP 服务器配置的 DHCP 客户端。

### DHCP 客户端

要使用 [dhcpleased(8)](https://man.openbsd.org/dhcpleased)，请编辑网络接口的 [hostname.if(5)](https://man.openbsd.org/hostname.if) 文件。[无线网络](https://www.openbsd.org/faq/faq6.html#Wireless)部分解释了如何设置无线接口。对于以太网接口，一行就足够了：

```
inet autoconf
```

OpenBSD 将在启动时从 DHCP 服务器收集其 IP 地址、默认网关和 DNS 服务器。其他选项可以在 [dhcpleased.conf(5)](https://man.openbsd.org/dhcpleased.conf) 中指定。

要从命令行通过 DHCP 获取 IP，请运行：

```
# ifconfig xl0 inet autoconf
```

将 `xl0` 替换为接口名称。

### DHCP 服务器

要使用 OpenBSD 作为 DHCP 服务器，请在启动时启用 [dhcpd(8)](https://man.openbsd.org/dhcpd) 守护进程：

```
# rcctl enable dhcpd
```

下次启动时，dhcpd 将运行并附加到 [dhcpd.conf(5)](https://man.openbsd.org/dhcpd.conf) 中具有有效配置的所有 NIC。可以通过明确地命名来指定单个接口：

```
# rcctl set dhcpd flags em1 em2
```

`/etc/dhcpd.conf` 示例文件可能如下所示：

```
# Home
subnet 192.168.1.0 netmask 255.255.255.0 {
	option domain-name-servers 192.168.1.2;
	option routers 192.168.1.1;
	range 192.168.1.3 192.168.1.50;
}

# Guests
subnet 172.16.0.0 netmask 255.255.255.0 {
	option domain-name-servers 1.2.3.4, 5.6.7.8;
	option routers 172.16.0.1;
	range 172.16.0.2 172.16.0.254;
}
```

本示例中有两个子网：家庭网络和访客网络。客户端将自动获得一个 IP 地址，并指向配置文件各自部分中指定的网关和 DNS 服务器。有关更多选项，请参阅 [dhcp-options(5)](https://man.openbsd.org/dhcp-options)。

### PXE 启动（i386，amd64）

预引导执行环境 (PXE) 是仅使用网络引导系统的标准方法。客户端的具有 PXE 功能的 NIC 在[启动过程](https://www.openbsd.org/faq/faq14.html#BootAmd64)开始时广播 DHCP 请求，并且不仅接收基本的 IP/DNS 信息，还提供一个用于启动的文件。在 OpenBSD 上，这个文件被称为 [pxeboot(8)](https://man.openbsd.org/pxeboot)，通常由 [tftpd(8)](https://man.openbsd.org/tftpd) 提供服务。

## 无线网络

OpenBSD 支持[多种无线芯片组](https://man.openbsd.org/?query=wireless&apropos=1)。更多支持的设备可以在 [usb(4)](https://man.openbsd.org/usb) 和 [pci(4)](https://man.openbsd.org/pci) 中找到。驱动程序手册页中描述了它们支持的确切范围。

以下网卡支持基于主机的接入点 (Host-based Access Point, HostAP) 模式，允许它们用作[无线接入点](https://www.openbsd.org/faq/pf/example1.html)：

- [acx(4)](https://man.openbsd.org/acx) - TI ACX100/ACX111
- [ath(4)](https://man.openbsd.org/ath) - Atheros 802.11a/b/g
- [athn(4)](https://man.openbsd.org/athn) - Atheros 802.11/a/g/n devices
- [bwfm(4)](https://man.openbsd.org/bwfm) - Broadcom 和 Cypress IEEE 802.11a/ac/b/g/n 无线网络设备
- [pgt(4)](https://man.openbsd.org/pgt) - Conexant/Intersil Prism GT Full-MAC 802.11a/b/g
- [ral(4)](https://man.openbsd.org/ral) 和 [ural(4)](https://man.openbsd.org/ural) - Ralink Technology RT25x0 802.11a/b/g
- [rtw(4)](https://man.openbsd.org/rtw) - Realtek 8180 802.11b
- [rum(4)](https://man.openbsd.org/rum) - Ralink Technology RT2501USB
- [wi(4)](https://man.openbsd.org/wi) - Prism2/2.5/3

ifconfig(8) 的 `media` 子命令可用于显示网络接口的媒体功能。对于无线设备，它显示支持的 802.11a/b/g/n 媒体模式和支持的操作模式（`hostap`、`ibss`、`monitor`）。 例如，要查看接口 `ath0` 的媒体功能，请运行：

```
$ ifconfig ath0 media
```

为了使用某些无线网卡，可能需要通过 [fw_update(1)](https://man.openbsd.org/fw_update) 获取的固件文件。一些制造商拒绝允许[自由](https://www.openbsd.org/faq/faq1.html#ReallyFree)分发他们的固件，因此它不能包含在 OpenBSD 中。

另一个需要考虑的选择：为基于 OpenBSD 的防火墙使用传统的 NIC 和外部桥接无线接入点。

### 配置无线适配器

基于受支持芯片的适配器可以像任何其他网络接口一样使用。要将 OpenBSD 系统连接到现有的无线网络，请使用 [ifconfig(8)](https://man.openbsd.org/ifconfig) 实用程序。

无线客户端的 [hostname.if(5)](https://man.openbsd.org/hostname.if) 文件示例可能是：

```
nwid puffyuberalles wpakey passwordhere
inet autoconf
```

或者，对于多个接入点：

```
join home-net wpakey passwordhere
join work-net wpakey passwordhere
join cafe-wifi
inet autoconf
```

请注意 `inet autoconf` 应该放置在其他配置行之后，因为在配置完成之前网络适配器将无法发送 DHCP 请求。

### 中继无线适配器

中继（Trunks）是由一个或多个网络接口组成的虚拟接口。在本节中，我们的示例将是带有有线 [bge0](https://man.openbsd.org/bge) 接口和无线 [iwn0](https://man.openbsd.org/iwn) 接口的笔记本电脑。我们将使用它们构建一个 [trunk(4)](https://man.openbsd.org/trunk) 接口。有线和无线接口必须连接到同一个二层网络。

为此，我们首先激活两个物理端口，然后将它们分配给 `trunk0`。

```
# echo up > /etc/hostname.bge0
```

然而，无线接口需要更多的配置。它需要连接到我们受 WPA 保护的无线网络：

```
$ cat /etc/hostname.iwn0
nwid puffynet wpakey mysecretkey
up
```

现在，我们的中继接口定义如下：

```
$ cat /etc/hostname.trunk0
trunkproto failover trunkport bge0
trunkport iwn0
inet autoconf
```

中继设置为故障转移模式，因此可以使用任一接口。如果两者都可用，它将首选 `bge0` 端口，因为这是添加到中继设备的第一个端口。

## 搭建网桥

[bridge(4)](https://man.openbsd.org/bridge) 是两个或多个独立网络之间的链接。与路由器不同，数据包透明地通过网桥：两个网段对任一侧的节点都显示为一个。网桥只会转发必须从一个网段传递到另一个网段的数据包，因此网桥中的接口可能有也可能没有自己的 IP 地址。如果是这样，该接口实际上具有两种操作模式：一种作为网桥的一部分，另一种作为独立的 NIC。如果两个接口都没有 IP 地址，则网桥将传递网络数据，但无法进行外部维护（这可以是一个功能）。

### 作为 DHCP 服务器的网桥

假设我们有一个系统，它有四个 [vr(4)](https://man.openbsd.org/vr) 接口，`vr0` 到 `vr3`。 我们希望将 `vr1`、`vr2` 和 `vr3` 桥接在一起，而将 `vr0` 留给上行链路。 我们还希望通过桥接接口上的 DHCP 提供 IP 地址。作为 DHCP 服务器和上行链路路由器，该盒子需要在桥接网络上有一个 IP 地址。

无法将 IP 地址直接分配给网桥接口。IP 地址应该添加到成员接口之一，但我们不能使用物理接口，因为链接可能会关闭，在这种情况下，地址将无法访问。 幸运的是，有可用于此目的的 [vether(4)](https://man.openbsd.org/vether)（虚拟以太网）驱动程序。我们将它添加到网桥，为其分配 IP 地址并使 dhcpd(8) 在那里监听。

- 本节不再对 [DHCP 服务器配置](https://www.openbsd.org/faq/faq6.html#DHCPserver)进行描述，但这里使用的寻址方案是相同的。
- 这也将是桥接网络的上行链路路由器，因此我们将使用 IP 地址 `192.168.1.1` 来匹配 DHCP 服务器配置。
- 我们不会在这里介绍上行链路、路由或防火墙配置。

首先，将 `vr1`、`vr2` 和 `vr3` 接口标记为 `up`：

```
# echo up > /etc/hostname.vr1
# echo up > /etc/hostname.vr2
# echo up > /etc/hostname.vr3
```

然后创建 `vether0` 配置：

```
# echo 'inet 192.168.1.1 255.255.255.0 192.168.1.255' > /etc/hostname.vether0
```

配置桥接接口以包含上述所有接口：

```
$ cat /etc/hostname.bridge0
add vether0
add vr1
add vr2
add vr3
up
```

最后我们让 DHCP 守护进程监听 vether0 接口：

```
# rcctl set dhcpd flags vether0
```

最后，重新启动系统。

### 在网桥上过滤

[PF](https://www.openbsd.org/faq/pf/index.html) 可用于限制通过网桥的流量。请记住，根据网桥的性质，相同的数据流经两个接口，因此只需要在一个接口上进行过滤。

### 桥接技巧

- 通过使用 [ifconfig(8)](https://man.openbsd.org/ifconfig) 或 [hostname.bridge0](https://man.openbsd.org/hostname.if) 中的 `blocknonip` 选项，可以防止非 IP 流量（例如 IPX 或 NETBEUI）绕过过滤器。 这在某些情况下可能很重要，但请注意网桥适用于所有类型的流量，而不仅仅是 IP。
- 桥接要求 NIC 处于混杂模式（promiscuous mode）。它们侦听**所有**网络流量，而不仅仅是针对接口的流量。这会给处理器和总线带来比预期更高的负载。

## 等价多路径路由

等价多路径路由（Equal-Cost Multipath Routing）是指在同一网络的路由表中有多条路由，如默认路由 0.0.0.0/0。 当内核进行路由查找以确定将发往该网络的数据包发送到何处时，它可以从任何等价路由中进行选择。在大多数情况下，多路径路由用于提供冗余上行链路连接，例如到 Internet 的冗余连接。

[route(8)](https://man.openbsd.org/route) 命令用于在路由表中添加/更改/删除路由。添加多路径路由时使用 `-mpath` 参数。

```
# route add -mpath default 10.130.128.1
# route add -mpath default 10.132.0.1
```

验证路由：

```
# netstat -rnf inet | grep default
default     10.130.128.1      UGS       2      134      -     fxp1
default     10.132.0.1        UGS       0      172      -     fxp2
```

在这个例子中，我们可以看到一个默认路由指向 `10.130.128.1`，可以通过 `fxp1` 接口访问，另一个指向 `10.132.0.1`，可以通过 `fxp2` 访问。

由于 [mygate(5)](https://man.openbsd.org/mygate) 文件尚不支持多路径默认路由，因此应将上述命令添加到 fxp1 和 fxp2 接口的 [hostname.if(5)](https://man.openbsd.org/hostname.if) 文件的底部。然后应该删除 `/etc/mygate` 文件。

```
$ tail -1 /etc/hostname.fxp1
!route add -mpath default 10.130.128.1
$ tail -1 /etc/hostname.fxp2
!route add -mpath default 10.132.0.1
```

最后，不要忘记通过启用适当的 [sysctl(8)](https://man.openbsd.org/sysctl.8) 变量来激活多路径路由的使用。

```
# sysctl net.inet.ip.multipath=1
# sysctl net.inet6.ip6.multipath=1
```

请务必编辑 [sysctl.conf(5)](https://man.openbsd.org/sysctl.conf) 以使更改永久生效。

现在尝试跟踪路由到不同的目的地。内核将负载平衡每条多路径路由上的流量。

```
# traceroute -n 154.11.0.4
traceroute to 154.11.0.4 (154.11.0.4), 64 hops max, 60 byte packets
 1  10.130.128.1  19.337 ms  18.194 ms  18.849 ms
 2  154.11.95.170  17.642 ms  18.176 ms  17.731 ms
 3  154.11.5.33  110.486 ms  19.478 ms  100.949 ms
 4  154.11.0.4  32.772 ms  33.534 ms  32.835 ms

# traceroute -n 154.11.0.5
traceroute to 154.11.0.5 (154.11.0.5), 64 hops max, 60 byte packets
 1  10.132.0.1  14.175 ms  14.503 ms  14.58 ms
 2  154.11.95.38  13.664 ms  13.962 ms  13.445 ms
 3  208.38.16.151  13.964 ms  13.347 ms  13.788 ms
 4  154.11.0.5  30.177 ms  30.95 ms  30.593 ms
```

有关如何选择路由的更多信息，请参阅 [RFC2992](https://www.ietf.org/rfc/rfc2992.txt)，“等价多路径算法的分析（Analysis of an Equal-Cost Multi-Path Algorithm）”。

值得注意的是，如果多路径路由使用的接口出现故障（即丢失载波（carrier）），内核仍将尝试使用指向该接口的路由转发数据包。这种流量当然会被黑洞化，最终无处可去。强烈建议使用 [ifstated(8)](https://man.openbsd.org/ifstated) 来检查不可用的接口并相应地调整路由表。

## 使用 NFS

网络文件系统 NFS 用于通过网络共享文件系统。

本节将介绍简单 NFS 设置的步骤。该示例详细介绍了 LAN 上的服务器，客户端访问 LAN 上的 NFS。该节不包括保护 NFS。

### 搭建一个 NFS 服务器

首先，在服务器上启用 [portmap(8)](https://man.openbsd.org/portmap)、[mountd(8)](https://man.openbsd.org/mountd) 和 [nfsd(8)](https://man.openbsd.org/nfsd) 服务：

```
# rcctl enable portmap mountd nfsd
```

然后配置将可用的文件系统列表。

在本例中，我们有一个 IP 地址为 `10.0.0.1` 的服务器。此服务器将仅向其自己子网内的客户端提供 NFS 服务。这是在下列 exports(5) 文件中配置的：

```
$ cat /etc/exports
/docs -alldirs -ro -network=10.0.0 -mask=255.255.255.0
```

本地文件系统 `/docs` 将通过 NFS 提供。`-alldirs` 选项指定客户端将能够在 `/docs` 以及 `/docs` 本身下的任何位置挂载。`-ro` 选项指定客户端将仅被授予只读访问权限。最后两个参数指定只有使用 `255.255.255.0` 网络掩码的 `10.0.0.0` 网络中的客户端才能被授权挂载此文件系统。

现在启动服务器相关的服务：

```
# rcctl start portmap mountd nfsd
```

如果在 NFS 已经运行时对 `/etc/exports` 进行了更改，则必须重启 `mountd` 重新载入配置。

```
# rcctl reload mountd
```

### 挂载 NFS 文件系统

NFS 文件系统应该通过 [mount(8)](https://man.openbsd.org/mount_nfs)，或者更具体地说，[mount_nfs(8)](https://man.openbsd.org/mount_nfs) 挂载。

要将主机 `10.0.0.1` 上的 `/docs` 文件系统挂载到本地文件系统 `/mnt`，请运行：

```
# mount -t nfs 10.0.0.1:/docs /mnt
```

要在引导时挂载该文件系统，请在 [fstab(5)](https://man.openbsd.org/fstab) 中添加一行，如下所示：

```
# echo '10.0.0.1:/docs /mnt nfs ro,nodev,nosuid 0 0' >> /etc/fstab
```

在这一行的末尾使用 `0 0` 很重要，这样计算机就不会在启动时尝试 [fsck(8)](https://man.openbsd.org/fsck) NFS 文件系统。

当以 root 用户访问 NFS 挂载时，服务器会自动将 root 的访问权限映射到用户名 `nobody` 和组 `nobody`。在考虑文件权限时了解这一点很重要。例如，获取具有以下权限的文件：

```
-rw-------    1 root     wheel           0 Dec 31 03:00 _daily.B20143
```

如果此文件位于 NFS 共享上并且 root 用户尝试从 NFS 客户端访问此文件，则访问将被拒绝。

root 映射到的用户和组可以通过 NFS 服务器上的 [exports(5)](https://man.openbsd.org/exports) 文件进行配置。

### 检查 NFS 上的统计信息

要确保 NFS 正常运行，需要检查的一件事是所有守护程序都已正确注册到 RPC。 为此，请使用 [rpcinfo(8)](https://man.openbsd.org/rpcinfo)。

```
$ rpcinfo -p 10.0.0.1
   program vers proto   port
    100000    2   tcp    111  portmapper
    100000    2   udp    111  portmapper
    100005    1   udp    633  mountd
    100005    3   udp    633  mountd
    100005    1   tcp    916  mountd
    100005    3   tcp    916  mountd
    100003    2   udp   2049  nfs
    100003    3   udp   2049  nfs
    100003    2   tcp   2049  nfs
    100003    3   tcp   2049  nfs
```

一些实用程序允许管理员查看 NFS 发生的情况，例如 [showmount(8)](https://man.openbsd.org/showmount) 和 [nfsstat(1)](https://man.openbsd.org/nfsstat)。