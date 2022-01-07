#  OpenBSD FAQ - 虚拟化

## 介绍

OpenBSD 带有 [vmm(4)](https://man.openbsd.org/vmm) 管理程序和 [vmd(8)](https://man.openbsd.org/vmd) 守护进程。你可以使用 [vmctl(8)](https://man.openbsd.org/vmctl) 控制程序控制虚拟机，也可使用存储在 [vm.conf(5)](https://man.openbsd.org/vm.conf) 文件中的配置设置来配置虚拟机。

支持使用以下功能：

- 对虚拟机的串行控制台访问
- [tap(4)](https://man.openbsd.org/tap) 接口
- 每个 VM 用户/组所有权
- 权限分离
- raw、qcow2 和 qcow2 派生镜像文件
- 访客系统（guest system）内存的转储和恢复
- 虚拟交换机管理
- 暂停和取消暂停虚拟机

以下功能暂不支持：

- 图形化界面
- 快照
- 访客 SMP 支持
- 硬件直通
- 跨主机实时迁移
- 实时硬件更改

支持的访客操作系统（guest operating system）目前仅限于 OpenBSD 和 Linux。 由于尚不支持 VGA，因此来宾操作系统必须支持串行控制台。

## 前提条件

需要具有嵌套分页支持的 CPU 才能使用 [vmm(4)](https://man.openbsd.org/vmm)。可以通过查看处理器特性标识来检查支持情况：SLAT 代表 AMD 或 EPT 代表 Intel。 在某些情况下，必须在系统的 BIOS 中手动启用虚拟化功能。执行此操作后，请务必运行 [fw_update(8)](https://man.openbsd.org/fw_update) 命令以获取所需的 `vmm-firmware` 包。

可以使用以下命令检查处理器兼容性：

```
$ dmesg | egrep '(VMX/EPT|SVM/RVI)'
```

然后启用并启动 [vmd(8)](https://man.openbsd.org/vmd) 服务。

```
# rcctl enable vmd
# rcctl start vmd
```

## 启动一个虚拟机

在以下示例中，将创建一个具有 50GB 磁盘空间和 1GB RAM 的虚拟机（virtual machine，VM）。它将从 install70.iso 镜像文件启动。

```
# vmctl create -s 50G disk.qcow2
vmctl: qcow2 imagefile created
# vmctl start -m 1G -L -i 1 -r install70.iso -d disk.qcow2 example
vmctl: started vm 1 successfully, tty /dev/ttyp8
# vmctl show
   ID   PID VCPUS  MAXMEM  CURMEM     TTY        OWNER NAME
    1 72118     1    1.0G   88.1M   ttyp8         root example
```

要查看新创建的 VM 的控制台，请连接到其串行控制台：

```
# vmctl console example
Connected to /dev/ttyp8 (speed 115200)
```

要离开串行控制台，需要使用转义序列 `~.`。更多信息请参见 [cu(1)](https://man.openbsd.org/cu) 手册页。当通过 SSH 使用 `vmctl` 串行控制台时，`~`（波浪符）字符必须被转义以防止 [ssh(1)](https://man.openbsd.org/ssh) 丢弃连接。要通过 SSH 退出串行控制台，请使用 `~~.` 来代替。

可以使用 [vmctl(8)](https://man.openbsd.org/vmctl) 停止 VM。

```
# vmctl stop example
stopping vm: requested to shutdown vm 1
```

虚拟机可以在有或没有 [vm.conf(5)](https://man.openbsd.org/vm.conf) 文件的情况下启动。以下 `/etc/vm.conf` 示例将复制上述配置：

```
vm "example" {
    memory 1G
    enable
    disk /home/user/disk.qcow2
    local interface
}
```

[vm.conf(5)](https://man.openbsd.org/vm.conf) 中的一些配置属性可以由 [vmd(8)](https://man.openbsd.org/vmd) 在系统运行时重新加载。其他变化，如调整内存或磁盘空间的数量，需要重新启动虚拟机。

## 网络

可以通过多种不同的方式配置 vmm(4) 虚拟机的网络访问设置，本节详细介绍了其中的四种。

在下面的示例中，将针对不同的用例提及各种 IPv4 地址范围：

- 私有位址 ([RFC1918](https://tools.ietf.org/html/rfc1918)) 是为私有网络保留的地址，例如 `10.0.0.0/8`、`172.16.0.0/12` 和 `192.168.0.0/16`，它们不是全局可路由的。
- 共享地址（Shared Addresses） ([RFC6598](https://tools.ietf.org/html/rfc6598)) 类似于私有地址，因为它们不可全局路由，但旨在用于可以执行地址转换的设备上。地址空间为 `100.64.0.0/10`。

### 方法一：虚拟机只需要与主机和彼此交流

对于此设置，vmm 使用*本地接口（local interfaces）*：使用上面定义的共享地址空间的接口。

使用 [vmctl(8)](https://man.openbsd.org/vmctl) 的 `-L` 标志在虚拟机中创建一个本地接口，它将通过 DHCP 从 vmd 接收地址。这实质上创建了两个接口：一个用于主机，另一个用于 VM。

### 方法二：用于虚拟机的 NAT 网络

此设置建立在以前的基础上，并允许 VM 连接到主机外部的网络。它需要 [IP 转发](https://man.openbsd.org/sysctl.2#ip.forwarding)才能工作。

`/etc/pf.conf` 中的以下行将启用[网络地址转换](https://www.openbsd.org/faq/pf/nat.html)（NAT）并将 DNS 请求重定向到指定的服务器：

```
match out on egress from 100.64.0.0/10 to any nat-to (egress)
pass in proto { udp tcp } from 100.64.0.0/10 to any port domain \
	rdr-to $dns_server port domain
```

重新加载 pf 规则集，VM 现在可以连接到 Internet。

### 方法三：对 VM 网络配置的额外控制

有时，您可能希望对虚拟机的虚拟网络进行额外控制，例如能够将某些虚拟机放在它们自己的虚拟交换机上。这可以使用 [bridge(4)](https://man.openbsd.org/bridge) 和 [vether(4)](https://man.openbsd.org/vether) 接口来完成。

创建一个 `vether0` 接口，该接口将具有如上定义的私有 IPv4 地址。 在本例中，我们将使用 10.0.0.0/8 子网。

```
# echo 'inet 10.0.0.1 255.255.255.0' > /etc/hostname.vether0
# sh /etc/netstart vether0
```

使用 `vether0` 接口作为桥端口创建 `bridge0` 接口：

```
# echo 'add vether0' > /etc/hostname.bridge0
# sh /etc/netstart bridge0
```

如果虚拟网络上的 VM 需要访问物理机之外的网络，请确保正确设置 NAT。`/etc/pf.conf` 中调整后的 NAT 行可能如下所示：

```
match out on egress from vether0:network to any nat-to (egress)
```

[vm.conf(5)](https://man.openbsd.org/vm.conf) 中的以下几行可用于确保定义了虚拟交换机：

```
switch "my_switch" {
    interface bridge0
}

vm "my_vm" {
    ...
    interface { switch "my_switch" }
}
```

在 `my_vm` 虚拟机中，现在可以为 `vio0` 分配 `10.0.0.0/24` 网络上的地址并将默认路由设置为 `10.0.0.1`。

为方便起见，你可能需要在 `vether0` 上设置一个 [DHCP 服务器](https://www.openbsd.org/faq/faq6.html#DHCP)。

### 方法四：虚拟机作为同一网络上的真实主机

在这种情况下，VM 接口将连接到与主机相同的网络，因此可以对其进行配置，就好像它物理连接到主机网络一样。此选项仅适用于基于以太网的设备，因为 IEEE 802.11 标准禁止无线接口参与网络桥接。

使用主机网络接口作为桥端口创建 `bridge0` 接口。在此示例中，主机网络接口是 `em0` —— 在此，你应该将其替换为你希望将 VM 连接到的接口名称：

```
# echo 'add em0' > /etc/hostname.bridge0
# sh /etc/netstart bridge0
```

如上例所示，新建或修改 [vm.conf(5)](https://man.openbsd.org/vm.conf) 文件以确保定义了虚拟交换机：

```
switch "my_switch" {
    interface bridge0
}

vm "my_vm" {
    ...
    interface { switch "my_switch" }
}
```

`my_vm` 虚拟机现在可以参与到主机网络中，就像它被物理连接一样。

**注意**：如果主机接口（上例中的 em0）也使用 DHCP 配置，则在该接口上运行的 [dhcpleased(8)](https://man.openbsd.org/dhcpleased) 可能会阻止 DHCP 请求到达 VM。在这种情况下，你应该选择一个不使用 DHCP 的不同主机接口，或者在启动 VM 之前终止分配给该接口的任何 [dhcpleased(8)](https://man.openbsd.org/dhcpleased) 进程，或者为 VM 使用静态 IP 地址。