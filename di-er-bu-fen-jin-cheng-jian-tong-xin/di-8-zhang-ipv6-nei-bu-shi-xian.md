# 第 8 章 IPv6 内部实现

## 8.1. IPv6/IPsec 实现

本节将说明与 IPv6 和 IPsec 相关的实现细节。这些功能源自 [KAME 项目](http://www.kame.net/)。

### 8.1.1. IPv6

#### 8.1.1.1. 标准遵循情况

与 IPv6 相关的功能符合，或尽量符合最新的 IPv6 标准。为了将来参考，我们在下方列出一些相关文档（*注意*：这不是完整列表 —— 太难维护了……）。

如需详细信息，请参阅文档中相应章节、RFC、手册页或源代码中的注释。

TAHI 项目在 KAME STABLE 套件上进行了标准符合性测试。测试结果可见于 [http://www.tahi.org/report/KAME/](http://www.tahi.org/report/KAME/)。我们也曾参与新罕布什尔大学 IOL 的测试（[http://www.iol.unh.edu/](http://www.iol.unh.edu/)），测试对象为旧版本快照。

* RFC1639：FTP 在大地址记录上的操作（FOOBAR）

  * 推荐使用 RFC2428 代替 RFC1639。FTP 客户端会首先尝试 RFC2428，失败后才尝试 RFC1639。
* RFC1886：支持 IPv6 的 DNS 扩展
* RFC1933：IPv6 主机与路由器的过渡机制

  * 不支持 IPv4 兼容地址。
  * 不支持自动隧道（RFC 第 4.3 节中描述）。
  * [gif(4)](https://man.freebsd.org/cgi/man.cgi?query=gif&sektion=4&format=html) 接口以通用方式实现了 IPv\[46]-over-IPv\[46] 隧道，覆盖了该规范中的“配置隧道”。详情见本文件第 [23.5.1.5](https://docs.freebsd.org/en/books/developers-handbook/ipv6/#gif) 节。
* RFC1981：IPv6 的路径 MTU 发现
* RFC2080：IPv6 的 RIPng

  * usr.sbin/route6d 提供支持。
* RFC2292：IPv6 的高级套接字 API

  * 有关支持的库函数/内核 API，请见 **sys/netinet6/ADVAPI**。
* RFC2362：协议无关多播 - 稀疏模式（PIM-SM）

  * RFC2362 定义了 PIM-SM 的数据包格式。**draft-ietf-pim-ipv6-01.txt** 基于此撰写。
* RFC2373：IPv6 地址结构

  * 支持节点所需地址，并符合作用域要求。
* RFC2374：IPv6 可聚合全局单播地址格式

  * 支持 64 位接口 ID 长度。
* RFC2375：IPv6 多播地址分配

  * 用户空间应用程序使用 RFC 中分配的知名地址。
* RFC2428：IPv6 和 NAT 的 FTP 扩展

  * 推荐使用 RFC2428 代替 RFC1639。FTP 客户端会首先尝试 RFC2428，失败后再尝试 RFC1639。
* RFC2460：IPv6 规范
* RFC2461：IPv6 邻居发现

  * 详情见本文件第 [23.5.1.2](https://docs.freebsd.org/en/books/developers-handbook/ipv6/#neighbor-discovery) 节。
* RFC2462：IPv6 无状态地址自动配置

  * 详情见本文件第 [23.5.1.4](https://docs.freebsd.org/en/books/developers-handbook/ipv6/#ipv6-pnp) 节。
* RFC2463：IPv6 的 ICMPv6 规范

  * 详情见本文件第 [23.5.1.9](https://docs.freebsd.org/en/books/developers-handbook/ipv6/#icmpv6) 节。
* RFC2464：以太网上的 IPv6 分组传输
* RFC2465：IPv6 的 MIB：文本约定与常规组

  * 所需统计数据由内核收集。实际的 IPv6 MIB 支持通过 ucd-snmp 补丁包提供。
* RFC2466：IPv6 的 MIB：ICMPv6 组

  * 所需统计数据由内核收集。实际的 IPv6 MIB 支持通过 ucd-snmp 补丁包提供。
* RFC2467：FDDI 网络上的 IPv6 分组传输
* RFC2497：ARCnet 网络上的 IPv6 分组传输
* RFC2553：IPv6 的基本套接字接口扩展

  * 支持 IPv4 映射地址（第 3.7 节）和 IPv6 通配绑定套接字的特殊行为（第 3.8 节）。详情见本文件第 [23.5.1.12](https://docs.freebsd.org/en/books/developers-handbook/ipv6/#ipv6-wildcard-socket) 节。
* RFC2675：IPv6 Jumbo 报文

  * 详情见本文件第 [23.5.1.7](https://docs.freebsd.org/en/books/developers-handbook/ipv6/#ipv6-jumbo) 节。
* RFC2710：IPv6 的多播监听器发现（MLD）
* RFC2711：IPv6 路由器提醒选项
* **draft-ietf-ipngwg-router-renum-08**：IPv6 路由器重新编号
* **draft-ietf-ipngwg-icmp-namelookups-02**：通过 ICMP 的 IPv6 名称查询
* **draft-ietf-ipngwg-icmp-name-lookups-03**：通过 ICMP 的 IPv6 名称查询
* **draft-ietf-pim-ipv6-01.txt**：IPv6 的 PIM

  * [pim6dd(8)](https://man.freebsd.org/cgi/man.cgi?query=pim6dd&sektion=8&format=html) 实现了密集模式。[pim6sd(8)](https://man.freebsd.org/cgi/man.cgi?query=pim6sd&sektion=8&format=html) 实现了稀疏模式。
* **draft-itojun-ipv6-tcp-to-anycast-00**：断开面向 IPv6 Anycast 地址的 TCP 连接
* **draft-yamamoto-wideipv6-comm-model-00**

  * 详情见本文件第 [23.5.1.6](https://docs.freebsd.org/en/books/developers-handbook/ipv6/#ipv6-sas) 节。
* **draft-ietf-ipngwg-scopedaddr-format-00.txt**：IPv6 作用域地址格式扩展

#### 8.1.1.2. 邻居发现

邻居发现相当稳定。目前支持地址解析、重复地址检测（DAD）和邻居不可达检测。近期我们将会向内核中添加代理邻居通告支持，并提供无请求邻居通告的发送命令，作为管理员工具。

如果 DAD 失败，该地址将被标记为“重复”，并产生日志信息输出到 syslog（通常也会输出到控制台）。可以使用 [ifconfig(8)](https://man.freebsd.org/cgi/man.cgi?query=ifconfig&sektion=8&format=html) 查看“重复”标记。检查并处理 DAD 失败是管理员的责任。该行为应在近期得到改进。

部分网络驱动在混杂模式下会将多播数据包回送给自身，即使指示不要这样做。在这种情况下，DAD 可能会失败，因为 DAD 引擎会看到来自本节点的 NS 入站数据包，并将其视为重复地址的信号。你可以查看 sys/netinet6/nd6\_nbr.c 文件中的 nd6\_dad\_timer() 函数里用 `#if` 标记为“heuristics”的代码作为变通方案（注意，“heuristics”部分的代码不符合规范）。

邻居发现规范（RFC2461）没有说明以下情况中邻居缓存的处理方式：

1. 尚无邻居缓存条目时，节点收到未携带链路层地址的未经请求的 RS/NS/NA/重定向数据包；
2. 在无链路层地址的介质上如何处理邻居缓存（我们仍需要邻居缓存条目来存储 IsRouter 位）。

对于第一种情况，我们根据 IETF ipngwg 邮件列表中的讨论实现了一个变通方案。详情见源代码中的注释和邮件列表中从 1999 年 2 月 6 日（IPng 7155）开始的讨论线程。

IPv6 的“链路内”判断规则（RFC2461）与 BSD 网络代码中的假设大相径庭。目前尚不支持默认路由器列表为空时的“链路内”判断规则（RFC2461，第 5.2 节，第 2 段最后一句 —— 请注意，该规范在多个地方混用了“host”和“node”的含义）。

为避免可能的 DoS 攻击与无限循环，我们目前仅接受 ND 数据包中的前 10 个选项。因此，如果 RA 附带了 20 个前缀选项，只有前 10 个会被识别。如果这对你造成困扰，请在 FREEBSD-CURRENT 邮件列表中提出，或自行修改 **sys/netinet6/nd6.c** 中的 nd6\_maxndopt 变量。如果用户需求足够强烈，我们也可能为此变量提供 sysctl 控制项。


#### 8.1.1.3. 作用域索引（Scope Index）

IPv6 使用具有限定作用域的地址。因此，在使用 IPv6 地址时，指定作用域索引（对于链路本地地址是接口索引，对于站点本地地址是站点索引）非常重要。如果没有作用域索引，具有限定作用域的 IPv6 地址对于内核来说是模糊的，内核将无法确定数据包的出接口。

普通的用户态应用程序应该使用高级 API（RFC2292）来指定作用域索引或接口索引。为实现类似目的，RFC2553 在 `sockaddr_in6` 结构中定义了 `sin6_scope_id` 成员。然而，`sin6_scope_id` 的语义相当模糊。如果你关心应用程序的可移植性，我们建议你使用高级 API 而不是 `sin6_scope_id`。

在内核中，对于链路本地作用域的地址，接口索引嵌入在 IPv6 地址的第二个 16 位字（即第 3 和第 4 字节）中。例如，你可能会在路由表和接口地址结构（`struct in6_ifaddr`）中看到如下形式的地址：

```sh
fe80:1::200:f8ff:fe01:6317
```

上面的地址是一个链路本地单播地址，属于接口标识符为 1 的网络接口。嵌入索引的方式使我们能够在多个接口上有效地识别 IPv6 链路本地地址，同时只需很少的代码修改。

路由守护进程和配置程序，例如 [route6d(8)](https://man.freebsd.org/cgi/man.cgi?query=route6d&sektion=8&format=html) 和 [ifconfig(8)](https://man.freebsd.org/cgi/man.cgi?query=ifconfig&sektion=8&format=html)，需要处理“嵌入的”作用域索引。这些程序使用路由套接字和 ioctl（如 SIOCGIFADDR\_IN6），内核 API 将返回第二个 16 位字已填充的 IPv6 地址。这些 API 用于操作内核内部结构。使用这些 API 的程序必须为内核间差异做好准备。

当你在命令行中指定具有作用域的地址时，**切勿**使用嵌入形式（例如 `ff02:1::1` 或 `fe80:2::fedc`）。这是不应该被支持的。应始终使用标准形式，如 `ff02::1` 或 `fe80::fedc`，并使用命令行选项指定接口（如 `ping -6 -I ne0 ff02::1`）。一般而言，如果某个命令没有用于指定出接口的命令行选项，那么它尚未准备好处理具有作用域的地址。这似乎与 IPv6 支持“牙医办公室”（dentist office）情形的初衷相悖。我们认为这些规范仍需改进。

部分用户态工具支持扩展的 IPv6 数值语法，如 **draft-ietf-ipngwg-scopedaddr-format-00.txt** 中所述。你可以通过使用出接口的名称来指定出链路，例如 `fe80::1%ne0`。通过这种方式，你可以轻松指定链路本地作用域地址。

要在程序中使用此扩展，你需要使用 [getaddrinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getaddrinfo&sektion=3&format=html) 和 [getnameinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getnameinfo&sektion=3&format=html)，并带上 `NI_WITHSCOPEID`。当前的实现假设链路与接口之间是一对一的关系，这一假设比规范中的要求更为严格。

#### 8.1.1.4. 即插即用（Plug and Play）

大多数 IPv6 的无状态地址自动配置功能是在内核中实现的。邻居发现（Neighbor Discovery）功能整体上由内核实现。主机端对路由通告（RA）的输入在内核中实现；终端主机的路由请求（RS）输出、路由器的 RS 输入和 RA 输出则在用户态中实现。

##### 8.1.1.4.1. 链路本地地址和特殊地址的分配

IPv6 的链路本地地址是根据 IEEE802 地址（即以太网 MAC 地址）生成的。当接口变为激活状态（`IFF_UP`）时，会自动分配 IPv6 链路本地地址。同时，会将链路本地地址的直接路由加入路由表。

以下是 `netstat` 命令的输出示例：

```sh
Internet6:
Destination                   Gateway                   Flags      Netif Expire
fe80:1::%ed0/64               link#1                    UC          ed0
fe80:2::%ep0/64               link#2                    UC          ep0
```

对于没有 IEEE802 地址的接口（如隧道接口或 ppp 接口等伪接口），将尽可能借用其他接口（如以太网接口）的 IEEE802 地址。如果没有任何 IEEE802 硬件可用，则会以 `MD5(主机名)` 的伪随机值作为链路本地地址的来源作为最后手段。如果这种方式不适用于你的需求，你需要手动配置链路本地地址。

如果某个接口无法处理 IPv6（如不支持多播），则不会为该接口分配链路本地地址。详情请参见第 2 节。

每个接口都会加入 solicited 多播地址和链路本地 all-nodes 多播地址（例如接口所在链路上的 `fe80::1:ff01:6317` 和 `ff02::1`）。除了链路本地地址外，还会将环回地址（`::1`）分配给环回接口。同时，`::1/128` 和 `ff01::/32` 会自动添加到路由表中，环回接口也会加入节点本地多播组 `ff01::1`。

##### 8.1.1.4.2. 主机上的无状态地址自动配置

在 IPv6 规范中，节点分为两类：**路由器（router）** 和 **主机（host）**。路由器转发目的地为其他节点的数据包，主机则不转发数据包。`net.inet6.ip6.forwarding` 控制该节点的角色（若为 1 则为路由器，若为 0 则为主机）。

当主机接收到来自路由器的路由通告（RA）时，主机可以通过无状态地址自动配置为自己配置地址。该行为由 `net.inet6.ip6.accept_rtadv` 控制（设为 1 时，主机启用自动配置）。通过自动配置，会为接收接口添加网络地址前缀（通常是全局地址前缀），并配置默认路由。路由器会周期性地发送 RA 包。若希望请求邻居路由器发送 RA 包，主机可以发送路由请求（RS）。可随时使用 `rtsol` 命令发送 RS 包。系统也提供了 [rtsold(8)](https://man.freebsd.org/cgi/man.cgi?query=rtsold&sektion=8&format=html) 守护进程。\[rtsold(8)] 会在需要时自动发送路由请求，非常适合移动设备（如笔记本电脑）的使用场景。如果希望忽略路由通告，可以通过 `sysctl` 将 `net.inet6.ip6.accept_rtadv` 设为 0。

若需从路由器生成路由通告，请使用 [rtadvd(8)](https://man.freebsd.org/cgi/man.cgi?query=rtadvd&sektion=8&format=html) 守护进程。

请注意，IPv6 规范默认以下前提条件，对于不符合这些前提的情况，规范未做详细定义：

* 只有主机会监听路由通告；
* 主机仅有一个网络接口（不包括环回接口）。

因此，不建议在路由器或多接口主机上启用 `net.inet6.ip6.accept_rtadv`。配置错误的节点可能会表现异常（对于想做实验的人来说，这种非规范配置是允许的）。

以下是 sysctl 控制选项的总结：

```sh
accept_rtadv	forwarding	节点角色
-----------	-----------	-----------------------
0		0		主机（需手动配置）
0		1		路由器
1		0		自动配置主机
				（规范假定主机仅有一个接口；
				 多接口自动配置主机超出规范范围）
1		1		无效或实验用途
				（超出规范范围）
```

RFC2462 第 5.5.3 (e) 节对收到的 RA 中前缀信息选项规定了验证规则，以防止主机受到恶意（或配置错误）的路由器发送极短的前缀生命周期影响。Jim Bound 曾在 ipngwg 邮件列表中提出一项更新（可在归档中查找 “(ipng 6712)”），此更新已被实现。

关于 DAD（重复地址检测）与自动配置的关系，请参见本手册的 [23.5.1.2](https://docs.freebsd.org/en/books/developers-handbook/ipv6/#neighbor-discovery) 节。

#### 8.1.1.5. 通用隧道接口

GIF（通用接口，Generic InterFace）是一种用于配置隧道的伪接口。详情请参见 [gif(4)](https://man.freebsd.org/cgi/man.cgi?query=gif&sektion=4&format=html)。目前支持以下配置：

* v6 封装于 v6（v6 in v6）
* v6 封装于 v4（v6 in v4）
* v4 封装于 v6（v4 in v6）
* v4 封装于 v4（v4 in v4）

可以使用 [gifconfig(8)](https://man.freebsd.org/cgi/man.cgi?query=gifconfig&sektion=8&format=html) 为 gif 接口分配物理（外层）源地址和目标地址。若内外层 IP 头使用相同地址族（如 v4 封装于 v4，或 v6 封装于 v6），这种配置是危险的。非常容易配置出无限层隧道的接口与路由表。**请务必注意**。

gif 可以配置为支持 ECN（显式拥塞通知）友好行为。关于隧道的 ECN 友好性详见 [23.5.4.5](https://docs.freebsd.org/en/books/developers-handbook/ipv6/#ipsec-ecn)，以及如何配置详见 [gif(4)](https://man.freebsd.org/cgi/man.cgi?query=gif&sektion=4&format=html)。

如果你希望使用 gif 接口配置 IPv4 封装于 IPv6 的隧道，请仔细阅读 [gif(4)](https://man.freebsd.org/cgi/man.cgi?query=gif&sektion=4&format=html)。你需要手动移除自动分配给 gif 接口的 IPv6 链路本地地址。

#### 8.1.1.6. 源地址选择

当前的源地址选择规则是基于作用域（scope）优先的（但也有一些例外，见下文）。对于一个给定的目的地址，IPv6 源地址的选择遵循以下规则：

1. 如果用户显式指定了源地址（例如通过高级 API），则使用指定的地址。
2. 如果出口接口（通常通过查询路由表确定）上分配有与目的地址具有相同作用域的地址，则使用该地址。
   这是最常见的情况。
3. 如果没有符合上述条件的地址，则选择发送节点的任一接口上分配的全局地址。
4. 如果仍没有符合条件的地址，且目的地址是站点本地（site local）作用域，则选择任一接口上分配的站点本地地址。
5. 如果仍无法满足条件，则选择与目标地址的路由表项相关联的地址。此为最后手段，可能导致作用域违规。

例如，目的地址为 ff01::1 时会选取 ::1 作为源地址；目的地址为 fe80:1::2a0:24ff\:feab:839b 时会选取 fe80:1::200\:f8ff\:fe01:6317（注意嵌入的接口索引，详见 [23.5.1.3](https://docs.freebsd.org/en/books/developers-handbook/ipv6/#ipv6-scope-index)，帮助我们选择正确的源地址。这些嵌入索引不会出现在网络数据包中）。若出口接口上存在多个具有相同作用域的地址，将基于最长匹配原则选择（规则 3）。假设出口接口上配置了 2001:0DB8:808:1:200\:f8ff\:fe01:6317 与 2001:0DB8:9:124:200\:f8ff\:fe01:6317，那么对于目的地址 2001:0DB8:800::1，将选用 2001:0DB8:808:1:200\:f8ff\:fe01:6317 作为源地址。

需要注意的是，上述规则未在 IPv6 规范中定义，属于“由实现决定”的项目。以下是一些不遵循上述规则的情况：
例如建立 TCP 连接时，会使用保存在 tcb 中的地址作为源地址；再如发送邻居通告（Neighbor Advertisement）时，规范（RFC2461 第 7.2.2 节）要求 NA 的源地址为对应 NS 的目标地址。在这种情况下我们遵循规范，而非最长匹配规则。

对于新连接（规则 1 不适用的情况），如果存在已弃用的地址（preferred lifetime = 0），则在其他地址可用时不会选取它们作为源地址；若无其他选择，则会作为最后手段使用。如果存在多个可用的已弃用地址，将依照上述作用域规则选择其中之一。若你希望禁止使用已弃用的地址，可将 sysctl 参数 `net.inet6.ip6.use_deprecated` 设置为 0。与已弃用地址相关的问题详见 RFC2462 第 5.5.4 节（注意：IETF ipngwg 正在讨论如何使用“已弃用”地址）。

#### 8.1.1.7. 巨大负载（Jumbo Payload）

已实现了巨大负载跳跃式选项（Jumbo Payload hop-by-hop option），可以用于发送负载大于 65,535 字节的 IPv6 数据包。但目前没有支持超过 65,535 字节 MTU 的物理接口，因此此类负载仅能在环回接口（即 lo0）上看到。

如果你希望尝试巨大负载，首先需要重新配置内核，使环回接口的 MTU 大于 65,535 字节；在内核配置文件中添加以下内容：

`options "LARGE_LOMTU"  # 测试巨大负载`

然后重新编译新内核。

之后，你可以通过 [ping(8)](https://man.freebsd.org/cgi/man.cgi?query=ping&sektion=8&format=html) 命令测试巨大负载，使用 -6、-b 和 -s 选项。必须指定 -b 选项以增大套接字缓冲区大小，-s 选项指定数据包的长度，应该大于 65,535 字节。例如，输入以下命令：

```sh
% ping -6 -b 70000 -s 68000 ::1
```

IPv6 规范要求，如果数据包中携带了分片头，则不得使用巨大负载选项。如果违反此规定，必须向发送方发送 ICMPv6 参数问题消息。尽管规范如此，但通常无法看到由于此要求引发的 ICMPv6 错误。

当收到 IPv6 数据包时，会检查数据帧长度并与 IPv6 头中的负载长度字段或巨大负载选项中的长度（如果有）进行比较。如果前者小于后者，则丢弃数据包并增加统计计数。你可以通过 [netstat(8)](https://man.freebsd.org/cgi/man.cgi?query=netstat&sektion=8&format=html) 命令与 `-s -p ip6` 选项查看统计信息：

```sh
% netstat -s -p ip6
	  ip6:
		(省略)
		1 with data size < data length
```

因此，内核不会发送 ICMPv6 错误，除非数据包确实是一个巨大负载，即其包大小大于 65,535 字节。如上所述，目前没有支持如此大 MTU 的物理接口，因此 ICMPv6 错误的返回几乎不可能发生。

目前不支持在巨大负载（jumbogram）上使用 TCP/UDP。这是因为除了环回接口外，我们没有其他介质来进行测试。如果你有此需求，请联系我们。

IPsec 不支持巨大负载。这是因为在支持 AH（认证头）与巨大负载时存在一些规范上的问题（AH 头大小影响负载长度，这使得认证传入数据包时处理巨大负载选项以及 AH 变得非常困难）。

在 \*BSD 系统中，支持巨大负载存在一些根本性问题。我们希望解决这些问题，但需要更多时间来完成。列举几个问题：

* 在 4.4BSD 中，mbuf 的 pkthdr.len 字段被定义为“int”，因此它无法在 32 位架构的 CPU 上保存大于 2G 的巨大负载。如果我们希望正确支持巨大负载，必须扩展该字段以支持 4G + IPv6 头 + 链路层头。因此，该字段必须扩展为至少 `int64_t`（`u_int32_t` 不足够）。
* 我们错误地在许多地方使用“int”来保存数据包长度。需要将它们转换为更大的整数类型，这需要非常小心，因为在数据包长度计算过程中可能会发生溢出。
* 我们错误地在多个地方检查 IPv6 头的 `ip6_plen` 字段来确定数据包负载长度。实际上应检查 mbuf 的 pkthdr.len。`ip6_input()` 会在输入时对巨大负载选项进行合理性检查，之后我们可以安全地使用 mbuf 的 pkthdr.len。
* 当然，TCP 代码也需要在许多地方进行仔细更新。

#### 8.1.1.8. 头部处理中的循环防止

IPv6 规范允许将任意数量的扩展头添加到数据包中。如果我们按照 BSD 的 IPv4 代码实现 IPv6 数据包处理，内核堆栈可能会因为长时间的函数调用链而导致溢出。sys/netinet6 代码经过精心设计，避免了内核堆栈溢出，因此 sys/netinet6 代码定义了自己的协议切换结构，即 "struct ip6protosw"（参见 **netinet6/ip6protosw\.h**）。IPv4 部分（sys/netinet）没有进行类似的更新以保持兼容性，但它对 pr\_input() 原型进行了小幅修改。因此，"struct ipprotosw" 也被定义了。结果是，如果接收到带有大量 IPsec 头的 IPsec-over-IPv4 数据包，内核堆栈可能会溢出。但 IPsec-over-IPv6 是安全的。（当然，为了处理这些 IPsec 头，每个 IPsec 头必须通过每个 IPsec 检查，因此匿名攻击者无法利用这种攻击。）

#### 8.1.1.9. ICMPv6

RFC2463 发布后，IETF ipngwg 决定禁止对 ICMPv6 重定向消息发送 ICMPv6 错误数据包，以防止网络介质上发生 ICMPv6 风暴。此项已在内核中实现。

#### 8.1.1.10. 应用程序

对于用户空间编程，我们支持 IPv6 套接字 API，符合 RFC2553、RFC2292 和即将发布的互联网草案。

IPv6 上的 TCP/UDP 可用且相当稳定。你可以使用 [telnet(1)](https://man.freebsd.org/cgi/man.cgi?query=telnet&sektion=1&format=html)、[ftp(1)](https://man.freebsd.org/cgi/man.cgi?query=ftp&sektion=1&format=html)、[rlogin(1)](https://man.freebsd.org/cgi/man.cgi?query=rlogin&sektion=1&format=html)、[rsh(1)](https://man.freebsd.org/cgi/man.cgi?query=rsh&sektion=1&format=html)、[ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html) 等应用程序。这些应用程序是协议独立的，即它们会根据 DNS 自动选择 IPv4 或 IPv6。

#### 8.1.1.11. 内核内部

虽然 `ip_forward()` 调用 `ip_output()`，但 `ip6_forward()` 直接调用 `if_output()`，因为路由器不应将 IPv6 数据包分割成分片。

ICMPv6 应尽可能包含原始数据包，直到 1280 字节。例如，UDP6/IP6 端口不可达错误应该包含所有扩展头和 **未修改** 的 UDP6 和 IP6 头。因此，除了 TCP 之外，所有 IP6 函数都不会将网络字节序转换为主机字节序，以保留原始数据包。

`tcp_input()`、`udp6_input()` 和 `icmp6_input()` 不能假设 IP6 头部紧接着传输层头部，因为存在扩展头。因此，`in6_cksum()` 被实现用于处理 IP6 头部和传输头部不连续的数据包。TCP/IP6 和 UDP6/IP6 头结构并不存在于校验和计算中。

为了便于处理 IP6 头部、扩展头部和传输头部，网络驱动程序现在要求将数据包存储在一个内部 mbuf 或一个或多个外部 mbuf 中。旧版驱动程序通常为 96 - 204 字节的数据准备两个内部 mbuf，但现在这类数据包数据会存储在一个外部 mbuf 中。

`netstat -s -p ip6` 命令可以告诉你你的驱动程序是否符合这一要求。在下面的例子中，"cce0" 不符合要求。（更多信息，请参阅第 2 节。）

```c
Mbuf statistics:
                317 one mbuf
                two or more mbuf::
                        lo0 = 8
			cce0 = 10
                3282 one ext mbuf
                0 two or more ext mbuf
```

每个输入函数在开始时调用 `IP6_EXTHDR_CHECK` 来检查 IP6 头与其扩展头之间的区域是否连续。如果 mbuf 设置了 `M_LOOP` 标志，即数据包来自环回接口，则 `IP6_EXTHDR_CHECK` 会调用 `m_pullup()`；否则，来自物理网络接口的数据包则不会调用 `m_pullup()`。

IP 和 IP6 的重组函数都不会调用 `m_pullup()`。

#### 8.1.1.12. IPv4 映射地址与 IPv6 通配符套接字

RFC2553 描述了 IPv4 映射地址（3.7）和 IPv6 通配符绑定套接字的特殊行为（3.8）。该规范允许你：

* 通过 AF\_INET6 通配符绑定套接字接收 IPv4 连接。
* 通过使用类似 `::ffff:10.1.1.1` 这样的地址格式，通过 AF\_INET6 套接字传输 IPv4 数据包。

但该规范本身非常复杂，并未明确指定套接字层应该如何处理。我们在此将前者称为“监听端”，将后者称为“发起端”，以供参考。

你可以在同一个端口上对两个地址族进行通配符绑定。

下表展示了 FreeBSD 4.x 的行为：

```
listening side          initiating side
                (AF_INET6 wildcard      (connection to ::ffff:10.1.1.1)
                socket gets IPv4 conn.)
                ---                     ---
FreeBSD 4.x     configurable            supported
                default: enabled
```

接下来的章节将提供更多详细信息，以及如何配置这些行为。

关于监听端的注释：

看起来 RFC2553 对于通配符绑定问题，尤其是端口空间问题、失败模式以及 AF\_INET/INET6 通配符绑定之间的关系，讨论得较少。对于此 RFC，可能有多个不同的解释，它们符合规范但行为不同。因此，为了实现可移植的应用程序，你应该对内核中的行为不作假设。使用 [getaddrinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getaddrinfo&sektion=3&format=html) 是最安全的方法。端口号空间和通配符绑定问题曾在 1999 年 3 月中旬的 ipv6imp 邮件列表中详细讨论过，似乎并没有达成明确的共识（意味着取决于实现者）。你可能想查阅该邮件列表的归档。

如果一个服务器应用程序希望同时接受 IPv4 和 IPv6 连接，则有两种选择。

一种方法是使用 AF\_INET 和 AF\_INET6 套接字（你需要两个套接字）。使用 [getaddrinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getaddrinfo&sektion=3&format=html) 和 AI\_PASSIVE 作为 ai\_flags，然后使用 [socket(2)](https://man.freebsd.org/cgi/man.cgi?query=socket&sektion=2&format=html) 和 [bind(2)](https://man.freebsd.org/cgi/man.cgi?query=bind&sektion=2&format=html) 绑定所有返回的地址。通过打开多个套接字，你可以接受连接并将其绑定到适当的地址族。IPv4 连接将由 AF\_INET 套接字接收，IPv6 连接将由 AF\_INET6 套接字接收。

另一种方法是使用一个 AF\_INET6 通配符绑定套接字。使用 [getaddrinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getaddrinfo&sektion=3&format=html) 和 AI\_PASSIVE 设置 ai\_flags，使用 AF\_INET6 设置 ai\_family，并将主机名参数设置为 NULL。然后使用 [socket(2)](https://man.freebsd.org/cgi/man.cgi?query=socket&sektion=2&format=html) 和 [bind(2)](https://man.freebsd.org/cgi/man.cgi?query=bind&sektion=2&format=html) 绑定返回的地址（应为 IPv6 不指定地址）。通过这个单一套接字，你可以同时接收 IPv4 和 IPv6 数据包。

为了便于便捷地仅支持 IPv6 流量，使用 AF\_INET6 通配符绑定套接字时，始终在连接建立时检查对端地址。如果地址是 IPv4 映射地址，你可能希望拒绝该连接。你可以通过使用 `IN6_IS_ADDR_V4MAPPED()` 宏来检查这一条件。

为了更轻松地解决此问题，系统提供了一个与系统相关的 [setsockopt(2)](https://man.freebsd.org/cgi/man.cgi?query=setsockopt&sektion=2&format=html) 选项 `IPV6_BINDV6ONLY`，用法如下：

```c
int on;

	setsockopt(s, IPPROTO_IPV6, IPV6_BINDV6ONLY,
		   (char *)&on, sizeof (on)) < 0));
```

当此调用成功时，套接字将仅接收 IPv6 数据包。

#### 8.1.1.12.1. 发起端的注释

建议应用程序实现者：为了实现一个可移植的 IPv6 应用程序（在多个 IPv6 内核上工作），我们认为以下几点是成功的关键：

* **永远不要硬编码 AF\_INET 或 AF\_INET6。**
* 在系统中始终使用 [getaddrinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getaddrinfo&sektion=3&format=html) 和 [getnameinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getnameinfo&sektion=3&format=html)。永远不要使用 gethostby\*()、getaddrby\*()、inet\_*() 或 getipnodeby*()。（为了轻松更新现有应用程序以支持 IPv6，有时 getipnodeby\*() 会很有用。但如果可能，请尝试重写代码以使用 [getaddrinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getaddrinfo&sektion=3&format=html) 和 [getnameinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getnameinfo&sektion=3&format=html)。）
* 如果你想连接到目标，请使用 [getaddrinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getaddrinfo&sektion=3&format=html)，并尝试所有返回的目标地址，就像 [telnet(1)](https://man.freebsd.org/cgi/man.cgi?query=telnet&sektion=1&format=html) 所做的那样。
* 某些 IPv6 栈附带了有缺陷的 [getaddrinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getaddrinfo&sektion=3&format=html)。将一个最小可工作的版本随你的应用程序一起发布，并在必要时作为最后的选择使用。

如果你希望使用 AF\_INET6 套接字来处理 IPv4 和 IPv6 的外发连接，你将需要使用 [getipnodebyname(3)](https://man.freebsd.org/cgi/man.cgi?query=getipnodebyname&sektion=3&format=html)。如果你希望以最小的工作量更新现有应用程序以支持 IPv6，可以选择这种方法。但请注意，这是一个临时解决方案，因为 [getipnodebyname(3)](https://man.freebsd.org/cgi/man.cgi?query=getipnodebyname&sektion=3&format=html) 本身并不推荐使用，因为它根本不处理作用域 IPv6 地址。对于 IPv6 名称解析，推荐使用 [getaddrinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getaddrinfo&sektion=3&format=html) API。因此，当你有时间时，应重写应用程序，使用 [getaddrinfo(3)](https://man.freebsd.org/cgi/man.cgi?query=getaddrinfo&sektion=3&format=html)。

在编写需要发起连接的应用程序时，如果你将 AF\_INET 和 AF\_INET6 视为完全独立的地址族，事情会变得更简单。{set,get}sockopt 问题会更简单，DNS 问题也会简化。我们不建议依赖 IPv4 映射地址。

##### 8.1.1.12.1. 统一的 TCP 和 inpcb 代码

FreeBSD 4.x 在 IPv4 和 IPv6 之间共享 TCP 代码（来自 sys/netinet/tcp\*），并分开处理 UDP4/6 代码。它使用统一的 inpcb 结构。

该平台可以配置为支持 IPv4 映射地址。内核配置总结如下：

* 默认情况下，AF\_INET6 套接字将在某些条件下接管 IPv4 连接，并且可以发起到嵌入在 IPv4 映射 IPv6 地址中的 IPv4 目标的连接。
* 你可以通过 sysctl 禁用整个系统的 IPv4 映射地址，方法如下：
  `sysctl net.inet6.ip6.mapped_addr=0`

###### 8.1.1.12.1.1. 监听端

每个套接字可以配置为支持特殊的 AF\_INET6 通配符绑定（默认启用）。你可以通过 [setsockopt(2)](https://man.freebsd.org/cgi/man.cgi?query=setsockopt&sektion=2&format=html) 禁用它，方法如下：

```c
int on;

	setsockopt(s, IPPROTO_IPV6, IPV6_BINDV6ONLY,
		   (char *)&on, sizeof (on)) < 0));
```

只有在以下条件满足时，通配符 AF\_INET6 套接字才会接管 IPv4 连接：

* 没有与 IPv4 连接匹配的 AF\_INET 套接字。
* AF\_INET6 套接字配置为接受 IPv4 流量，即 getsockopt(IPV6\_BINDV6ONLY) 返回 0。

打开/关闭顺序没有问题。

###### 8.1.1.12.1.2. 发起端

FreeBSD 4.x 支持向 IPv4 映射地址（::ffff:10.1.1.1）发起连接，如果节点配置为支持 IPv4 映射地址。

#### 8.1.1.13. sockaddr\_storage

当 RFC2553 即将定稿时，关于如何命名 `struct sockaddr_storage` 成员有过讨论。一个提案是将 `*` 前缀加到成员名上（例如“\*ss\_len”），因为它们不应被直接修改。另一个提案是不加前缀（例如“ss\_len”），因为我们需要直接操作这些成员。对此没有明确的共识。

因此，RFC2553 定义了 `struct sockaddr_storage` 如下：

```c
struct sockaddr_storage {
		u_char	__ss_len;	/* 地址长度 */
		u_char	__ss_family;	/* 地址族 */
		/* 一堆填充 */
	};
```

相反，XNET 草案定义如下：

```c
struct sockaddr_storage {
		u_char	ss_len;		/* 地址长度 */
		u_char	ss_family;	/* 地址族 */
		/* 一堆填充 */
	};
```

在 1999 年 12 月，达成了一致意见，RFC2553bis 应该采纳后者（XNET）定义。

当前实现遵循 XNET 定义，基于 RFC2553bis 的讨论。

如果你查看多个 IPv6 实现，你会看到这两种定义。作为用户空间程序员，处理它的最便捷方式是：

1. 确保平台上有 `ss_family` 和/或 `ss_len`，可以使用 GNU autoconf，
2. 使用 `-Dss_family=*ss_family` 统一所有实例（包括头文件）为 `*ss_family`，或者
3. 永远不要触碰 `__ss_family`，将其转换为 `sockaddr *` 并使用 `sa_family`，如：

   ```
   struct sockaddr_storage ss;
   	family = ((struct sockaddr *)&ss)->sa_family;
   ```

### 8.1.2. 网络驱动程序

以下两个项目是标准驱动程序需要支持的：

1. **mbuf 聚类要求**。在这个稳定版本中，我们将 `MINCLSIZE` 改为 `MHLEN+1`，以便所有操作系统的驱动程序都能按照预期行为工作。
2. **多播**。如果 [ifmcstat(8)](https://man.freebsd.org/cgi/man.cgi?query=ifmcstat&sektion=8&format=html) 显示某个接口没有多播组，那么该接口必须进行修补。

如果任何驱动程序不支持这些要求，那么该驱动程序将无法用于 IPv6 和/或 IPsec 通信。如果你在使用 IPv6/IPsec 时发现任何问题，请报告给 [FreeBSD 问题报告邮件列表](https://lists.freebsd.org/subscription/freebsd-bugs)。

（注意：过去我们要求所有 PCMCIA 驱动程序必须调用 `in6_ifattach()`。现在我们不再有这样的要求。）

### 8.1.3. 翻译器

我们将 IPv4/IPv6 翻译器分为四种类型：

* **翻译器 A** --- 用于过渡的早期阶段，使得 IPv6 主机能够在 IPv6 网络中与 IPv4 主机建立连接。
* **翻译器 B** --- 用于过渡的早期阶段，使得 IPv4 主机能够在 IPv4 网络中与 IPv6 主机建立连接。
* **翻译器 C** --- 用于过渡的后期阶段，使得 IPv4 主机能够在 IPv4 网络中与 IPv6 主机建立连接。
* **翻译器 D** --- 用于过渡的后期阶段，使得 IPv6 主机能够在 IPv6 网络中与 IPv4 主机建立连接。

### 8.1.4. IPsec

IPsec 主要由三个组件组成：

1. 策略管理
2. 密钥管理
3. AH 和 ESP 处理

#### 8.1.4.1. 策略管理

内核实现了实验性的策略管理代码。有两种方式可以管理安全策略。一种是通过 [setsockopt(2)](https://man.freebsd.org/cgi/man.cgi?query=setsockopt&sektion=2&format=html) 配置每个套接字的策略。在这种情况下，策略配置在 [ipsec\_set\_policy(3)](https://man.freebsd.org/cgi/man.cgi?query=ipsec_set_policy&sektion=3&format=html) 中描述。另一种是通过 PF\_KEY 接口配置基于内核包过滤器的策略，使用 [setkey(8)](https://man.freebsd.org/cgi/man.cgi?query=setkey&sektion=8&format=html)。

策略条目不会与其索引重新排序，因此添加条目的顺序非常重要。

#### 8.1.4.2. 密钥管理

该工具包（sys/netkey）实现的密钥管理代码是一个自制的 PFKEY v2 实现，符合 RFC2367。

自制的 IKE 守护进程 "racoon" 包含在该工具包中（kame/kame/racoon）。基本上，你需要将 racoon 作为守护进程运行，然后设置一个策略以要求密钥（如 `ping -P 'out ipsec esp/transport//use'`）。内核将在必要时联系 racoon 守护进程以交换密钥。

#### 8.1.4.3. AH 和 ESP 处理

IPsec 模块作为标准 IPv4/IPv6 处理的“钩子”实现。当发送数据包时，`ip{,6}_output()` 会检查是否需要 ESP/AH 处理，通过检查是否找到匹配的 SPD（安全策略数据库）。如果需要 ESP/AH，`{esp,ah}{4,6}_output()` 会被调用，并且 mbuf 将相应更新。当数据包接收时，`{esp,ah}4_input()` 会根据协议号（即 `(*inetsw[proto])()`）被调用。`{esp,ah}4_input()` 会解密/检查数据包的真实性，并去除 ESP/AH 的链式头部和填充。由于我们在接收时不会直接使用收到的数据包，因此去除 ESP/AH 头部是安全的。

通过使用 ESP/AH，TCP4/6 的有效数据段大小将受到 ESP/AH 插入的额外链式头部的影响。我们的代码已经处理了这一情况。

基本的加密功能可以在 "sys/crypto" 目录中找到。ESP/AH 转换列在 `{esp,ah}_core.c` 文件中，包含包装函数。如果你希望添加某个算法，可以在 `{esp,ah}_core.c` 中添加包装函数，并将加密算法代码加入 `sys/crypto`。

隧道模式在本版本中部分支持，具有以下限制：

* IPsec 隧道不能与 GIF 泛型隧道接口结合使用。需要特别小心，因为我们可能会在 `ip_output()` 和 `tunnelifp→if_output()` 之间创建一个无限循环。关于是否统一它们，意见不一。
* MTU 和不分段位（IPv4）需要更多的检查，但基本上可以正常工作。
* AH 隧道的身份验证模型需要重新审视。最终，我们需要改进策略管理引擎。

#### 8.1.4.4. 遵循 RFC 和 ID 的一致性

内核中的 IPsec 代码符合（或尽力符合）以下标准：

“旧版 IPsec”规范在 **rfc182\[5-9].txt** 中有文档说明。

“新版 IPsec”规范在 **rfc240\[1-6].txt**、**rfc241\[01].txt**、**rfc2451.txt** 和 **draft-mcdonald-simple-ipsec-api-01.txt**（草案已过期，但可以从 [ftp://ftp.kame.net/pub/internet-drafts/](ftp://ftp.kame.net/pub/internet-drafts/) 获取）中有文档说明。 （注：IKE 规范 **rfc241\[7-9].txt** 在用户空间实现，作为 “racoon” IKE 守护进程）

当前支持的算法有：

* 旧版 IPsec AH

  * 空加密校验和（无文档，仅用于调试）
  * 使用 128 位加密校验和的键控 MD5（**rfc1828.txt**）
  * 使用 128 位加密校验和的键控 SHA1（无文档）
  * 使用 128 位加密校验和的 HMAC MD5（**rfc2085.txt**）
  * 使用 128 位加密校验和的 HMAC SHA1（无文档）
* 旧版 IPsec ESP

  * 空加密（无文档，类似于 **rfc2410.txt**）
  * DES-CBC 模式（**rfc1829.txt**）
* 新版 IPsec AH

  * 空加密校验和（无文档，仅用于调试）
  * 使用 96 位加密校验和的键控 MD5（无文档）
  * 使用 96 位加密校验和的键控 SHA1（无文档）
  * 使用 96 位加密校验和的 HMAC MD5（**rfc2403.txt**）
  * 使用 96 位加密校验和的 HMAC SHA1（**rfc2404.txt**）
* 新版 IPsec ESP

  * 空加密（**rfc2410.txt**）
  * 使用派生 IV 的 DES-CBC（**draft-ietf-ipsec-ciph-des-derived-01.txt**，草案已过期）
  * 使用显式 IV 的 DES-CBC（**rfc2405.txt**）
  * 使用显式 IV 的 3DES-CBC（**rfc2451.txt**）
  * BLOWFISH CBC（**rfc2451.txt**）
  * CAST128 CBC（**rfc2451.txt**）
  * RC5 CBC（**rfc2451.txt**）
  * 以上每种都可以与以下认证组合使用：

    * 使用 HMAC-MD5(96bit) 的 ESP 身份验证
    * 使用 HMAC-SHA1(96bit) 的 ESP 身份验证

不支持的算法：

* 旧版 IPsec AH

  * 使用 128 位加密校验和 + 64 位重放防止的 HMAC MD5（**rfc2085.txt**）
  * 使用 160 位加密校验和 + 32 位填充的键控 SHA1（**rfc1852.txt**）

IPsec（内核中）和 IKE（作为用户空间的“racoon”）已在多个互操作性测试活动中进行了测试，并且已知与许多其他实现良好互操作。此外，目前的 IPsec 实现覆盖了 RFC 中文档化的 IPsec 加密算法（仅涵盖没有知识产权问题的算法）。

#### 8.1.4.5. IPsec 隧道中的 ECN 考虑

如 **draft-ipsec-ecn-00.txt** 中所述，支持 ECN 友好的 IPsec 隧道。

正常的 IPsec 隧道在 RFC2401 中有描述。在封装时，IPv4 TOS 字段（或 IPv6 流量类别字段）将从内层 IP 头复制到外层 IP 头。在解封装时，外层 IP 头将被简单地丢弃。解封装规则与 ECN 不兼容，因为外层 IP TOS/流量类别字段中的 ECN 位将丢失。

为了使 IPsec 隧道支持 ECN，我们应修改封装和解封装过程。这在 [http://www.aciri.org/floyd/papers/draft-ipsec-ecn-00.txt](http://www.aciri.org/floyd/papers/draft-ipsec-ecn-00.txt) 第 3 章中有所描述。

IPsec 隧道实现可以通过设置 `net.inet.ipsec.ecn`（或 `net.inet6.ipsec6.ecn`）为某个值来实现三种行为：

* RFC2401：不考虑 ECN（sysctl 值为 -1）
* 禁止 ECN（sysctl 值为 0）
* 允许 ECN（sysctl 值为 1）

请注意，行为是按节点进行配置的，而不是按安全关联（SA）配置的（draft-ipsec-ecn-00 提出了按 SA 配置，但我认为这过于复杂）。

行为总结如下（更多细节请参阅源代码）：

```
封装                                解封装
                ---                             ---
RFC2401         将所有 TOS 位从内层复制到外层   丢弃外层的 TOS 位（按原样使用内层 TOS 位）

ECN forbidden   将除了 ECN 外的所有 TOS 位复制   丢弃外层的 TOS 位（按原样使用内层 TOS 位）
                （与 0xfc 相掩码）从内层复制到外层  将 ECN 位设置为 0。

ECN allowed     将除了 ECN 外的所有 TOS 位复制   使用内层 TOS 位，并做一些修改。如果外层 ECN CE 位
                （与 0xfe 相掩码）从内层复制到外层   为 1，则在内层启用 ECN CE 位
                将 ECN CE 位设置为 0。              。
```

配置的通用策略如下：

* 如果两个 IPsec 隧道端点都支持 ECN 友好的行为，最好将两端都配置为“允许 ECN”（sysctl 值为 1）。
* 如果另一端对 TOS 位要求非常严格，则使用“RFC2401”（sysctl 值为 -1）。
* 在其他情况下，使用“禁止 ECN”（sysctl 值为 0）。

默认行为是“禁止 ECN”（sysctl 值为 0）。

有关更多信息，请参阅：

[http://www.aciri.org/floyd/papers/draft-ipsec-ecn-00.txt](http://www.aciri.org/floyd/papers/draft-ipsec-ecn-00.txt)、RFC2481（显式拥塞通知）、`src/sys/netinet6/{ah,esp}_input.c`

（感谢 Kenjiro Cho [kjc@csl.sony.co.jp](mailto:kjc@csl.sony.co.jp) 提供详细分析）

#### 8.1.4.6. 互操作性

以下是 KAME 代码过去在多个平台上测试 IPsec/IKE 互操作性的情况。请注意，双方可能已修改过其实现，因此请仅将以下列表作为参考。

Altiga、Ashley-laurent（vpcom.com）、Data Fellows（F-Secure）、Ericsson ACC、FreeS/WAN、HITACHI、IBM AIX®、IIJ、Intel、Microsoft® Windows NT®、NIST（Linux IPsec + plutoplus）、Netscreen、OpenBSD、RedCreek、Routerware、SSH、Secure Computing、Soliton、Toshiba、VPNet、Yamaha RT100i
