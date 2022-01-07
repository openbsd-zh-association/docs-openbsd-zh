#  OpenBSD FAQ - 虚拟专用网络（VPN） 

## 介绍

OpenBSD 带有 [iked(8)](https://man.openbsd.org/iked)，这是一个现代的、权限分离的 IKEv2 服务器。它可以同时充当响应者（responder），例如接收连接请求的服务器；或发起者（initiator），例如客户端发起与响应者的连接。[ikectl(8)](https://man.openbsd.org/ikectl) 实用程序用于控制服务器，它从 [iked.conf(5)](https://man.openbsd.org/iked.conf) 文件中获取其配置。

[ikectl(8)](https://man.openbsd.org/ikectl) 实用程序还允许您为 IKEv2 对等方维护一个简单的 X.509 数字证书认证机构 (CA)。

IKEv1 服务器 ([isakmpd(8)](https://man.openbsd.org/isakmpd)) 也可用，结合 [npppd(8)](https://man.openbsd.org/npppd)，它允许您构建 IKEv1/L2TP VPN，其中 IKEv2 无法部署。

也可通过 [wg(4)](https://man.openbsd.org/wg) 设备获得原生的 WireGuard 支持。正如手册所解释的，它可以像 OpenBSD 中的所有其他[网络接口](https://www.openbsd.org/faq/faq6.html)一样进行配置。

## 身份验证

[iked(8)](https://man.openbsd.org/iked) 支持以下认证方法：

- 预共享密钥（不推荐）
- RSA 和 ECDSA 公钥：连接到 iked、RouterOS 和其他一些实现时轻松设置
- EAP MSCHAPv2（服务端带有 X.509 证书）：iked 仅在“响应者”（服务器）端支持此功能
- X.509 证书：Windows、Android 和 Apple 客户端通常需要

默认情况下，RSA 公钥在启动时在 `/etc/iked/local.pub` 中生成，私钥存储在 `/etc/iked/private/local.key` 中。

## 配置 IKEv2 服务器

### 构建点对点 VPN

这可以通过交换默认提供的 RSA 公钥来实现：应该将第一个系统（“server1”）上的 `/etc/iked/local.pub` 复制到第二个系统上的 `/etc/iked/pubkeys/fqdn/server1.domain` 系统（“server2”）。 然后，应该将第二个系统上的 `/etc/iked/local.pub` 复制到第一个系统上的 `/etc/iked/pubkeys/fqdn/server2.domain`。 将 “serverX.domain” 替换为你自己的 FQDN。

从这时起，我们假设 server1 的公共 IP 为 `192.0.2.1`，内网 IP 为 `10.0.1.0/24`，而 server2 的公共 IP 为 `198.51.100.1`，内网 IP 为 `10.0.2.0/24`。

要使发起方能够到达响应方，应在响应方上打开 `isakmp` UDP 端口。如果对等方之一在 NAT 之后，则 `ipsec-nat-t` UDP 端口也应在响应方上打开。如果两个对等点都有公共 IP，则应该允许 ESP 协议。

```
pass in log on $ext_if proto udp from 198.51.100.1 to 192.0.2.1 port {isakmp, ipsec-nat-t} tag IKED
pass in log on $ext_if proto esp from 198.51.100.1 to 192.0.2.1 tag IKED
```

（充当响应者的）server1 的示例 `/etc/iked.conf` 配置可能如下所示：

```
ikev2 'server1_rsa' passive esp \
        from 10.0.1.0/24 to 10.0.2.0/24 \
        local 192.0.2.1 peer 198.51.100.1 \
        srcid server1.domain
```

以及作为发起者的 server2 的简单配置：

```
ikev2 'server2_rsa' active esp \
        from 10.0.2.0/24 to 10.0.1.0/24 \
        peer 192.0.2.1 \
        srcid server2.domain
```

使用 `iked -dv` 可以帮助你了解交换过程。在这个例子中，响应者处于 NAT 网络中：

```
server1# iked -dv
...
ikev2_recv: IKE_SA_INIT request from initiator 198.51.100.1:500 to 192.0.2.1:500 policy 'server1_rsa' id 0, 510 bytes
ikev2_msg_send: IKE_SA_INIT response from 192.0.2.1:500 to 198.51.100.1:500 msgid 0, 451 bytes
ikev2_recv: IKE_AUTH request from initiator 198.51.100.1:4500 to 192.0.2.1:4500 policy 'server1_rsa' id 1, 800 bytes
ikev2_msg_send: IKE_AUTH response from 192.0.2.1:4500 to 198.51.100.1:4500 msgid 1, 720 bytes, NAT-T
sa_state: VALID -> ESTABLISHED from 198.51.100.1:4500 to 192.0.2.1:4500 policy 'server1_rsa'
```

在发起者：

```
server2# iked -dv
...
ikev2_msg_send: IKE_SA_INIT request from 0.0.0.0:500 to 192.0.2.1:500 msgid 0, 510 bytes
ikev2_recv: IKE_SA_INIT response from responder 192.0.2.1:500 to 198.51.100.1:500 policy 'server2_rsa' id 0, 451 bytes
ikev2_msg_send: IKE_AUTH request from 198.51.100.1:4500 to 192.0.2.1:4500 msgid 1, 800 bytes, NAT-T
ikev2_recv: IKE_AUTH response from responder 192.0.2.1:4500 to 198.51.100.1:4500 policy 'server2_rsa' id 1, 720 bytes
sa_state: VALID -> ESTABLISHED from 192.0.2.1:4500 to 198.51.100.1:4500 policy 'server2_rsa'
```

可以使用 [ipsecctl(8)](https://man.openbsd.org/ipsecctl) 查看 IPsec 流：

```
server1# ipsecctl -sa
FLOWS:
flow esp in from 10.0.2.0/24 to 10.0.1.0/24 peer 198.51.100.1 srcid FQDN/server1.domain dstid FQDN/server2.domain type use
flow esp out from 10.0.1.0/24 to 10.0.2.0/24 peer 198.51.100.1 srcid FQDN/server1.domain dstid FQDN/server2.domain type require
flow esp out from ::/0 to ::/0 type deny

SAD:
esp tunnel from 192.0.2.1 to 198.51.100.1 spi 0xabb5968a auth hmac-sha2-256 enc aes-256
esp tunnel from 198.51.100.1 to 192.0.2.1 spi 0xb1fc90b8 auth hmac-sha2-256 enc aes-256

server2# ipsecctl -sa
FLOWS:
flow esp in from 10.0.1.0/24 to 10.0.2.0/24 peer 192.0.2.1 srcid FQDN/server2.domain dstid FQDN/server1.domain type use
flow esp out from 10.0.2.0/24 to 10.0.1.0/24 peer 192.0.2.1 srcid FQDN/server2.domain dstid FQDN/server1.domain type require
flow esp out from ::/0 to ::/0 type deny

SAD:
esp tunnel from 192.0.2.1 to 198.51.100.1 spi 0xabb5968a auth hmac-sha2-256 enc aes-256
esp tunnel from 198.51.100.1 to 192.0.2.1 spi 0xb1fc90b8 auth hmac-sha2-256 enc aes-256
```

这样，两个内部网络应该能够相互访问。 它们之间的流量应该在 `enc0` 接口上解封装后出现，并且可以这样过滤。在该示例中，`tag VPN` 已添加到策略中：

```
# pfctl -vvsr|grep VPN
@16 pass log on enc0 tagged VPN
# tcpdump -nei pflog0 rnr 16
00:03:26.793522 rule 16/(match) pass in on enc0: 10.0.2.24 > 10.0.1.13: icmp: echo request
```

一些警告的话：

- 如果响应者未设置 `srcid`，则 iked 将默认尝试使用与其 FQDN 匹配的密钥。
- 响应者**不需要**设置 `local`，它只是确保 `iked` 在正确的接口上监听。
- 响应者**不需要**设置 `peer`，它只是确保连接来自受信任的 IP。

如果 VPN 端点需要到达远程内部网络，或者内部网络需要到达远程 VPN 端点，则必须在两端设置额外的流：

- 从本地公网 IP 到远程网络
- 从内网到远程公网 IP

响应者配置将如下所示：

```
ikev2 'server1_rsa' passive esp \
        from 10.0.1.0/24 to 10.0.2.0/24 \
        from 10.0.1.0/24 to 198.51.100.1 \
        from 192.0.2.1 to 10.0.2.0/24 \
        local 192.0.2.1 peer 198.51.100.1 \
        srcid server1.domain
```

发起者的配置将是：

```
ikev2 'server2_rsa' active esp \
        from 10.0.2.0/24 to 10.0.1.0/24 \
        from 10.0.2.0/24 to 192.0.2.1 \
        from 198.51.100.1 to 10.0.1.0/24 \
        peer 192.0.2.1 \
        srcid server2.domain
```

## 连接到 IKEv2 OpenBSD VPN

作为 *road warrior* 连接到 IKEv2 VPN 与前一种情况类似，不同之处在于发起者通常计划通过响应者路由其互联网流量，响应者将对其应用 NAT，因此发起者流量似乎来自响应者的 公网IP。

根据用例，由于所有流量都将通过响应方，因此必须确保将发起方配置为使用它可以访问的 DNS 服务器（可能是响应方上的一个）。

### 使用 OpenBSD 客户端

在我们的示例中，`10.0.5.0/24` 网络用于支持 VPN。实际的内部 IP 地址将由 iked 自动安装在 lo1 接口上。我们假设客户端的公共 IP 是 `203.0.113.2`。

与前面的示例一样，交换默认提供的 RSA 公钥足以在响应者和发起者之间设置简单的身份验证：第一个系统（“server1”）上的 `/etc/iked/local.pub` 应复制到 第二个系统（“roadwarrior”）上的 `/etc/iked/pubkeys/fqdn/server1.domain`。然后，应该首先将第二个系统上的 `/etc/iked/local.pub` 复制到 `/etc/iked/pubkeys/fqdn/roadwarrior`。 将 “serverX.domain” 替换为你自己的 FQDN。

响应者的 [iked.conf(5)](https://man.openbsd.org/iked.conf) 创建从任何目的地到地址池中的动态 IP 租用的流，这将在运行时决定，并用 ROADW 标记数据包：

```
ikev2 'responder_rsa' passive esp \
        from any to dynamic \
        local 192.0.2.1 peer any \
        srcid server1.domain \
        config address 10.0.5.0/24 \
        tag "ROADW"
```

响应者需要向发起者提供一个 IP 地址。这是通过 `config ` 指令实现的。使用 `config address` 选项时，`to dynamic` 将替换为分配的动态 IP 地址。

它还需要允许来自任何主机的 IPsec（因为客户端可能从任何地方连接），允许在 `enc0` 上标记为 ROADW 的流量并对其应用 NAT：

```
pass in log on $ext_if proto udp from any to 192.0.2.1 port {isakmp, ipsec-nat-t} tag IKED
pass in log on $ext_if proto esp from any to 192.0.2.1 tag IKED
pass log on enc0 tagged ROADW
match out log on $ext_if inet tagged ROADW nat-to $ext_if
```

发起者配置一个全局流以将其所有流量发送给响应者，告诉它使用名为 “roadwarrior” 的密钥来标识自己：

```
ikev2 'roadwarrior' active esp \
        from dynamic to any \
        peer 192.0.2.1 \
        srcid roadwarrior \
        dstid server1.domain \
        request address any \
        iface lo1
```

发起者使用 `request address any` 选项向响应方请求动态 IP 地址。`iface lo1` 选项指定将安装接收地址和相应路由的接口。响应者应该为 *road warrior* 客户端正确配置 NAT。

你可以使用 `ikectl decouple`（`iked` 仍在运行，挂起 `ikectl Couple` 以便它重新连接到响应者）或使用 `ikectl reset sa && rcctl stop iked` 永久停止 `iked` 并确保没有被遗弃的数据流，从而优雅地停止启动器上的 VPN .

### 使用 Android 客户端

默认的 Android VPN 客户端仅支持 IKEv1。 要使用 IKEv2，可以选择 [strongSwan](https://www.strongswan.org/)。

你还需要设置 PKI 和 X.509 证书，以便发起方可以验证响应方通告的证书：

```
server1# ikectl ca vpn create
server1# ikectl ca vpn install
certificate for CA 'vpn' installed into /etc/iked/ca/ca.crt
CRL for CA 'vpn' installed to /etc/iked/crls/ca.crl
server1# ikectl ca vpn certificate server1.domain create
server1# ikectl ca vpn certificate server1.domain install
writing RSA key
server1# cp /etc/iked/ca/ca.crt /var/www/htdocs/
```

在 android 设备上，进入 `http://192.0.2.1/ca.crt` 并在 strongSwan 客户端中导入 CA 证书。现在有几种选择可以向响应者验证发起者：

- 带有用户名/密码的 EAP
- X.509 证书

#### 使用 MSCHAP-V2 进行 EAP 身份验证

响应者配置需要一份指定用户名/密码列表，并且它将使用 `eap "mschap-v2"`（这是目前唯一支持的 EAP 方法），如下所示：

```
user 'android' 'password'
ikev2 'responder_eap' passive esp \
        from any to dynamic \
        local 192.0.2.1 peer any \
        srcid server1.domain \
        eap "mschap-v2" \
        config address 10.0.5.0/24 \
        config name-server 192.0.2.1 \
        tag "ROADW"
```

在 strongSwan 客户端中，使用以下命令配置新配置文件：

- VPN 类型的 *IKEv2 EAP*
- `192.0.2.1` 为*服务器*领域
- 在响应者配置中设置的登录名/密码值（login/password）
- *CA 证书*（CA certificate）字段新导入的 `CN=VPN CA` 证书
- *用户身份*（User identity）字段的 `client1.domain`
- *服务器标识*（Server identity）字段中的 server1.domain（在“高级设置”下）

这样，Android 设备就可以连接到响应者，使用 CA 证书验证响应者证书，使用 EAP 登录名/密码向响应者验证自身，获取 `10.0.5.0/24` 网络中的地址，而它的所有流量都通过 VPN，使用 `192.0.2.1` 作为其 DNS 服务器。

#### 使用 X.509 证书认证

对于这种方法，会为客户端生成一个证书，安装在 `iked ca` 中，导出为归档文件，并且 `.pfx` 文件应该在线可用，以便客户端可以安装它。`.pfx` 文件包含了：

- X.509 证书
- 使用 RSA 加密的 X.509 私钥，
- 用于加密 X.509 私钥的 RSA 私钥
- RSA 公钥

```
server1# ikectl ca vpn certificate client1.domain create
server1# cp /etc/ssl/vpn/client1.domain.crt /etc/iked/certs/
server1# ikectl ca vpn certificate client1.domain export
server1# tar -C /tmp -xzf client1.domain.tgz *pfx
server1# cp /tmp/export/client1.domain.pfx /var/www/htdocs/client1.domain.pfx
```

配置新配置文件时，必须在 strongSwan 客户端中导入 CA 公共证书和客户端证书包。

响应者配置稍微简单一些，因为不需要指定 eap 也不需要设置用户名/密码：

```
ikev2 'responder_x509' passive esp \
        from any to dynamic \
        local 192.0.2.1 peer any \
        srcid server1.domain \
        config address 10.0.5.0/24 \
        config name-server 192.0.2.1 \
        tag "ROADW"
```

在 strongSwan 客户端中，配置一个新的配置文件需要使用：

- VPN 类型的 *IKEv2 证书*
- `192.0.2.1` 为服务器领域
- *用户证书*字段新导入的 `CN=client1.domain` 证书
- *用户身份*字段的 `client1.domain`
- *服务器标识*字段中的 `server1.domain`（在“高级设置”下）

与 EAP 案例一样，Android 设备现在可以连接到响应者并使用 VPN。

### 使用 Windows 客户端

Windows 7 及更高版本提供 IKEv2 启动器，该启动器也需要使用 X.509 证书，需要将其导出为 .pfx/.p12 包并导入到本地计算机（而不是用户帐户）证书存储中，两者均用于 CA 和客户端，使用图形化 Microsoft 管理控制台（在命令行中键入 `mmc` 并将证书管理单元添加为计算机帐户）或使用 Windows 10 的 `certutil` 命令。在 `certificate authority` 存储中导入 `ca.crt` 和在 `personal` 存储中导入 `ClientIP .p12`。[StrongSwan](https://wiki.strongswan.org/projects/strongswan/wiki/Win7Certs) 项目有一个带有截图的，关于这个主题的很好的文档。

Windows 不允许为客户端设置 `srcid` 参数，因此客户端证书的 CN 字段必须与发送到响应者的客户端 FQDN 或默认情况下的 IP 匹配。 还要求响应者上的 `srcid` 与响应者 FQDN（或它的 IP，如果不使用 FQDN）相匹配 —— 否则可能会遇到可怕的 `error 3801`。Libreswan 项目有有关于这些[要求](https://libreswan.org/wiki/Interoperability#Windows_Certificate_requirements)的[宝贵细节](https://libreswan.org/wiki/VPN_server_for_remote_clients_using_IKEv2#Common_Windows_7_client_errors)。

导入证书后，使用以下命令配置新的 VPN 连接：

- 常规选项卡中填入目标主机名的响应者 FQDN
- 安全选项卡中的类型设置为 *IKEv2*
- 身份验证选择 “机器证书（machine certificate）” 或 “EAP 身份验证（EAP authentication）”
- 如果使用 EAP，连接时会提示登录名/密码

响应者的配置文件将类似于前文的 Android 案例。

```
user 'windows' 'password'
ikev2 'responder_eap' passive esp \
        from any to dynamic \
        local 192.0.2.1 peer any \
        srcid server1.domain.fqdn \
        eap "mschap-v2" \
        config address 10.0.5.0/24 \
        config name-server 192.0.2.1 \
        tag "ROADW"
```

默认情况下，所有 Windows 流量现在都将通过 IKEv2 VPN。

在撰写本文时，当前版本的 Windows 默认使用弱加密 (3DES/SHA1)。 可以使用 PowerShell 命令 `Set-VpnConnectionIPsecConfiguration` 更正此问题。

## 连接到 IKEv1/L2TP OpenBSD VPN

有时，人们无法控制 VPN 服务器，只能选择连接到 IKEv1 服务器。 在这种情况下，需要 `xl2tpd` 第三方包作为 L2TP 客户端。

首先需要启用 [isakmpd(8)](https://man.openbsd.org/isakmpd) 和 `ipsec` 服务，以便启动守护进程并在启动时加载 [ipsec.conf(5)](https://man.openbsd.org/ipsec.conf) 配置文件：

```
# rcctl enable ipsec
# rcctl enable isakmpd
# rcctl set isakmpd flags -K
```

以下 [ipsec.conf(5)](https://man.openbsd.org/ipsec.conf) 配置应该允许使用提供的 PSK 连接到 `A.B.C.D` 的 IKEv1 服务器，只允许 L2TP 的 UDP 端口 1701：

```
ike dynamic esp transport proto udp from egress to A.B.C.D port l2tp \
        psk mekmitasdigoat
```

启动 [isakmpd(8)](https://man.openbsd.org/isakmpd) 并使用 [ipsecctl(8)](https://man.openbsd.org/ipsec.conf) 加载 [ipsec.conf(5)](https://man.openbsd.org/ipsec.conf) 应该允许您可视化配置的安全关联 (Security Associations, SA) 和流：

```
# rcctl start isakmpd
# ipsecctl -f /etc/ipsec.conf
# ipsecctl -sa
FLOWS:
flow esp in proto udp from A.B.C.D port l2tp to W.X.Y.Z peer A.B.C.D srcid my.client.fqdn dstid A.B.C.D/32 type use
flow esp out proto udp from W.X.Y.Z to A.B.C.D port l2tp peer A.B.C.D srcid my.client.fqdn dstid A.B.C.D/32 type require

SAD:
esp transport from A.B.C.D to W.X.Y.Z spi 0x0d16ad1c auth hmac-sha1 enc aes
esp transport from W.X.Y.Z to A.B.C.D spi 0xcd0549ba auth hmac-sha1 enc aes
```

如果不是这种情况，当双方交换加密参数以商定可用的最佳组合时，可能需要调整阶段 1（主要）和阶段 2（快速）参数。 理想情况下，这些参数应该由远程服务器管理员提供，并且应该在 [ipsec.conf(5)](https://man.openbsd.org/ipsec.conf) 中使用：

```
ike dynamic esp transport proto udp from egress to A.B.C.D port l2tp \
        main auth "hmac-sha1" enc "3des" group modp1024 \
        quick auth "hmac-sha1" enc "aes" \
        psk mekmitasdigoat
```

一旦 IKEv1 隧道启动并运行，就需要配置 L2TP 隧道。OpenBSD 默认不提供 L2TP 客户端，因此需要安装 `xl2tpd`。

```
# pkg_add xl2tpd
```

有关如何正确设置 L2TP 客户端的说明，请参阅 `/usr/local/share/doc/pkg-readmes/xl2tpd`。