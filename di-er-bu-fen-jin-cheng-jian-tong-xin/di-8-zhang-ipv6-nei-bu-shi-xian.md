# 第 8 章 IPv6 内部实现

## 8.1 IPv6/IPsec 实施

本节应解释与 IPv6 和 IPsec 相关的实现内部。这些功能源自 KAME 项目

### 8.1.1. IPv6

#### 8.1.1.1. 一致性

IPv6 相关功能符合或试图符合最新的 IPv6 规范集。为了以后参考，我们列出以下一些相关文件（注意：这不是一个完整的列表——这太难维护了……）。

详情请参阅文档中的具体章节、RFCs、手册页面或源代码中的注释。

在 KAME 稳定套件上进行了符合性测试，结果可以在 TAHI 项目的网站上查看。我们过去也参加过新罕布什尔大学 IOL 测试（http://www.iol.unh.edu/），使用我们以前的快照。

* RFC1639：FTP 大地址记录上的操作（FOOBAR）

  * RFC2428 优先于 RFC1639。FTP 客户端将首先尝试 RFC2428，如果失败则尝试 RFC1639。
* RFC1886：支持 IPv6 的 DNS 扩展
* RFC1933：IPv6 主机和路由器的过渡机制

  * 不支持 IPv4 兼容地址。
  * 不支持自动隧道（RFC 中第 4.3 节描述）。
  * gif(4)接口以通用方式实现 IPv[46]隧道，并涵盖规范中描述的"配置隧道"。有关详细信息，请参见本文档中的 23.5.1.5。
* RFC1981：IPv6 的路径 MTU 发现
* RFC2080：IPv6 的 RIPng

  * usr.sbin/route6d 支持此功能。
* RFC2292：IPv6 的高级套接字 API

  * 有关支持的库函数/内核 API，请参阅 sys/netinet6/ADVAPI。
* RFC2362：协议无关组播-稀疏模式（PIM-SM）

  * RFC2362 定义了 PIM-SM 的数据包格式。draft-ietf-pim-ipv6-01.txt 是基于这个协议编写的。
* RFC2373：IPv6 地址架构

  * 支持节点必需地址，并符合范围要求。
* RFC2374：IPv6 可聚合全局单播地址格式

  * 支持 64 位接口 ID 长度。
* RFC2375：IPv6 组播地址分配

  * 用户态应用程序使用 RFC 中分配的知名地址。
* RFC2428：IPv6 和 NAT 的 FTP 扩展

  * RFC2428 优先于 RFC1639。FTP 客户端首先尝试 RFC2428，如果失败则尝试 RFC1639。
* RFC2460：IPv6 规范
* RFC2461：IPv6 邻居发现

  * 查看本文档中的 23.5.1.2 节以获取详细信息。
* RFC2462：IPv6 无状态地址自动配置

  * 查看本文档中的 23.5.1.4 节以获取详细信息。
* RFC2463：IPv6 规范的 ICMPv6

  * 有关详细信息，请参阅本文档中的 23.5.1.9。
* RFC2464：在以太网网络上传输 IPv6 数据包
* RFC2465：IPv6 的 MIB：文本约定和通用组

  * 必要的统计数据由内核收集。实际的 IPv6 MIB 支持作为 ucd-snmp 的补丁包提供。
* RFC2466：IPv6 的 MIB：ICMPv6 组

  * 内核收集必要的统计数据。 实际的 IPv6 MIB 支持作为 ucd-snmp 的补丁包提供。
* RFC2467：在 FDDI 网络上传输 IPv6 数据包
* RFC2497：在 ARCnet 网络上传输 IPv6 数据包
* RFC2553：IPv6 的基本套接字接口扩展

  * IPv4 映射地址（3.7）和 IPv6 通配符绑定套接字（3.8）的特殊行为都受支持。详情请参阅本文档中的 23.5.1.12。
* RFC2675：IPv6 巨字节

  * 查看本文档中的 23.5.1.7 以获取详细信息。
* RFC2710：IPv6 的组播监听者发现
* RFC2711：IPv6 路由器警报选项
* IPv6 路由器重新编号
* 通过 ICMP 进行 IPv6 名称查找
* 通过 ICMP 进行 IPv6 名称查找
* draft-ietf-pim-ipv6-01.txt：IPv6 的 PIM

  * pim6dd(8)实现密集模式。pim6sd(8)实现稀疏模式。
* draft-itojun-ipv6-tcp-to-anycast-00：断开面向 IPv6 任播地址的 TCP 连接
* 草案 yamamoto-wideipv6-comm-model-00

  * 有关详细信息，请参阅本文档中的 23.5.1.6。
* draft-ietf-ipngwg-scopedaddr-format-00.txt：IPv6 范围地址格式的扩展

#### 8.1.1.2. 邻居发现

邻居发现相当稳定。当前支持地址解析、重复地址检测和邻居不可达检测。在不久的将来，我们将在内核中添加代理邻居通告支持，并作为管理员工具添加不经请求的邻居通告传输命令。

如果重复地址检测失败，地址将被标记为“重复”，并生成消息到系统日志（通常也会到控制台）。管理员有责任检查并从重复地址检测失败中恢复。未来应该改进这种行为。

有些网络驱动程序会将多播数据包环回到自身，即使被指示不这样做（特别是在混杂模式下）。在这种情况下，DAD 可能会失败，因为 DAD 引擎看到入站的 NS 数据包（实际上来自节点本身），并将其视为重复的标志。您可能希望查看 sys/netinet6/nd6_nbr.c:nd6_dad_timer() 中标记为“启发式”条件的 #if 语句作为解决方法（请注意，“启发式”部分中的代码片段不符合规范）。

邻居发现规范（RFC2461）未涉及以下情况中的邻居缓存处理：

1. 当没有邻居缓存条目时，节点收到了未经请求的 RS/NS/NA/重定向数据包而没有链路层地址
2. 在没有链路层地址的中介上处理邻居缓存（我们需要一个带有 IsRouter 位的邻居缓存条目）

对于第一种情况，我们根据 IETF ipngwg 邮件列表上的讨论实施了基于临时解决方案。有关详细信息，请参阅源代码中的注释和从（IPng 7155）开始的电子邮件线程，日期为 1999 年 2 月 6 日。

IPv6 的本地链路确定规则（RFC2461）与 BSD 网络代码中的假设有很大不同。目前，不支持当默认路由器列表为空时的本地链路确定规则（RFC2461，第 5.2 节，第二段的最后一句 - 请注意，规范在该节中在几个地方误用了“主机”和“节点”这两个词）。

为避免可能的 DoS 攻击和无限循环，现在仅接受 ND 数据包上的 10 个选项。因此，如果您有 20 个前缀选项附加到 RA 上，只有前 10 个前缀将被识别。如果这给您带来麻烦，请在 FREEBSD-CURRENT 邮件列表上提出，并/或修改 sys/netinet6/nd6.c 中的 nd6_maxndopt。如果有很高的需求，我们可能会为该变量提供 sysctl 旋钮。

#### 8.1.1.3. 范围索引

IPv6 使用范围地址。因此，对于 IPv6 地址，指定范围索引（链路本地地址的接口索引，或站点本地地址的站点索引）非常重要。没有范围索引，范围 IPv6 地址对内核来说是模棱两可的，内核将无法确定数据包的出站接口。

普通用户空间应用程序应使用高级 API（RFC2292）来指定范围索引或接口索引。为了类似的目的，在 sockaddr_in6 结构中定义了 sin6_scope_id 成员，该结构在 RFC2553 中定义。然而，对于 sin6_scope_id 的语义相当模糊。如果您关心应用程序的可移植性，我们建议您使用高级 API 而不是 sin6_scope_id。

在内核中，用于链路本地范围地址的接口索引嵌入到 IPv6 地址的第 2 个 16 位字（第 3 个和第 4 个字节）中。例如，您可能会看到类似以下内容：

```
	fe80:1::200:f8ff:fe01:6317
```

在路由表和接口地址结构（struct in6_ifaddr）中。上面的地址是一个属于接口标识符为 1 的网络接口的链路本地单播地址。嵌入的索引使我们能够有效地识别多个接口上的 IPv6 链路本地地址，并且只需进行少量代码更改。

路由守护程序和配置程序，如 route6d(8) 和 ifconfig(8)，需要操作 "嵌入式" 范围索引。这些程序使用路由套接字和 ioctls（如 SIOCGIFADDR_IN6），内核 API 将返回填入第二个 16 位字的 IPv6 地址。这些 API 用于操作内核的内部结构。使用这些 API 的程序必须准备处理内核版本之间的差异。

当您在命令行中指定范围地址时，永远不要写入嵌入式形式（例如 ff02:1::1 或 fe80:2::fedc）。这不应该起作用。始终使用标准形式，例如 ff02::1 或 fe80::fedc，并使用命令行选项来指定接口（例如 ping -6 -I ne0 ff02::1 ）。通常，如果一个命令没有命令行选项来指定出口接口，那么该命令就没有准备好接受范围地址。这可能与 IPv6 支持 "牙医诊所" 情况的前提相反。我们认为，规范需要一些改进。

一些用户空间工具支持扩展的数值 IPv6 语法，如 draft-ietf-ipngwg-scopedaddr-format-00.txt 中所述。您可以通过使用出口接口的名称，例如 "fe80::1%ne0"，来指定出口链路。这样，您就能够轻松地指定链路本地范围的地址。

要在程序中使用这个扩展，你需要使用 getaddrinfo(3) 和带有 NI_WITHSCOPEID 的 getnameinfo(3)。目前的实现假设链路和接口之间存在 1 对 1 的关系，这比规范所要求的更严格。

#### 8.1.1.4. 即插即用

大多数 IPv6 无状态地址自动配置是在内核中实现的。邻居发现功能作为一个整体在内核中实现。主机的路由器通告 (RA) 输入在内核中实现。终端主机的路由器请求 (RS) 输出、路由器的 RS 输入以及路由器的 RA 输出是在用户空间中实现的。

##### 8.1.1.4.1. 链路局域网和特殊地址的分配

IPv6 链路局域网地址是根据 IEEE802 地址（以太网 MAC 地址）生成的。每个接口在变为上行接口（IFF_UP）时会自动分配一个 IPv6 链路局域网地址。此外，链路局域网地址的直接路由也会添加到路由表中。

这是 netstat 命令的输出：

```
Internet6:
Destination                   Gateway                   Flags      Netif Expire
fe80:1::%ed0/64               link#1                    UC          ed0
fe80:2::%ep0/64               link#2                    UC          ep0
```

无 IEEE802 地址的接口（伪接口如隧道接口或 ppp 接口）将尝试从其他接口（如以太网接口）借用 IEEE802 地址。如果没有附加 IEEE802 硬件，将使用最后的备选伪随机值 MD5(hostname) 作为链路本地地址的来源。如果这对您的使用不适合，您将需要手动配置链路本地地址。

如果接口无法处理 IPv6（如缺乏组播支持），则不会为该接口分配链路本地地址。详细信息请参阅第 2 节。

每个接口都加入了被请求的多播地址和链路本地所有节点多播地址（例如，fe80::1:ff01:6317 和 ff02::1，在接口所连接的链路上）。除了链路本地地址之外，环回地址（::1）将被分配给环回接口。此外，::1/128 和 ff01::/32 会自动添加到路由表中，环回接口还加入了节点本地多播组 ff01::1。

##### 主机上的无状态地址自动配置

在 IPv6 规范中，节点分为两类：路由器和主机。路由器转发发送给其他节点的数据包，主机不转发数据包。net.inet6.ip6.forwarding 定义了该节点是路由器还是主机（如果为 1，则为路由器；如果为 0，则为主机）。

当主机从路由器那里收到路由器通告时，主机可以通过无状态地址自动配置来自动配置自身。此行为可以通过 net.inet6.ip6.accept_rtadv 来控制（如果设置为 1，则主机会自动配置自身）。通过自动配置，接收接口的网络地址前缀（通常是全局地址前缀）被添加。默认路由也被配置。路由器定期生成路由器通告数据包。要请求相邻路由器生成 RA 数据包，主机可以发送路由器请求。要随时生成 RS 数据包，请使用 rtsol 命令。也可以使用 rtsold(8)守护程序。rtsold(8)在必要时生成路由器请求，并且非常适合移动使用（笔记本电脑）。如果希望忽略路由器通告，请使用 sysctl 将 net.inet6.ip6.accept_rtadv 设置为 0。

从路由器生成路由器通告，请使用 rtadvd(8)守护程序。

请注意，IPv6 规范假定以下项目，不符合规范的情况未指定：

* 只有主机会收听路由器通告
* 主机只有单个网络接口（除回环接口外）

因此，在路由器或多接口主机上启用 net.inet6.ip6.accept_rtadv 是不明智的。配置错误的节点可能表现出奇怪的行为（允许不符合规范的配置，适合那些希望进行一些实验的人）。

总结一下 sysctl 开关：

```
	accept_rtadv	forwarding	role of the node
	---		---		---
	0		0		host (to be manually configured)
	0		1		router
	1		0		autoconfigured host
					(spec assumes that host has single
					interface only, autoconfigured host
					with multiple interface is
					out-of-scope)
	1		1		invalid, or experimental
					(out-of-scope of spec)
```

RFC2462 对传入 RA 前缀信息选项的验证规则在 5.5.3（e）中。这是为了保护主机免受恶意（或配置错误）路由器广告非常短的前缀生存期。Jim Bound 向 ipngwg 邮件列表进行了更新（在存档中查找“(ipng 6712)”），并实施了 Jim 的更新。

查看文档中的 23.5.1.2，了解 DAD 与自动配置之间的关系。

#### 8.1.1.5. 通用隧道接口

GIF（通用接口）是用于配置隧道的伪接口。详细信息请参阅 gif(4)。当前

* v6 in v6
* v6 in v4
* 在 v6 中的 v4
* 在 v4 中的 v4

可用。使用 gifconfig(8)为 gif 接口分配物理（外部）源和目标地址。在内部和外部 IP 头部使用相同地址族（v4 中的 v4，或 v6 中的 v6）的配置是危险的。很容易配置接口和路由表以执行无限级别的隧道。请注意。

gif 可以配置为 ECN-friendly。请查看章节 23.5.4.5 了解隧道的 ECN-friendly 设置，以及如何配置 gif(4)。

如果您想要使用 gif 接口配置一个 IPv4-in-IPv6 隧道，请仔细阅读 gif(4)。您需要自动删除分配给 gif 接口的 IPv6 链路本地地址。

#### 8.1.1.6. 源地址选择

当前的源选择规则是面向范围的（有一些例外情况-请参见下文）。 对于给定的目的地，将通过以下规则选择源 IPv6 地址：

1. 如果用户明确指定了源地址（例如，通过高级 API），则使用指定的地址。
2. 如果对于外发接口分配了一个具有与目的地地址相同范围的地址（通常通过查找路由表确定），则使用该地址。 这是最典型的情况。
3. 如果没有满足上述条件的地址，请选择发送节点上一个接口分配的全局地址。
4. 如果没有满足上述条件的地址，并且目标地址是站点本地范围，请选择发送节点上一个接口分配的站点本地地址。
5. 如果没有满足上述条件的地址，请选择与目标的路由表条目关联的地址。这是最后的手段，可能会导致范围违规。

例如，对于 ff01::1，为 fe80:1::2a0:24ff:feab:839b 选择 fe80:1::200:f8ff:fe01:6317（请注意，嵌入式接口索引 - 描述在 23.5.1.3 中 - 帮助我们选择正确的源地址。这些嵌入索引不会传输到网络）。如果出站接口具有适用于范围的多个地址，则根据最长匹配原则选择源地址（规则 3）。假设出站接口提供了 2001:0DB8:808:1:200:f8ff:fe01:6317 和 2001:0DB8:9:124:200:f8ff:fe01:6317。将选择 2001:0DB8:808:1:200:f8ff:fe01:6317 作为目标 2001:0DB8:800::1 的源。

请注意，上述规则未在 IPv6 规范中记录。它被视为“由实现决定”的项目。有些情况下，我们不遵循上述规则。一个例子是连接的 TCP 会话，我们使用在 tcb 中保存的地址作为源地址。另一个例子是 Neighbor Advertisement 的源地址。根据规范（RFC2461 7.2.2），NA 的源地址应该是相应 NS 的目标地址。在这种情况下，我们遵循规范而不是上述最长匹配规则。

对于新连接（不适用规则 1 时），如果有其他选择可用，则不会选择已弃用地址（首选生存时间=0）作为源地址。如果没有其他选择可用，则废弃地址将被用作最后的选择。如果有多个废弃地址可选，则将使用上述范围规则从这些废弃地址中进行选择。如果出于某种原因希望禁止使用废弃地址，请将 net.inet6.ip6.use_deprecated 配置为 0。有关废弃地址的问题在 RFC2462 5.5.4 中有描述（注意：IETF ipngwg 正在就如何使用“废弃”地址进行一些讨论）。

#### 8.1.1.7. 巨型负载

巨型负载逐跳选项已实现，并可用于发送载荷超过 65,535 八位组的 IPv6 数据包。但目前不支持 MTU 超过 65,535 的物理接口，因此这样的负载只能在环回接口（即 lo0）上看到。

如果您想尝试巨型负载，首先必须重新配置内核，使环回接口的 MTU 超过 65,535 字节；将以下内容添加到内核配置文件中：

`options "LARGE_LOMTU" #To test jumbo payload`

然后重新编译新内核。

然后，您可以使用 ping(8) 命令的 -6、-b 和 -s 选项来测试巨型负载。必须指定 -b 选项以扩大套接字缓冲区的大小，-s 选项指定数据包的长度，应大于 65,535。例如，输入以下命令：

```
% ping -6 -b 70000 -s 68000 ::1
```

IPv6 规范要求巨型负载选项不能用于携带片段头的数据包中。如果违反此条件，将向发送方发送 ICMPv6 参数问题消息。虽然遵循规范，但通常不会因此要求导致 ICMPv6 错误消息。

当接收到一个 IPv6 包时，会检查帧长度并将其与 IPv6 报头中的有效载荷长度字段或 Jumbo Payload 选项中的值（如果有的话）进行比较。如果前者短于后者，则丢弃该包并增加统计数据。你可以使用 `-s -p ip6` 选项查看 netstat(8)命令的输出统计数据：

```
% netstat -s -p ip6
	  ip6:
		(snip)
		1 with data size < data length
```

因此，除非错误的数据包是真正的 Jumbo Payload，即其数据包大小超过 65,535 字节，否则内核不会发送 ICMPv6 错误。如上所述，目前不支持具有如此巨大 MTU 的物理接口，因此很少返回 ICMPv6 错误。

目前不支持 TCP/UDP over jumbogram。这是因为我们没有介质（除了回环）来测试这个。如果你需要这个功能，请联系我们。

IPsec 不能在超大数据包上工作。这是由于支持 AH 的规范扭曲所致，在使用超大数据包时 AH 头的大小会影响有效载荷长度，这使得验证传入带有超大有效载荷选项的数据包变得非常困难。

*BSD 对超大数据包的支持存在基本问题。我们希望解决这些问题，但我们需要更多时间来完成。举几个例子：

* 在 4.4BSD 中，mbuf pkthdr.len 字段被类型化为"int"，因此在 32 位架构 CPU 上，它将无法容纳长度> 2G 的超大数据包。如果我们希望正确地支持超大数据包，该字段必须扩展以容纳 4G + IPv6 头 + 链路层头。因此，它必须扩展至至少 int64_t（u_int32_t 是不够的）。
* 我们错误地在许多地方使用"int"来保存数据包长度。我们需要将它们转换为更大的整数类型。需要非常小心，因为在计算数据包长度时可能会发生溢出。
* 我们错误地在各个地方检查 IPv6 头部的 ip6_plen 字段作为数据包有效载荷长度。我们应该检查 mbuf pkthdr.len。ip6_input()将在输入时对巨大有效载荷选项执行完整性检查，之后我们可以安全地使用 mbuf pkthdr.len。
* TCP 代码需要在很多地方进行仔细更新，当然。

#### 8.1.1.8. 头部处理中的环路预防

IPv6 规范允许将任意数量的扩展标头放置在数据包上。 如果我们以 BSD IPv4 代码的方式实现 IPv6 数据包处理代码，则由于函数调用链过长，内核堆栈可能会溢出。 sys/netinet6 代码经过精心设计以避免内核堆栈溢出，因此 sys/netinet6 代码定义了自己的协议开关结构，称为"netinet6/ip6protosw.h"中的"struct ip6protosw"。 对于与 IPv4 部分（sys/netinet）兼容性的更新，没有进行这样的更新，但在其 pr_input()原型中添加了小更改。 因此，如果接收到具有大量 IPsec 标头的 IPsec-over-IPv4 数据包，则内核堆栈可能会崩溃。 IPsec-over-IPv6 没问题。 （当然，要处理所有这些 IPsec 标头，每个 IPsec 标头都必须通过每个 IPsec 检查。 因此，匿名攻击者将无法执行此类攻击。）

#### 8.1.1.9. ICMPv6

RFC2463 发布后，IETF ipngwg 已决定禁止针对 ICMPv6 重定向的 ICMPv6 错误数据包，以防止网络介质上的 ICMPv6 风暴。这已经实现在内核中。

#### 8.1.1.10. 应用程序

对于用户空间编程，我们支持 RFC2553、RFC2292 和即将发布的互联网草案中规定的 IPv6 套接字 API。

TCP/UDP over IPv6 is available and quite stable. You can enjoy [telnet(1)](https://man.freebsd.org/cgi/man.cgi?query=telnet&sektion=1&format=html), [ftp(1)](https://man.freebsd.org/cgi/man.cgi?query=ftp&sektion=1&format=html), [rlogin(1)](https://man.freebsd.org/cgi/man.cgi?query=rlogin&sektion=1&format=html), [rsh(1)](https://man.freebsd.org/cgi/man.cgi?query=rsh&sektion=1&format=html), [ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html), etc. These applications are protocol independent. That is, they automatically chooses IPv4 or IPv6 according to DNS.

#### 8.1.1.11. Kernel Internals

While ip_forward() calls ip_output(), ip6_forward() directly calls if_output() since routers must not divide IPv6 packets into fragments.

ICMPv6 应尽可能包含原始数据包，最大长度为 1280 字节。例如，UDP6/IP6 不可达错误应包含所有扩展头部和未更改的 UDP6 和 IP6 头部。因此，除 TCP 外，所有 IP6 功能均不将网络字节顺序转换为主机字节顺序，以保存原始数据包。

tcp_input()、udp6_input() 和 icmp6_input() 不能假设 IP6 头部紧随传输头部之后，因为存在扩展头部。因此，in6_cksum() 被实现用于处理 IP6 头部和传输头部不连续的数据包。TCP/IP6 和 UDP6/IP6 头部结构不存在于校验和计算中。

为了更轻松地处理 IP6 头部、扩展头部和传输头部，网络驱动程序现在要求将数据包存储在一个或多个内部 mbuf 中。典型的旧驱动程序为 96 到 204 字节的数据准备两个内部 mbuf，但现在此类数据包数据存储在一个外部 mbuf 中。

netstat -s -p ip6 告诉您您的驱动程序是否符合此要求。在以下示例中，“cce0” 违反了该要求。（有关更多信息，请参阅第 2 节。）

```
Mbuf statistics:
                317 one mbuf
                two or more mbuf::
                        lo0 = 8
			cce0 = 10
                3282 one ext mbuf
                0 two or more ext mbuf
```

每个输入函数在开始时调用 IP6_EXTHDR_CHECK 来检查 IP6 和其标头之间的区域是否连续。IP6_EXTHDR_CHECK 仅在 mbuf 具有 M_LOOP 标志时调用 m_pullup()，也就是说，数据包来自环回接口。对于来自物理网络接口的数据包，永远不会调用 m_pullup()。

IP 和 IP6 的重组函数从不调用 m_pullup()。

#### 8.1.1.12. IPv4 映射地址和 IPv6 通配符套接字

RFC2553 描述了 IPv4 映射地址（3.7）和 IPv6 通配符绑定套接字（3.8）的特殊行为。规范允许您：

* 通过 AF_INET6 通配符绑定套接字接受 IPv4 连接。
* 通过使用类似 ::ffff:10.1.1.1 的特殊地址形式在 AF_INET6 套接字上传输 IPv4 数据包。

但规范本身非常复杂，并未指定套接字层应如何行为。在这里，我们将前者称为“监听端”，将后者称为“发起端”，供参考之用。

您可以在相同的 port 上为两个地址族执行通配符绑定。

以下表格显示 FreeBSD 4.x 的行为。

```
listening side          initiating side
                (AF_INET6 wildcard      (connection to ::ffff:10.1.1.1)
                socket gets IPv4 conn.)
                ---                     ---
FreeBSD 4.x     configurable            supported
                default: enabled
```

以下各节将为您提供更多详细信息，以及如何配置行为。

关于监听端的评论：

看起来 RFC2553 对通配符绑定问题讨论得太少，特别是在port空间问题、故障模式和 AF_INET/INET6 通配符绑定之间的关系方面。对于这个 RFC 可能会有几种不同的解释，符合它但行为不同。因此，为了实现可移植的应用程序，您应该不假思索地假定内核的行为。使用 getaddrinfo(3)是最安全的方式。Port号空间和通配符绑定问题在 ipv6imp 邮件列表中详细讨论过，1999 年 3 月中旬，看起来没有明确的共识（即，直到实现者）。您可能想要查看邮件列表存档。

如果服务器应用程序希望接受 IPv4 和 IPv6 连接，将有两种选择。

一种是使用 AF_INET 和 AF_INET6 套接字（您将需要两个套接字）。使用 getaddrinfo(3)将 AI_PASSIVE 传递给 ai_flags，并使用 socket(2)和 bind(2)绑定返回的所有地址。通过打开多个套接字，您可以接受具有适当地址族的套接字的连接。IPv4 连接将由 AF_INET 套接字接受，IPv6 连接将由 AF_INET6 套接字接受。

另一种方法是使用一个 AF_INET6 通配符绑定套接字。使用 getaddrinfo(3)将 AI_PASSIVE 置入 ai_flags，并将 AF_INET6 置入 ai_family，并将第一个参数主机名设置为 NULL。然后使用 socket(2)和 bind(2)绑定返回的地址。（应为 IPv6 未指定地址）。您可以通过这一个套接字接受 IPv4 和 IPv6 数据包。

为了支持在 AF_INET6 通配符绑定套接字上仅支持 IPv6 流量，始终在向 AF_INET6 监听套接字发起连接时检查对等地址。如果地址是 IPv4 映射地址，则可能希望拒绝连接。可以使用 IN6_IS_ADDR_V4MAPPED()宏来检查条件。

为了更容易解决这个问题，有一个系统相关的 setsockopt(2)选项，IPV6_BINDV6ONLY，使用如下。

```
	int on;

	setsockopt(s, IPPROTO_IPV6, IPV6_BINDV6ONLY,
		   (char *)&on, sizeof (on)) < 0));
```

当此调用成功时，此套接字只接收 IPv6 数据包。

关于启动方面的评论：

给应用程序实现者的建议：为了实现一个可移植的 IPv6 应用程序（可在多个 IPv6 内核上运行），我们认为以下是成功的关键：

* 绝不要在代码中硬编码 AF_INET 或 AF_INET6。
* 在整个系统中使用 getaddrinfo(3) 和 getnameinfo(3)。永远不要使用 gethostby()、getaddrby()、inet_() 或 getipnodeby()。（为了轻松地使现有应用程序具备 IPv6 意识，有时 getipnodeby*() 会很有用。但如果可能的话，尽量重写代码以使用 getaddrinfo(3) 和 getnameinfo(3)。）
* 如果希望连接到目的地，请使用 getaddrinfo(3) 并尝试连接返回的所有目的地，就像 telnet(1) 一样。
* 一些 IPv6 堆栈带有有缺陷的 getaddrinfo(3)。将一个与您的应用程序一起提供的最小工作版本，并将其作为最后手段使用。

如果您想要为 IPv4 和 IPv6 的出站连接都使用 AF_INET6 套接字，您将需要使用 getipnodebyname(3)。当您希望尽可能少的工作将现有应用程序升级为 IPv6 兼容时，可能会选择这种方法。但请注意，这只是一个临时解决方案，因为 getipnodebyname(3)本身并不推荐，因为它根本不处理范围 IPv6 地址。对于 IPv6 名称解析，getaddrinfo(3)是首选的 API。因此，当您有时间这样做时，应重写您的应用程序以使用 getaddrinfo(3)。

在编写制作输出连接的应用程序时，如果您将 AF_INET 和 AF_INET6 视为完全独立的地址族，那么整个故事就会简单得多。{set,get}sockopt 问题会更简单，DNS 问题也会变得更简单。我们不建议您依赖 IPv4 映射地址。

##### 8.1.1.12.1. 统一的 tcp 和 inpcb 代码

FreeBSD 4.x 在 IPv4 和 IPv6 之间使用共享的 tcp 代码（来自 sys/netinet/tcp*），并使用单独的 udp4/6 代码。 它使用统一的 inpcb 结构。

可以配置平台以支持 IPv4 映射地址。 内核配置总结如下：

* 通过默认，AF_INET6 套接字在某些条件下会获取 IPv4 连接，并可以发起连接到 IPv4 目的地中的 IPv4 映射 IPv6 地址。
* 您可以像下面这样在整个系统上禁用它的 sysctl。

###### 8.1.1.12.1.1. 监听端

每个套接字都可以配置为支持特殊的 AF_INET6 通配符绑定（默认情况下已启用）。您可以像下面这样在每个套接字上禁用它。

```
	int on;

	setsockopt(s, IPPROTO_IPV6, IPV6_BINDV6ONLY,
		   (char *)&on, sizeof (on)) < 0));
```

通配符 AF_INET6 套接字仅在满足以下条件时抓取 IPv4 连接：

* 没有与 IPv4 连接匹配的 AF_INET 套接字。
* AF_INET6 套接字配置为接受 IPv4 流量，即 getsockopt(IPV6_BINDV6ONLY)返回 0。

打开/关闭顺序没有问题。

###### 8.1.1.12.1.2. 启动端

FreeBSD 4.x 支持对 IPv4 映射地址(::ffff:10.1.1.1)的传出连接，如果节点配置为支持 IPv4 映射地址。

#### 8.1.1.13. sockaddr_storage

当 RFC2553 即将最终确定时，关于如何命名 struct sockaddr_storage 成员进行了讨论。一个提议是在成员前面加上空格(如" ss_len")，因为它们不应该被修改。另一个提议是不加空格(如"ss_len")，因为我们需要直接访问这些成员。对此并没有明确的共识。

结果，RFC2553 如下定义结构 sockaddr_storage：

```
	struct sockaddr_storage {
		u_char	__ss_len;	/* address length */
		u_char	__ss_family;	/* address family */
		/* and bunch of padding */
	};
```

相反，XNET 草案如下定义：

```
	struct sockaddr_storage {
		u_char	ss_len;		/* address length */
		u_char	ss_family;	/* address family */
		/* and bunch of padding */
	};
```

1999 年 12 月达成共识，RFC2553bis 应选择后者（XNET）的定义。

当前实现符合 XNET 定义，基于 RFC2553bis 讨论。

如果你查看多个 IPv6 实现，你将能够看到两种定义。作为一个用户态程序员，最便捷的处理方式是：

1. 通过使用 GNU autoconf 确保 ss_family 和/或 ss_len 在平台上可用。
2. 将-Dss_family=ss_family 统一所有出现的地方（包括头文件）为 ss_family，或
3. 永远不要触碰__ss_family。转换为 sockaddr *并使用 sa_family，如：

    ```
    	struct sockaddr_storage ss;
    	family = ((struct sockaddr *)&ss)->sa_family
    ```

### 8.1.2. 网络驱动程序

现在，标准驱动程序需要支持以下两个项目:

1. mbuf 集群要求。在此稳定版本中，我们将 MINCLSIZE 更改为 MHLEN+1，以便使所有驱动程序的行为符合我们的预期。
2. 多播。如果 ifmcstat(8)未为接口提供任何多播组，则必须对该接口进行打补丁。

如果任何驱动程序不支持要求，则这些驱动程序不能用于 IPv6 和/或 IPsec 通信。如果您发现使用 IPv6/IPsec 时遇到任何问题，请将其报告给 FreeBSD 问题报告邮件列表。

（注意：过去我们要求所有 PCMCIA 驱动程序调用 in6_ifattach()。我们不再有此要求）

### 8.1.3. 翻译器

我们将 IPv4/IPv6 转换器分类为 4 种类型：

* 转换器 A --- 它用于过渡的早期阶段，使得从 IPv6 岛屿中的 IPv6 主机到 IPv4 海洋中的 IPv4 主机建立连接成为可能。
* 转换器 B --- 它用于过渡的早期阶段，使得从 IPv4 海洋中的 IPv4 主机到 IPv6 岛屿中的 IPv6 主机建立连接成为可能。
* 翻译者 C --- 它用于过渡的后期阶段，使得在 IPv4 岛中的 IPv4 主机与 IPv6 海洋中的 IPv6 主机建立连接成为可能。
* 翻译者 D --- 它用于过渡的后期阶段，使得在 IPv6 海洋中的 IPv6 主机与 IPv4 岛中的 IPv4 主机建立连接成为可能。

### 8.1.4. IPsec

IPsec 主要由三个组件组织。

1. 策略管理
2. 密钥管理
3. AH 和 ESP 处理

#### 8.1.4.1. 策略管理

内核实现了实验性的策略管理代码。有两种管理安全策略的方式。一种是使用 setsockopt(2) 配置每个套接字的策略。在这种情况下，策略配置在 ipsec_set_policy(3) 中描述。另一种方式是使用 PF_KEY 接口配置基于内核数据包过滤器的策略，通过 setkey(8)。

策略条目未按索引重新排序，因此在添加时条目的顺序非常重要。

#### 8.1.4.2. 密钥管理

此工具包中实现的密钥管理代码（sys/netkey）是自制的 PFKEY v2 实现。这符合 RFC2367。

用于 IPsec 协议的自制 IKE 守护程序"racoon"已包含在套件中（kame/kame/racoon）。基本上，您需要将 racoon 作为守护程序运行，然后设置一个需要密钥的策略（如 ping -P 'out ipsec esp/transport//use' ）。内核将根据需要与 racoon 守护程序联系以交换密钥。

#### 8.1.4.3. AH 和 ESP 处理

IPsec 模块实现为标准 IPv4/IPv6 处理的“钩子”。在发送数据包时，ip{,6}_output()会检查是否需要进行 ESP/AH 处理，方法是检查是否找到匹配的 SPD（安全策略数据库）。如果需要 ESP/AH，将调用{esp,ah}{4,6}_output()，并相应更新 mbuf。在接收数据包时，将根据协议号调用{esp,ah}4_input()，即(*inetsw[proto])()。{esp,ah}4_input()将对数据包进行解密/验证其真实性，并为 ESP/AH 去除级联的标头和填充。在数据包接收时安全地去除 ESP/AH 标头是安全的，因为我们永远不会原样使用接收到的数据包。

通过使用 ESP/AH，TCP4/6 的有效数据段大小将受到 ESP/AH 插入的额外级联标头的影响。我们的代码会处理这种情况。

基本加密函数可以在目录"sys/crypto"中找到。ESP/AH 转换在{esp,ah}_core.c 中列出了包装函数。如果您希望添加一些算法，请在{esp,ah}_core.c 中添加包装函数，并将您的加密算法代码添加到 sys/crypto 中。

隧道模式在此版本中部分受支持，有以下限制：

* IPsec 隧道未与 GIF 通用隧道接口结合。这需要非常小心，因为我们可能会在 ip_output()和 tunnelifp→if_output()之间创建一个无限循环。关于统一它们是否更好，意见各不相同。
* MTU 和不分段比特（IPv4）的考虑需要更多检查，但基本上工作正常。
* AH 隧道的认证模型必须重新审视。我们将需要改进策略管理引擎，最终。

#### 8.1.4.4. RFC 和 ID 的符合性

The IPsec code in the kernel conforms (or, tries to conform) to the following standards:

"old IPsec" specification documented in rfc182[5-9].txt

在 rfc240[1-6].txt、rfc241[01].txt、rfc2451.txt 和 draft-mcdonald-simple-ipsec-api-01.txt（草案已过期，但您可以从 ftp://ftp.kame.net/pub/internet-drafts/下载）中记录了新 IPsec 规范。 （注意：IKE 规范，rfc241[7-9].txt 在用户空间中实现为"racoon" IKE 守护程序）

目前支持的算法有：

* 旧 IPsec AH

  * 空 crypto 校验和（无文档，仅用于调试）
  * 带有 128 位 crypto 校验和的 keyed MD5（rfc1828.txt）
  * 带有 128 位 crypto 校验和的 keyed SHA1（无文档）
  * HMAC MD5 具有 128 位加密校验和 (rfc2085.txt)
  * HMAC SHA1 具有 128 位加密校验和 (没有文档)
* 旧的 IPsec ESP

  * 空加密（无文档，类似于 rfc2410.txt）
  * DES-CBC 模式（rfc1829.txt）
* 新 IPsec AH

  * 空的加密校验和（无文档，仅用于调试）
  * 带 96 位加密校验和的密钥 MD5（无文档）
  * 带 96 位加密校验和的密钥 SHA1（无文档）
  * 带有 96 位加密校验和的 HMAC MD5 (rfc2403.txt)
  * 带有 96 位加密校验和的 HMAC SHA1 (rfc2404.txt)
* 新的 IPsec ESP

  * 空加密（rfc2410.txt）
  * 带派生 IV 的 DES-CBC（draft-ietf-ipsec-ciph-des-derived-01.txt，草案已过期）
  * 带明确 IV 的 DES-CBC（rfc2405.txt）
  * 用明文 IV 的 3DES-CBC（rfc2451.txt）
  * BLOWFISH CBC（rfc2451.txt）
  * CAST128 CBC（rfc2451.txt）
  * RC5 CBC（rfc2451.txt）
  * 可以与上述每一个结合：

    * 使用 HMAC-MD5（96 位）的 ESP 认证
    * ESP 使用 HMAC-SHA1(96 位)进行身份验证

不支持以下算法:

* 老旧的 IPsec AH

  * HMAC MD5 带 128 位密码校验和+64 位重放预防（rfc2085.txt）
  * 带 160 位密码校验和的密钥 SHA1+32 位填充（rfc1852.txt）

IPsec（在内核中）和 IKE（在用户空间中作为"racoon"）已在多个互操作性测试活动中进行了测试，并且已知与许多其他实现很好地互操作。此外，当前的 IPsec 实现对 RFC 中记录的 IPsec 密码算法具有相当广泛的覆盖范围（我们仅涵盖没有知识产权问题的算法）。

#### 8.1.4.5. IPsec 隧道上的 ECN 考虑

支持与 draft-ipsec-ecn-00.txt 中描述的 ECN 友好的 IPsec 隧道。

RFC2401 中描述了普通的 IPsec 隧道。在封装时，IPv4 的 TOS 字段（或 IPv6 的流量类字段）将从内部 IP 头复制到外部 IP 头。在解封装时，外部 IP 头将被简单丢弃。解封装规则与 ECN 不兼容，因为外部 IP 的 TOS/流量类字段上的 ECN 位将丢失。

要使 IPsec 隧道支持 ECN，我们应修改封装和解封装过程。这在 http://www.aciri.org/floyd/papers/draft-ipsec-ecn-00.txt 的第 3 章中有描述。

通过将 net.inet.ipsec.ecn（或 net.inet6.ipsec6.ecn）设置为某个值，IPsec 隧道实现可以提供三种行为：

* RFC2401：不考虑 ECN（sysctl 值为-1）
* 禁止 ECN (sysctl 值为 0)
* 允许 ECN (sysctl 值为 1)

请注意，行为可在每个节点进行配置，而不是每个 SA 进行配置（draft-ipsec-ecn-00 希望每个 SA 配置，但对我来说太复杂了）。

行为如下总结（详细信息请参阅源代码）：

```
encapsulate                     decapsulate
                ---                             ---
RFC2401         copy all TOS bits               drop TOS bits on outer
                from inner to outer.            (use inner TOS bits as is)

ECN forbidden   copy TOS bits except for ECN    drop TOS bits on outer
                (masked with 0xfc) from inner   (use inner TOS bits as is)
                to outer.  set ECN bits to 0.

ECN allowed     copy TOS bits except for ECN    use inner TOS bits with some
                CE (masked with 0xfe) from      change.  if outer ECN CE bit
                inner to outer.                 is 1, enable ECN CE bit on
                set ECN CE bit to 0.            the inner.
```

配置的一般策略如下：

* 如果 IPsec 隧道端点都支持 ECN 友好行为，最好将两端都配置为“允许 ECN”（sysctl 值为 1）。
* 如果另一端对 TOS 位非常严格，请使用"RFC2401"（sysctl 值为-1）。
* 在其他情况下，请使用"ECN 禁止"（sysctl 值为 0）。

默认行为是"ECN 禁止"（sysctl 值为 0）。

有关更多信息，请参阅：

http://www.aciri.org/floyd/papers/draft-ipsec-ecn-00.txt，RFC2481（显式拥塞通知），src/sys/netinet6/{ah,esp}_input.c

（感谢 Kenjiro Cho kjc@csl.sony.co.jp 进行详细分析）

#### 8.1.4.6. 互操作性

这里是 KAME 代码过去测试 IPsec/IKE 互操作性的一些平台。请注意，双方可能已经修改了它们的实现，所以只需将以下列表用于参考目的。

Altiga, Ashley-laurent (vpcom.com), Data Fellows (F-Secure), Ericsson ACC, FreeS/WAN, 日立, IBM AIX®, IIJ, Intel, Microsoft® Windows NT®, NIST (Linux IPsec + plutoplus), Netscreen, OpenBSD, RedCreek, Routerware, SSH, Secure Computing, Soliton, 东芝, VPNet, 雅马哈 RT100i
