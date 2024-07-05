# 第 7 章 Socket

## 7.1. 概要

BSD sockets 将进程间通信提升到一个新水平。通信进程不再需要在同一台机器上运行。它们仍然可以，但不一定要这样。

这些进程不仅无需在同一台机器上运行，它们也无需在同一操作系统下运行。多亏了 BSD sockets，您的 FreeBSD 软件可以与运行在 Macintosh®上的程序，运行在 Sun™工作站上的另一个程序，以及在 Windows® 2000 下运行的另一个程序平滑合作，它们都连接在基于以太网的局域网上。

但您的软件同样可以与在另一个建筑物内运行的进程合作，或者在另一个大陆上，潜艇内，或航天飞机内运行的进程合作。

它还可以与不属于计算机（至少不是严格意义上的计算机）的进程合作，例如打印机、数码相机、医疗设备等设备。几乎任何具有数字通信能力的设备。

## 7.2. 网络与多样性

我们已经暗示了网络的多样性。许多不同的系统必须相互通信。而且它们必须说同一种语言。它们还必须以相同的方式理解这种语言。

人们常常认为肢体语言是普遍适用的。但事实并非如此。在我十几岁的时候，我父亲带我去了保加利亚。当时我们坐在索非亚的一个公园的桌子旁，一个小贩走过来想要卖给我们一些炒杏仁。

那时候我还没学到太多保加利亚语，所以，我没有说不，而是摇了摇头，这在全世界都被认为是“不”的肢体语言。小贩很快开始给我们端来了一些杏仁。

后来我想起在保加利亚，摇头意味着说“是”。我马上开始点头。小贩注意到了，拿走了他的杏仁，然后走开了。对于一个不了解情况的观察者来说，我的肢体语言并没有改变：我继续使用摇头和点头的语言。改变的是肢体语言的含义。起初，小贩和我解释同样的语言，却具有完全不同的意义。我必须调整我对那种语言的理解，以便让小贩理解。

与计算机相同：相同的符号可能具有不同，甚至完全相反的含义。因此，为了让两台计算机彼此理解，它们不仅必须就相同的语言达成一致，还必须就语言的相同解释达成一致。

## 7.3. 协议

虽然各种编程语言往往具有复杂的语法并使用许多多字母保留字（这使得它们易于人类程序员理解），但数据通信的语言往往非常简洁。它们通常不使用多字节字，而是经常使用单个位。这样做有一个非常令人信服的理由：当数据在计算机内部以接近光速的速度传输时，它在两台计算机之间传输时往往要慢得多。

由于数据通信中使用的语言非常简洁，我们通常将其称为协议而不是语言。

当数据从一台计算机传输到另一台计算机时，它总是使用多个协议。这些协议是分层的。数据可以被比作洋葱的内部：你必须剥离几层“外皮”才能获得数据。这最好是用图片来说明：

![layers](https://docs.freebsd.org/images/books/developers-handbook/layers.png)

图 1. 协议层

在这个例子中，我们正在尝试从通过以太网连接的网页中获取图像。

图像由原始数据组成，这只是我们的软件可以处理的一系列 RGB 值，即将其转换为图像并显示在我们的显示器上。

可惜，我们的软件无法知道原始数据是如何组织的：它是一系列 RGB 值，还是一系列灰度强度，或者是 CMYK 编码的颜色？数据是否由 8 位量表示，或者它们的大小为 16 位，或者也许是 4 位？图像由多少行和列组成？某些像素应该是透明的吗？

我觉得你明白了吧…

为了告诉我们的软件如何处理原始数据，它被编码为 PNG 文件。它可以是 GIF，也可以是 JPEG，但它是 PNG。

PNG 是一种协议。

在这一点上，我能听到你们中的一些人在喊叫，“不，它不是！它是一种文件格式！”

当然，它当然是一种文件格式。但从数据通信的角度来看，文件格式就是一种协议：文件结构是一种语言，一种简洁的语言，向我们的进程传达数据的组织方式。因此，它就是一种协议。

遗憾的是，如果我们收到的只是 PNG 文件，我们的软件将面临一个严重的问题：它应该如何知道数据代表的是图像，而不是一些文本，或者可能是声音，或其他什么？其次，它应该如何知道图像是 PNG 格式，而不是 GIF，JPEG 或其他某种图像格式？

为了获取这些信息，我们正在使用另一种协议：HTTP。该协议可以准确告诉我们数据代表一幅镜像，并且使用 PNG 协议。它还可以告诉我们一些其他的东西，但让我们在这里专注于协议层面。

所以，现在我们有一些数据包裹在 PNG 协议中，再包裹在 HTTP 协议中。我们如何从服务器获取它呢？

通过 TCP/IP 在以太网上，就是这样。确实，这是另外三个协议。而不是继续从内到外讲述，我现在要谈论以太网，因为这样更容易解释其余部分。

以太网是一种连接局域网（LAN）中计算机的有趣系统。每台计算机都有一个网络接口卡（NIC），其中包含一个称为地址的独特的 48 位 ID。世界上没有两个以太网 NIC 具有相同的地址。

这些 NIC 都彼此连接在一起。每当一台计算机想要与同一以太网 LAN 中的另一台计算机通信时，它会通过网络发送一条消息。每个 NIC 都会看到这条消息。但作为以太网协议的一部分，数据包含目标 NIC 的地址（以及其他内容）。因此，所有网络接口卡中只有一个会注意到它，其余的会忽略它。

但并非所有计算机都连接到同一网络。仅仅因为我们通过以太网接收到数据并不意味着它起源于我们自己的局域网。它可能来自我们的网络通过互联网连接的其他网络（甚至可能不是基于以太网的网络）。

所有数据都是使用 IP（即互联网协议）在互联网上传输的。它的基本作用是让我们知道数据是从世界上的哪里到达的，以及它应该去哪里。它不能保证我们会接收到数据，只能告诉我们如果接收到数据的话它是从哪里来的。

即使我们接收到数据，IP 也不能保证我们会按照其他计算机发送给我们的顺序接收到不同的数据块。因此，我们可以在接收到图像的左上角之前接收到图像的中心，在接收到图像的右下角之后接收到它，例如。

是 TCP（即传输控制协议）要求发送方重新发送任何丢失的数据，并将其全部按正确的顺序放置。

总而言之，一个计算机向另一个计算机传输图像的过程需要五种不同的协议。我们接收到的数据被包装成 PNG 协议，然后被包装成 HTTP 协议，再被包装成 TCP 协议，接着是 IP 协议，最后是以太网协议。

哦，顺便提一下，可能还涉及到其他几种协议。例如，如果我们的局域网通过拨号连接到互联网，那么会使用调制解调器上的 PPP 协议，该调制解调器又使用各种调制解调器协议，等等，等等，等等……

作为开发者，你现在可能会问，“我应该如何处理这一切？”

幸运的是，您不必处理所有这些。 您只需要处理其中的一部分，而不是全部。 具体来说，您无需担心物理连接（在我们的情况下是以太网和可能的 PPP 等）。 您也无需处理 Internet 协议或传输控制协议。

换句话说，您无需做任何事情来接收来自其他计算机的数据。 好吧，您确实需要请求数据，但这几乎和打开文件一样简单。

一旦您收到数据，就由您决定如何处理它。 在我们的情况下，您需要了解 HTTP 协议和 PNG 文件结构。

使用类比，所有的互联网协议都变成了一个灰色地带：并不是因为我们不理解它是如何工作的，而是因为我们不再关心它。套接字接口为我们处理了这个灰色地带：

![slayers](https://docs.freebsd.org/images/books/developers-handbook/slayers.png)

图 2. 套接字覆盖的协议层

我们只需要理解告诉我们如何解释数据的任何协议，而不需要了解如何从另一个进程接收数据，也不需要了解如何将数据发送到另一个进程。

## 7.4. BSD 套接字模型

BSD 套接字建立在基本的 UNIX® 模型之上：一切皆为文件。在我们的例子中，套接字允许我们接收一个类似 HTTP 文件的内容。然后我们需要从中提取 PNG 文件。

由于互联网工作的复杂性，我们不能仅仅使用 open 系统调用，或者 open() C 函数。相反，我们需要采取几个步骤来“打开”一个套接字。

Once we do, however, we can start treating the *socket* the same way we treat any *file descriptor*: We can `read` from it, `write` to it, `pipe` it, and, eventually, `close` it.

## 7.5. Essential Socket Functions

While FreeBSD offers different functions to work with sockets, we only *need* four to "open" a socket. And in some cases we only need two.

### 7.5.1. 客户端-服务器的区别

通常，基于套接字的数据通信的一端是服务器，另一端是客户端。

#### 7.5.1.1. 共同元素

##### 7.5.1.1.1. `socket`

客户端和服务器都使用的一个函数是 socket(2)。它是这样声明的：

```
int socket(int domain, int type, int protocol);
```

返回值的类型与 open 相同，都是整数。FreeBSD 从与文件句柄相同的池中分配其值。这使得套接字可以像文件一样对待。

domain 参数告诉系统您希望使用哪种协议族。它们中有很多，有些是特定供应商的，其他则非常常见。它们在 sys/socket.h 中声明。

使用 PF_INET 用于 UDP、TCP 和其他互联网协议（IPv4）。

为 type 参数定义了五个值，在 sys/socket.h 中再次定义。所有这些值都以“SOCK_”开头。最常见的是 SOCK_STREAM ，它告诉系统您正在请求可靠的流传递服务（与 PF_INET 一起使用时为 TCP）。

如果您请求 SOCK_DGRAM ，您将请求无连接的数据报传递服务（在我们的情况下为 UDP）。

如果您想要负责低层协议（如 IP），甚至是网络接口（例如以太网），您需要指定 SOCK_RAW 。

最后， protocol 参数取决于前两个参数，并非始终有意义。在这种情况下，请使用 0 作为其值。

|  | 未连接的套接字<br /><br />现在，在 socket 函数中，我们并没有指定连接到哪个其他系统。我们新创建的套接字仍然保持未连接状态。<br /><br />这是有意为之：用电话类比来说，我们只是把调制解调器连接到了电话线上。我们既没有告诉调制解调器拨号，也没有告诉它在电话响时接听。 |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

##### 7.5.1.1.2. `sockaddr`

sockets 系列的各种函数期望内存中的一个小区域的地址（或指针，用 C 术语来说）。sys/socket.h 中的各种 C 声明将其称为 struct sockaddr 。这个结构在同一个文件中声明：

```
/*
 * Structure used by kernel to store most
 * addresses.
 */
struct sockaddr {
	unsigned char	sa_len;		/* total length */
	sa_family_t	sa_family;	/* address family */
	char		sa_data[14];	/* actually longer; address value */
};
#define	SOCK_MAXADDRLEN	255		/* longest possible addresses */
```

请注意 sa_data 字段声明的模糊性，就像一个 14 字节的数组一样，注释暗示可能不止 14 个。

这种模糊性是有意的。套接字是一个非常强大的接口。尽管大多数人可能认为它只是互联网接口而已 - 现在大多数应用程序可能仅用于此目的 - 套接字可以用于几乎任何类型的进程间通信，其中互联网（或更准确地说是 IP）只是其中之一。

sys/socket.h 是指套接字将处理的各种协议类型作为地址族，并在 sockaddr 的定义之前列出它们。

```
/*
 * Address families.
 */
#define	AF_UNSPEC	0		/* unspecified */
#define	AF_LOCAL	1		/* local to host (pipes, portals) */
#define	AF_UNIX		AF_LOCAL	/* backward compatibility */
#define	AF_INET		2		/* internetwork: UDP, TCP, etc. */
#define	AF_IMPLINK	3		/* arpanet imp addresses */
#define	AF_PUP		4		/* pup protocols: e.g. BSP */
#define	AF_CHAOS	5		/* mit CHAOS protocols */
#define	AF_NS		6		/* XEROX NS protocols */
#define	AF_ISO		7		/* ISO protocols */
#define	AF_OSI		AF_ISO
#define	AF_ECMA		8		/* European computer manufacturers */
#define	AF_DATAKIT	9		/* datakit protocols */
#define	AF_CCITT	10		/* CCITT protocols, X.25 etc */
#define	AF_SNA		11		/* IBM SNA */
#define AF_DECnet	12		/* DECnet */
#define AF_DLI		13		/* DEC Direct data link interface */
#define AF_LAT		14		/* LAT */
#define	AF_HYLINK	15		/* NSC Hyperchannel */
#define	AF_APPLETALK	16		/* Apple Talk */
#define	AF_ROUTE	17		/* Internal Routing Protocol */
#define	AF_LINK		18		/* Link layer interface */
#define	pseudo_AF_XTP	19		/* eXpress Transfer Protocol (no AF) */
#define	AF_COIP		20		/* connection-oriented IP, aka ST II */
#define	AF_CNT		21		/* Computer Network Technology */
#define pseudo_AF_RTIP	22		/* Help Identify RTIP packets */
#define	AF_IPX		23		/* Novell Internet Protocol */
#define	AF_SIP		24		/* Simple Internet Protocol */
#define	pseudo_AF_PIP	25		/* Help Identify PIP packets */
#define	AF_ISDN		26		/* Integrated Services Digital Network*/
#define	AF_E164		AF_ISDN		/* CCITT E.164 recommendation */
#define	pseudo_AF_KEY	27		/* Internal key-management function */
#define	AF_INET6	28		/* IPv6 */
#define	AF_NATM		29		/* native ATM access */
#define	AF_ATM		30		/* ATM */
#define pseudo_AF_HDRCMPLT 31		/* Used by BPF to not rewrite headers
					 * in interface output routine
					 */
#define	AF_NETGRAPH	32		/* Netgraph sockets */
#define	AF_SLOW		33		/* 802.3ad slow protocol */
#define	AF_SCLUSTER	34		/* Sitara cluster protocol */
#define	AF_ARP		35
#define	AF_BLUETOOTH	36		/* Bluetooth sockets */
#define	AF_MAX		37
```

用于 IP 的一个是 AF_INET。 它是常量 2 的符号。

它是 sockaddr 的 sa_family 字段中列出的地址族，决定了如何使用 sa_data 这些名义模糊的字节。

具体来说，每当地址族是 AF_INET 时，我们可以使用 netinet/in.h 中找到的 struct sockaddr_in ，无论 sockaddr 期望什么：

```
/*
 * Socket address, internet style.
 */
struct sockaddr_in {
	uint8_t		sin_len;
	sa_family_t	sin_family;
	in_port_t	sin_port;
	struct	in_addr sin_addr;
	char	sin_zero[8];
};
```

我们可以这样来可视化它的组织结构：

![sain](https://docs.freebsd.org/images/books/developers-handbook/sain.png)

图 3. sockaddr_in 结构

三个重要字段分别是 sin_family ，即结构的第一个字节， sin_port ，一个在第 2 和第 3 字节中找到的 16 位值，以及 sin_addr ，一个 32 位整数表示的 IP 地址，存储在第 4-7 字节中。

现在，让我们试着填写它。假设我们正在尝试为白天协议编写客户端，该协议简单地表示其服务器将向 port 13 写入表示当前日期和时间的文本字符串。我们想使用 TCP/IP，因此需要在地址族字段中指定 AF_INET 。 AF_INET 被定义为 2 。让我们使用 192.43.244.18 的 IP 地址，这是美国联邦政府的时间服务器（ time.nist.gov ）。

![sainfill](https://docs.freebsd.org/images/books/developers-handbook/sainfill.png)

sockaddr_in 的特定示例图 4。

顺便说一下， sin_addr 字段被声明为 struct in_addr 类型，在 netinet/in.h 中定义：

```
/*
 * Internet address (a structure for historical reasons)
 */
struct in_addr {
	in_addr_t s_addr;
};
```

除此之外， in_addr_t 是一个 32 位整数。

192.43.244.18 仅仅是一个便利的表示法，以列举所有 8 位字节，从最高有效位开始，来表达 32 位整数。

到目前为止，我们把 sockaddr 看作一个抽象概念。 我们的计算机不把 short 整数作为一个单独的 16 位实体来存储，而是作为 2 个字节的序列。类似地，它将 32 位整数存储为 4 个字节的序列。

假设我们编写了这样的代码：

```
sa.sin_family      = AF_INET;
sa.sin_port        = 13;
sa.sin_addr.s_addr = (((((192 << 8) | 43) << 8) | 244) << 8) | 18;
```

结果会是什么样子呢？

当然，这取决于情况。在基于 Pentium®或其他 x86 的计算机上，它会是这样的：

![sainlsb](https://docs.freebsd.org/images/books/developers-handbook/sainlsb.png)

Intel 系统上的 sockaddr_in 图 5。

在另一个系统上，它可能看起来像这样：

![sainmsb](https://docs.freebsd.org/images/books/developers-handbook/sainmsb.png)

MSB 系统上的 sockaddr_in 图 6。

在 PDP 上可能会看起来有所不同。但以上两种是今天最常见的使用方式。

通常，希望编写可移植代码的程序员会假装这些差异不存在。他们可以逃脱（除非他们用汇编语言编码）。然而，当编写套接字时，你不能那么轻易地逃脱。

 为什么？

因为在与另一台计算机通信时，通常不知道它是先存储数据的最高有效字节（MSB）还是最低有效字节（LSB）。

也许你在想，“所以，套接字不会帮我处理吗？”

 不会。

While that answer may surprise you at first, remember that the general sockets interface only understands the `sa_len` and `sa_family` fields of the `sockaddr` structure. You do not have to worry about the byte order there (of course, on FreeBSD `sa_family` is only 1 byte anyway, but many other UNIX® systems do not have `sa_len` and use 2 bytes for `sa_family`, and expect the data in whatever order is native to the computer).

But the rest of the data is just `sa_data[14]` as far as sockets goes. Depending on the *address family*, sockets just forwards that data to its destination.

Indeed, when we enter a port number, it is because we want the other computer to know what service we are asking for. And, when we are the server, we read the port number so we know what service the other computer is expecting from us. Either way, sockets only has to forward the port number as data. It does not interpret it in any way.

同样，我们输入 IP 地址告诉路上的每个人数据应该发送到哪里。套接字只是将其作为数据转发。

这就是为什么我们（程序员，而不是套接字）必须区分我们计算机使用的字节顺序和发送数据给其他计算机的传统字节顺序之间的差异。

我们将我们计算机使用的字节顺序称为主机字节顺序，或者只是主机顺序。

在 IP 上发送多字节数据的惯例是先发送最高有效字节。我们将其称为网络字节顺序，或简称网络顺序。

现在，如果我们为基于英特尔的计算机编译上述代码，我们的主机字节顺序将产生：

![sainlsb](https://docs.freebsd.org/images/books/developers-handbook/sainlsb.png)

图 7. 英特尔系统上的主机字节顺序

但网络字节顺序要求我们先将数据存储为 MSB：

![sainmsb](https://docs.freebsd.org/images/books/developers-handbook/sainmsb.png)

图 8. 网络字节顺序

不幸的是，我们的主机顺序恰好与网络顺序相反。

我们有几种处理它的方法。一种方法是在我们的代码中反转值：

```
sa.sin_family      = AF_INET;
sa.sin_port        = 13 << 8;
sa.sin_addr.s_addr = (((((18 << 8) | 244) << 8) | 43) << 8) | 192;
```

这将欺骗我们的编译器将数据存储为网络字节顺序。在某些情况下，这确实是解决问题的方法（例如，在汇编语言中编程时）。然而，在大多数情况下，这可能会导致问题。

假设你用 C 编写了一个基于套接字的程序。你知道它将在 Pentium® 上运行，所以你把所有常量反向输入，并强制它们符合网络字节顺序。这运行良好。

然后，有一天，您信任的旧奔腾®变成了生锈的旧奔腾®。您用一个主机顺序与网络顺序相同的系统来替换它。您需要重新编译所有的软件。除了您写的那个程序之外，所有软件继续表现良好。

自那以后，您已经忘记了您曾经强制所有常量为相反的主机顺序。您花费一些质量时间拔掉您的头发，叫出您听说过的所有神明的名字（和一些您编造的），用软皮球棒击打您的监视器，并执行尝试弄清为什么一切运作良好的事情突然完全不起作用的所有其他传统仪式。

最终，您弄清楚了，说了几句脏话，然后开始重新编写代码。

幸运的是，你不是第一个面对这个问题的人。其他人已经创建了 htons(3)和 htonl(3) C 函数，用于将主机字节顺序转换为网络字节顺序中的 short 和 long ，以及用于另一种方式的 ntohs(3)和 ntohl(3) C 函数。

在 MSB 优先系统上，这些函数不起作用。在 LSB 优先系统上，它们会将值转换为正确的顺序。

因此，无论你的软件在哪种系统上编译，如果你使用这些函数，你的数据都将按正确的顺序排列。

#### 7.5.1.2. 客户端功能

通常情况下，客户端会发起与服务器的连接。客户端知道它要调用哪个服务器：它知道服务器的 IP 地址，并且知道服务器所在的 port。这类似于你拿起电话拨打号码（地址），然后在有人接听后，询问负责 wingdings 的人（port）。

##### 7.5.1.2.1. `connect`

一旦客户端创建了一个套接字，它需要将其连接到远程系统上的特定 port。它使用 connect(2)：

```
int connect(int s, const struct sockaddr *name, socklen_t namelen);
```

s 参数是套接字，即 socket 函数返回的值。 name 是指向我们广泛讨论过的 sockaddr 结构的指针。最后， namelen 通知系统我们的 sockaddr 结构中有多少字节。

如果 connect 成功，则返回 0 。否则返回 -1 并将错误代码存储在 errno 中。

有许多原因可能导致 connect 失败。例如，尝试连接到互联网时，IP 地址可能不存在，或者它可能已关闭，或者忙得不可开交，或者在指定的 port 处没有服务器在监听。或者它可能直接拒绝对特定代码的任何请求。

##### 7.5.1.2.2. 我们的第一个客户

现在我们已经了解足够的信息，可以编写一个非常简单的客户端，该客户端将从 192.43.244.18 获取当前时间并将其打印到标准输出。

```
/*
 * daytime.c
 *
 * Programmed by G. Adam Stanislav
 */
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>

int main() {
  register int s;
  register int bytes;
  struct sockaddr_in sa;
  char buffer[BUFSIZ+1];

  if ((s = socket(PF_INET, SOCK_STREAM, 0)) < 0) {
    perror("socket");
    return 1;
  }

  bzero(&sa, sizeof sa);

  sa.sin_family = AF_INET;
  sa.sin_port = htons(13);
  sa.sin_addr.s_addr = htonl((((((192 << 8) | 43) << 8) | 244) << 8) | 18);
  if (connect(s, (struct sockaddr *)&sa, sizeof sa) < 0) {
    perror("connect");
    close(s);
    return 2;
  }

  while ((bytes = read(s, buffer, BUFSIZ)) > 0)
    write(1, buffer, bytes);

  close(s);
  return 0;
}
```

继续，在您的编辑器中输入它，保存为 daytime.c，然后编译并运行它：

```
% cc -O3 -o daytime daytime.c
% ./daytime

52079 01-06-19 02:29:25 50 0 1 543.9 UTC(NIST) *
%
```

在这种情况下，日期是 2001 年 6 月 19 日，时间是 02:29:25 UTC。当然，您的结果会有所不同。

#### 7.5.1.3. 服务器功能

典型的服务器不会发起连接。相反，它等待客户端调用并请求服务。它不知道客户端何时会调用，也不知道会有多少客户端会调用。它可能会静静地坐在那里等待，一会儿的功夫，下一刻可能会发现自己被来自多个客户端的请求淹没，所有客户端都在同一时间内呼叫。

套接字接口提供了三个基本函数来处理这个。

##### 7.5.1.3.1. `bind`

Ports 就像是电话线的分机：拨打号码后，您可以拨分机号码来联系特定的人员或部门。

IP ports 有 65535 个，但服务器通常只处理其中一个进来的请求。这就像告诉电话室操作员我们已经上班，并且可以在特定的分机上接听电话。我们使用 bind(2)告诉套接字我们要服务的port。

```
int bind(int s, const struct sockaddr *addr, socklen_t addrlen);
```

除了在 addr 中指定port外，服务器可能还包括其 IP 地址。但是，它可以只使用象征常数 INADDR_ANY 来指示它将为指定的port处理所有请求，而不管其 IP 地址是什么。这个符号以及其他几个类似的符号在 netinet/in.h 中声明。

```
#define	INADDR_ANY		(u_int32_t)0x00000000
```

假设我们正在为 TCP/IP 上的白天协议编写服务器。回想一下，它使用端口port 13。我们的 sockaddr_in 结构将如下所示：

![sainserv](https://docs.freebsd.org/images/books/developers-handbook/sainserv.png)

图 9. 示例服务器 sockaddr_in

##### 7.5.1.3.2. `listen`

继续我们办公电话类比，告诉电话总机您将在哪个分机后，您现在走进办公室，确保自己的电话已插好线并且铃声已打开。此外，确保您已激活呼叫等待功能，这样您在与某人通话时也能听到电话响铃。

服务器通过 listen(2) 函数确保所有这些。

```
int listen(int s, int backlog);
```

在这里， backlog 变量告诉套接字在您忙于处理上一个请求时要接受多少传入请求。换句话说，它确定了挂起连接队列的最大大小。

##### 7.5.1.3.3. `accept`

在你听到电话响的声音之后，通过接听电话来接受通话。您现在已与您的客户建立了连接。此连接保持活动状态，直到您或您的客户挂断电话。

服务器使用 accept(2)函数接受连接。

```
int accept(int s, struct sockaddr *addr, socklen_t *addrlen);
```

请注意，此时 addrlen 是一个指针。这是必要的，因为在这种情况下，是套接字填充 addr ， sockaddr_in 结构。

返回值是一个整数。实际上， accept 返回一个新的套接字。您将使用这个新套接字与客户端通信。

旧套接字会发生什么？它会继续监听更多请求（记住我们传递给 listen 的 backlog 变量吗？）直到我们 close 它。

现在，新套接字仅用于通信。它已完全连接。我们不能再次将其传递给 listen ，尝试接受其他连接。

##### 7.5.1.3.4. 我们的第一个服务器

我们的第一个服务器将比我们的第一个客户端复杂一些：我们不仅有更多的套接字函数可用，而且需要将其编写为守护进程。

最好的方法是在绑定port后创建一个子进程。然后主进程退出并将控制权返回给调用它的shell（或任何其他调用它的程序）。

该子项调用 listen ，然后启动一个无限循环，该循环接受连接、为其提供服务，最终关闭套接字。

```
/*
 * daytimed - a port 13 server
 *
 * Programmed by G. Adam Stanislav
 * June 19, 2001
 */
#include <stdio.h>
#include <string.h>
#include <time.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define BACKLOG 4

int main() {
    register int s, c;
    int b;
    struct sockaddr_in sa;
    time_t t;
    struct tm *tm;
    FILE *client;

    if ((s = socket(PF_INET, SOCK_STREAM, 0)) < 0) {
        perror("socket");
        return 1;
    }

    bzero(&sa, sizeof sa);

    sa.sin_family = AF_INET;
    sa.sin_port   = htons(13);

    if (INADDR_ANY)
        sa.sin_addr.s_addr = htonl(INADDR_ANY);

    if (bind(s, (struct sockaddr *)&sa, sizeof sa) < 0) {
        perror("bind");
        return 2;
    }

    switch (fork()) {
        case -1:
            perror("fork");
            return 3;
            break;
        default:
            close(s);
            return 0;
            break;
        case 0:
            break;
    }

    listen(s, BACKLOG);

    for (;;) {
        b = sizeof sa;

        if ((c = accept(s, (struct sockaddr *)&sa, &b)) < 0) {
            perror("daytimed accept");
            return 4;
        }

        if ((client = fdopen(c, "w")) == NULL) {
            perror("daytimed fdopen");
            return 5;
        }

        if ((t = time(NULL)) < 0) {
            perror("daytimed time");

            return 6;
        }

        tm = gmtime(&t);
        fprintf(client, "%.4i-%.2i-%.2iT%.2i:%.2i:%.2iZ\n",
            tm->tm_year + 1900,
            tm->tm_mon + 1,
            tm->tm_mday,
            tm->tm_hour,
            tm->tm_min,
            tm->tm_sec);

        fclose(client);
    }
}
```

我们首先创建一个套接字。然后在 sa 中填写 sockaddr_in 结构。注意对 INADDR_ANY 的条件性使用：

```
if (INADDR_ANY)
        sa.sin_addr.s_addr = htonl(INADDR_ANY);
```

它的值是 0 。由于我们刚刚使用 bzero 的整个结构，将其再次设置为 0 是多余的。但是如果我们将我们的代码移植到另一个系统，其中 INADDR_ANY 或许不是零，那么我们需要将其分配给 sa.sin_addr.s_addr 。大多数现代 C 编译器足够聪明，注意到 INADDR_ANY 是一个常量。只要它是零，它们就会优化整个条件语句从代码中移除。

在我们成功地调用 bind 后，我们准备成为守护程序：我们使用 fork 来创建一个子进程。在父进程和子进程中， s 变量是我们的套接字。父进程不需要它，所以它调用 close ，然后返回 0 以通知它自己的父进程它已成功终止。

与此同时，子进程继续在后台工作。它调用 listen 并将其后备设置为 4 。这里不需要一个很大的值，因为白天不是许多客户端一直请求的协议，并且因为它可以立即处理每个请求。

最后，守护程序启动一个无休止的循环，执行以下步骤：

1. 调用 accept 。它在这里等待，直到客户端联系。此时，它会接收一个新的套接字 c ，用于与特定客户端通信。
2. 使用 C 函数 fdopen 将套接字从低级文件描述符转换为 C 风格的 FILE 指针。这将允许稍后使用 fprintf 。
3. 检查时间，并以 ISO 8601 格式打印到 client "文件"。然后使用 fclose 关闭文件。这将自动关闭套接字。

我们可以概括这一点，并将其用作许多其他服务器的模型:

![serv](https://docs.freebsd.org/images/books/developers-handbook/serv.png)

图 10. 顺序服务器

这个流程图适用于顺序服务器，即一次只能为一个客户提供服务的服务器，就像我们之前在白天服务器中所能做的一样。只有在客户端和服务器之间没有真正的“对话”时才有可能：一旦服务器检测到与客户端的连接，它就会发送一些数据然后关闭连接。整个操作可能只需几纳秒，就完成了。

这个流程图的优点是，除了父 fork 退出之前的短暂时刻外，始终只有一个进程处于活动状态：我们的服务器不会占用太多内存和其他系统资源。

请注意，我们在流程图中添加了初始化守护程序。我们不需要初始化自己的守护程序，但这是程序流程中设置任何 signal 处理程序、打开可能需要的任何文件等的好地方。

流程图中的几乎所有内容都可以在许多不同的服务器上直接使用。serve 条目是个例外。我们将其视为一个“黑匣子”，即，您专门为自己的服务器设计的东西，只需“将其插入其余部分”即可。

并非所有协议都那么简单。许多协议接收来自客户端的请求，然后回复该请求，接着再次接收来自同一客户端的请求。因此，它们事先无法知道为客户端提供服务的时长。这类服务器通常为每个客户端启动一个新进程。在新进程为其客户端提供服务时，守护程序可以继续侦听更多连接。

现在，继续吧，将上述源代码保存为 daytimed.c（以字母 0 结尾的守护程序名称是惯例）。编译后，尝试运行它：

```
% ./daytimed
bind: Permission denied
%
```

发生了什么？正如你所记得的，白天协议使用 13。但是 1024 以下的所有端口都保留给超级用户（否则，任何人都可以启动一个假装为常用端口提供服务的守护程序，从而造成安全漏洞）。

请再试一次，这次以超级用户身份：

```
# ./daytimed
#
```

什么... 什么也没有？让我们再试一次：

```
# ./daytimed

bind: Address already in use
#
```

每个port 一次只能由一个程序绑定。我们的第一次尝试确实成功了：它启动了子守护程序并静静地返回。它仍在运行，并将继续运行，直到您杀死它，或者它的任何系统调用失败，或者您重新启动系统。

好吧，我们知道它在后台运行。但它是否正常工作呢？我们如何知道它是一个适合白天运行的服务器呢？简单：

```
% telnet localhost 13

Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
2001-06-19T21:04:42Z
Connection closed by foreign host.
%
```

telnet 尝试了新的 IPv6，但失败了。它改用 IPv4 再次尝试，成功了。守护进程有效。

如果您可以通过 telnet 访问另一个 UNIX®系统，则可以使用它来测试远程访问服务器。我的计算机没有静态 IP 地址，所以我这样做：

```
% who

whizkid          ttyp0   Jun 19 16:59   (216.127.220.143)
xxx              ttyp1   Jun 19 16:06   (xx.xx.xx.xx)
% telnet 216.127.220.143 13

Trying 216.127.220.143...
Connected to r47.bfm.org.
Escape character is '^]'.
2001-06-19T21:31:11Z
Connection closed by foreign host.
%
```

再次，它有效。使用域名会有效吗？

```
% telnet r47.bfm.org 13

Trying 216.127.220.143...
Connected to r47.bfm.org.
Escape character is '^]'.
2001-06-19T21:31:40Z
Connection closed by foreign host.
%
```

顺便说一下，telnet 在我们的守护程序关闭套接字后打印“连接由外部主机关闭”消息。这向我们表明，确实，在我们的代码中使用 fclose(client); 是有效的。

## 7.6. 助手函数

FreeBSD C 库包含许多用于套接字编程的辅助函数。例如，在我们的示例客户端中，我们硬编码了 time.nist.gov IP 地址。但我们并不总是知道 IP 地址。即使我们知道，如果软件允许用户输入 IP 地址甚至域名，它会更加灵活。

### 7.6.1. `gethostbyname`

虽然没有办法将域名直接传递给任何套接字函数，但 FreeBSD C 库带有在 netdb.h 中声明的 gethostbyname(3) 和 gethostbyname2(3) 函数。

```
struct hostent * gethostbyname(const char *name);
struct hostent * gethostbyname2(const char *name, int af);
```

两者都返回指向 hostent 结构的指针，其中包含许多关于域的信息。就我们的目的而言，结构的 h_addr_list[0] 字段指向已存储在网络字节顺序中的 h_length 字节的正确地址。

这使我们能够创建一个更加灵活和更有用的白天程序的版本：

```
/*
 * daytime.c
 *
 * Programmed by G. Adam Stanislav
 * 19 June 2001
 */
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>

int main(int argc, char *argv[]) {
  register int s;
  register int bytes;
  struct sockaddr_in sa;
  struct hostent *he;
  char buf[BUFSIZ+1];
  char *host;

  if ((s = socket(PF_INET, SOCK_STREAM, 0)) < 0) {
    perror("socket");
    return 1;
  }

  bzero(&sa, sizeof sa);

  sa.sin_family = AF_INET;
  sa.sin_port = htons(13);

  host = (argc > 1) ? (char *)argv[1] : "time.nist.gov";

  if ((he = gethostbyname(host)) == NULL) {
    herror(host);
    return 2;
  }

  bcopy(he->h_addr_list[0],&sa.sin_addr, he->h_length);

  if (connect(s, (struct sockaddr *)&sa, sizeof sa) < 0) {
    perror("connect");
    return 3;
  }

  while ((bytes = read(s, buf, BUFSIZ)) > 0)
    write(1, buf, bytes);

  close(s);
  return 0;
}
```

现在，我们可以在命令行上输入域名（或 IP 地址，两者均可），程序将尝试连接到其白天服务器。否则，它仍将默认为 time.nist.gov 。但是，即使在这种情况下，我们也将使用 gethostbyname 而不是硬编码 192.43.244.18 。这样，即使其 IP 地址在未来发生变化，我们仍然可以找到它。

由于从您的本地服务器获取时间几乎不需要时间，您可以连续两次运行白天：首先从 time.nist.gov 获取时间，第二次从您自己的系统获取时间。然后，您可以比较结果，看看您的系统时钟是多么精确。

```
% daytime ; daytime localhost

52080 01-06-20 04:02:33 50 0 0 390.2 UTC(NIST) *
2001-06-20T04:02:35Z
%
```

正如您所看到的，我的系统比 NIST 时间提前两秒。

### 7.6.2. `getservbyname`

有时候，您可能不确定某个服务使用了什么port。在这种情况下，getservbyname(3)函数非常方便，该函数也在 netdb.h 中声明：

```
struct servent * getservbyname(const char *name, const char *proto);
```

结构体 servent 包含了 s_port ，其中包含了已经按网络字节顺序排列的正确port。

如果我们不知道白天服务的正确port，我们可以通过这种方式找到它：

```
struct servent *se;
  ...
  if ((se = getservbyname("daytime", "tcp")) == NULL {
    fprintf(stderr, "Cannot determine which port to use.\n");
    return 7;
  }
  sa.sin_port = se->s_port;
```

通常您确实知道port。 但是，如果您正在开发新协议，可能会在非官方port上进行测试。 将来的某一天，您将会注册协议及其port（如果不在别处，在您的 / etc / services 中至少会有 getservbyname ）。 在上述代码中，而不是返回错误，您只需使用临时port号码。 一旦您在 / etc / services 中列出了协议，您的软件将自动找到其port，而无需重新编写代码。

## 7.7. 并发服务器

与顺序服务器不同，并发服务器必须能够同时为多个客户端提供服务。例如，聊天服务器可能会为特定客户端提供几个小时的服务，它不能等到停止为一个客户端提供服务后再为下一个客户端提供服务。

这要求我们的流程图有显著变化：

![serv2](https://docs.freebsd.org/images/books/developers-handbook/serv2.png)

图 11. 并发服务器

我们将服务器从守护进程移动到自己的服务器进程中。但是，由于每个子进程继承所有打开的文件（而套接字被视为文件），新进程不仅继承了“已接受句柄”即 accept 调用返回的套接字，还继承了顶部套接字，即顶部进程在一开始打开的套接字。

但是，服务器进程不需要此套接字，应立即 close 。同样，守护进程不再需要已接受的套接字，不仅应该，而且必须 close 它，否则，迟早会耗尽可用的文件描述符。

在服务器进程完成服务后，应关闭已接受的套接字。它现在退出，而不是返回 accept 。

在 UNIX®下，一个进程并不真正退出。相反，它会返回给其父进程。通常，父进程会等待其子进程，并获取返回值。然而，我们的守护进程不能简单地停止并等待。那样会背离创建额外进程的初衷。但如果它永远不这样做，其子进程将变成僵尸-不再起作用但仍然在系统中游荡。

出于这个原因，守护进程需要在其初始化守护进程阶段设置信号处理程序。至少要处理一个 SIGCHLD 信号，这样守护进程可以从系统中移除僵尸返回值，并释放它们占用的系统资源。

这就是为什么我们的流程图现在包含一个处理信号的框，它不连接到任何其他框。顺便说一句，许多服务器也处理 SIGHUP 信号，并通常解释为超级用户发出的信号，告诉它们应重新读取其配置文件。这使我们能够在无需终止和重新启动这些服务器的情况下更改设置。
